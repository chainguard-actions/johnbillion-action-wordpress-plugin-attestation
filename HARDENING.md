<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Fetch zip from the plugin directory' step writes values derived from untrusted inputs to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization.

1. `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` — `zipurl` is constructed from `$ZIP_URL` (inputs.zip-url), `$PLUGIN` (inputs.plugin), and `$VERSION` (inputs.version), all caller-controlled. A newline embedded in any of these inputs could inject arbitrary environment variables into $GITHUB_ENV.

2. `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"` — same untrusted source written to $GITHUB_OUTPUT without sanitization.

Neither write is preceded by the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker controlling any of inputs.zip-url, inputs.plugin, or inputs.version can inject newlines to set arbitrary GITHUB_ENV or GITHUB_OUTPUT entries.

Locations:

- `action.yml:69`
- `action.yml:70`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Fetch zip from the plugin directory' step in action.yml. The two writes to $GITHUB_ENV and $GITHUB_OUTPUT that used the unsanitized `zipurl` variable (derived from caller-controlled inputs: zip-url, plugin, version) have been replaced with sanitized versions. Added `safe_zipurl=$(printf '%s' "$zipurl" | tr -d '\n\r')` and `safe_plugin_host=$(printf '%s' "$zipurl" | awk -F/ '{print $3}' | tr -d '\n\r')` before writing to $GITHUB_ENV and $GITHUB_OUTPUT respectively. This prevents newline injection attacks where an attacker could embed newlines in any of the inputs to inject arbitrary environment variables or output values.

