<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `zipurl` is constructed from untrusted inputs (`inputs.zip-url` → $ZIP_URL, `inputs.plugin` → $PLUGIN, `inputs.version` → $VERSION) and then written to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline in any of these inputs could inject additional environment variables or output key-value pairs.

Line 72: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` — unsanitized write to GITHUB_ENV.
Line 73: `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"` — unsanitized write to GITHUB_OUTPUT.

Locations:

- `action.yml:72`
- `action.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in action.yml at lines 72-73. The `zipurl` value (constructed from user-controlled inputs `inputs.zip-url`, `inputs.plugin`, and `inputs.version`) was being written directly to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization. Fixed by: (1) creating `safe_zipurl` via `printf '%s' "$zipurl" | tr -d '\n\r'` and writing that to $GITHUB_OUTPUT, and (2) creating `safe_plugin_host` via `printf '%s' "$zipurl" | awk -F/ '{print $3}' | tr -d '\n\r'` and writing that to $GITHUB_ENV. This prevents an attacker-controlled newline in any of the inputs from injecting additional environment variables or output key-value pairs.

