<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Debug' step directly interpolates `${{ toJSON(inputs) }}` inside a `run:` block. GitHub Actions performs template substitution before the shell executes, so all inputs (including attacker-controlled values like `inputs.plugin`, `inputs.version`, `inputs.zip-url`) are expanded into the shell command string before bash sees it. This allows an attacker to inject arbitrary shell commands via any input value. Offending line: `echo '${{ toJSON(inputs) }}'`

Locations:

- `action.yml:42`

### github-env-injection (severity: high)

The 'Fetch ZIP from the plugin directory' step writes the `PLUGIN_HOST` variable to `$GITHUB_ENV` using a value derived from `$ZIP_URL` (which is set from `inputs.zip-url`, an untrusted input). The write is: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`. There is no sanitization (`printf '%s' ... | tr -d '\n\r'`) applied before the write. An attacker can supply a `zip-url` input containing newline characters to inject arbitrary environment variables into `$GITHUB_ENV`, potentially overwriting sensitive variables for subsequent steps.

Locations:

- `action.yml:57`

### unpinned-uses (severity: high)

The 'Generate attestation for the ZIP' step references `actions/attest-build-provenance@v1`, which uses a mutable version tag (`@v1`) rather than a pinned 40-character commit SHA. A compromised or updated tag could silently substitute malicious code. It should be pinned to a full SHA, e.g. `actions/attest-build-provenance@<40-char-sha> # v1`.

Locations:

- `action.yml:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed three security findings in hardened/action/action.yml: (1) script-injection: moved `${{ toJSON(inputs) }}` out of the run block into an env var `INPUTS_JSON` and echoed it as `"$INPUTS_JSON"`; (2) github-env-injection: sanitized the PLUGIN_HOST value by using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV to prevent newline injection; (3) unpinned-uses: pinned `actions/attest-build-provenance@v1` to its full commit SHA `ef244123eb79f2f7a7e75d99086184180e6d0018` with a `# v1` comment.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted variable expansions in the 'Fetch ZIP from the plugin directory' step of action.yml:
1. Quoted $PLUGIN in bash parameter substitution: `${zipurl//%plugin%/"$PLUGIN"}`
2. Quoted $VERSION in bash parameter substitution: `${zipurl//%version%/"$VERSION"}`
3. Quoted ${TIMEOUT} in arithmetic context: `$(( "${TIMEOUT}" * $per_minute ))`

These changes prevent shell metacharacters (whitespace, glob characters, etc.) in untrusted input values from being interpreted by the shell.

