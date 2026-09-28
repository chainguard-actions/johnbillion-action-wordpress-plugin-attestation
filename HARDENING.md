<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Debug' step directly interpolates `${{ toJSON(inputs) }}` inside a `run:` shell command. GitHub Actions expands `${{ }}` expressions before the shell executes the script, so attacker-controlled input values (inputs.plugin, inputs.version, inputs.zip-url, etc.) embedded in the JSON output can inject arbitrary shell commands. The offending line is: `echo '${{ toJSON(inputs) }}'`

Locations:

- `action.yml:42`

### github-env-injection (severity: high)

The 'Fetch ZIP from the plugin directory' step writes `PLUGIN_HOST` to `$GITHUB_ENV` without sanitization. The value is derived from `$ZIP_URL` (inputs.zip-url), `$PLUGIN` (inputs.plugin), and `$VERSION` (inputs.version) — all untrusted caller-controlled inputs — via shell string substitution. A newline character in any of these inputs could inject additional environment variable assignments. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent. Offending line: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`

Locations:

- `action.yml:55`

### unpinned-uses (severity: high)

The step 'Generate attestation for the ZIP' references `actions/attest-build-provenance@v1`, which uses a mutable version tag (`v1`) rather than a pinned 40-character commit SHA. If the tag is moved (intentionally or via a supply-chain compromise), the action will silently execute different code.

Locations:

- `action.yml:97`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed three findings in hardened/action/action.yml: (1) script-injection: moved `${{ toJSON(inputs) }}` out of the run shell into an env var `INPUTS_JSON`, referenced as `"$INPUTS_JSON"` in the script; (2) github-env-injection: sanitized PLUGIN_HOST by piping through `tr -d '\n\r'` before writing to $GITHUB_ENV, using `printf '%s'` to safely pass the value; (3) unpinned-uses: pinned `actions/attest-build-provenance@v1` to full SHA `ef244123eb79f2f7a7e75d99086184180e6d0018` with `# v1` comment.

