<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **johnbillion--action-wordpress-plugin-attestation/0.6.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Debug' step directly interpolates `${{ toJSON(inputs) }}` inside a `run:` shell command. GitHub Actions performs template substitution before the shell executes, so attacker-controlled inputs (plugin, version, zip-url, etc.) can inject arbitrary shell commands. The offending line is: `echo '${{ toJSON(inputs) }}'`

Locations:

- `action.yml:42`

### script-injection (severity: high)

Sub-rule (b): In the 'Fetch ZIP from the plugin directory' step, the env vars `$PLUGIN`, `$VERSION`, and `$ZIP_URL` (all sourced from untrusted `inputs.*`) are expanded unquoted inside shell parameter substitutions: `zipurl=${zipurl//%plugin%/$PLUGIN}` and `zipurl=${zipurl//%version%/$VERSION}`. Unquoted expansions allow shell metacharacter injection from attacker-controlled input values.

Locations:

- `action.yml:57`
- `action.yml:58`

### github-env-injection (severity: high)

The 'Fetch ZIP from the plugin directory' step writes `PLUGIN_HOST` to `$GITHUB_ENV` using a value derived from `$ZIP_URL`, `$PLUGIN`, and `$VERSION` — all sourced from untrusted `inputs.*` — without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into these inputs to poison `$GITHUB_ENV` with arbitrary environment variable assignments. The offending line is: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`

Locations:

- `action.yml:60`

### unpinned-uses (severity: high)

The 'Generate attestation for the ZIP' step references `actions/attest-build-provenance@v1`, which uses a mutable tag instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to a different (potentially malicious) commit.

Locations:

- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all four findings in action.yml:
1. script-injection (a): Moved `${{ toJSON(inputs) }}` into an env var `INPUTS_JSON` and echoed it safely as `"$INPUTS_JSON"` in the Debug step.
2. script-injection (b): Quoted the bash parameter substitutions in the Fetch ZIP step: `"${zipurl//%plugin%/"$PLUGIN"}"` and `"${zipurl//%version%/"$VERSION"}"` to prevent shell metacharacter injection.
3. github-env-injection: Sanitized the `PLUGIN_HOST` value before writing to `$GITHUB_ENV` using `printf '%s' "$zipurl" | awk -F/ '{print $3}' | tr -d '\n\r'` to strip newlines.
4. unpinned-uses: Pinned `actions/attest-build-provenance@v1` to full SHA `ef244123eb79f2f7a7e75d99086184180e6d0018` with a `# v1` comment.

