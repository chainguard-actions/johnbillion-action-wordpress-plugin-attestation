<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `zipurl` is constructed from untrusted inputs (`inputs.zip-url` → `$ZIP_URL`, `inputs.plugin` → `$PLUGIN`, `inputs.version` → `$VERSION`) and then written to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`).

1. `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` — the hostname extracted from a caller-controlled URL is written to GITHUB_ENV unsanitized.
2. `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"` — the full caller-controlled URL is written to GITHUB_OUTPUT unsanitized.

An attacker who controls `inputs.zip-url` (or `inputs.plugin`/`inputs.version`) can embed newline characters to inject arbitrary key=value pairs into the runner's environment or output context, potentially overwriting sensitive variables consumed by later steps.

Locations:

- `action.yml:62`
- `action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Fetch zip from the plugin directory' step in action.yml. The zipurl value (derived from caller-controlled inputs zip-url, plugin, and version) was being written to $GITHUB_ENV and $GITHUB_OUTPUT without newline sanitization. Added sanitization using `printf '%s' ... | tr -d '\n\r'` for both the full URL written to GITHUB_OUTPUT and the hostname extracted for GITHUB_ENV. The safe_zipurl and safe_host variables are now stripped of newline/carriage-return characters before being written to the runner's environment and output context.

