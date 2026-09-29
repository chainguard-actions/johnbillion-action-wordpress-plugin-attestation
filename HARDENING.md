<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The Debug step directly interpolates `${{ toJSON(inputs) }}` inside a `run:` shell command string. GitHub Actions substitutes this expression before the shell executes the script. If any input value contains a single quote, it breaks out of the surrounding single-quoted string (`echo '${{ toJSON(inputs) }}'`) and allows arbitrary shell command injection. All inputs — including `inputs.plugin`, `inputs.version`, `inputs.zip-url`, etc. — are attacker-controllable.

Locations:

- `action.yml:44`

### github-env-injection (severity: high)

The 'Fetch ZIP from the plugin directory' step writes `PLUGIN_HOST` to `$GITHUB_ENV` using a value derived from `$zipurl`, which is constructed from the untrusted inputs `inputs.zip-url`, `inputs.plugin`, and `inputs.version` (via env vars `$ZIP_URL`, `$PLUGIN`, `$VERSION`). The write `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` is performed without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into the input values to write arbitrary key=value pairs into the GitHub environment file, potentially overriding environment variables used by subsequent steps.

Locations:

- `action.yml:60`

### unpinned-uses (severity: high)

The composite action step uses `actions/attest-build-provenance@v1`, which is pinned to a mutable tag (`@v1`) rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `actions/attest-build-provenance@<40-char-sha> # v1`.

Locations:

- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

1. script-injection (line 44): Moved `${{ toJSON(inputs) }}` from the run shell string into an env var `INPUTS_JSON`, then referenced it as `"$INPUTS_JSON"` in the shell script to prevent shell injection via attacker-controlled input values. 2. github-env-injection (line 60): Replaced the direct `echo PLUGIN_HOST=...` write with a sanitized two-step approach: extract the host with printf+awk, then strip newlines with `tr -d '\n\r'` before writing to $GITHUB_ENV to prevent newline injection attacks. 3. unpinned-uses (line 100): Pinned `actions/attest-build-provenance@v1` to the immutable commit SHA `ef244123eb79f2f7a7e75d99086184180e6d0018` with a `# v1` comment for readability.

