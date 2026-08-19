<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch ZIP from the plugin directory' step, the variable `zipurl` is constructed from untrusted inputs (`inputs.zip-url`, `inputs.plugin`, `inputs.version`) via the env vars `$ZIP_URL`, `$PLUGIN`, and `$VERSION`. This value is then written directly to `$GITHUB_ENV` (as `PLUGIN_HOST`) and `$GITHUB_OUTPUT` (as `zip-url`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who controls any of these inputs could inject newlines to set arbitrary environment variables or output values, potentially hijacking subsequent steps.

Locations:

- `action.yml:68`
- `action.yml:69`

### unpinned-uses (severity: high)

The composite action step 'Generate attestation for the ZIP' references `actions/attest-build-provenance@v2`, which uses a mutable version tag (`@v2`) rather than a pinned 40-character commit SHA. If the tag is moved (intentionally or via a supply-chain compromise), the action will silently execute different code.

Locations:

- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses

**Notes:**

1. Fixed github-env-injection in the 'Fetch ZIP from the plugin directory' step: introduced `safe_zipurl` and `safe_plugin_host` variables that strip newlines/carriage-returns via `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT. 2. Pinned actions/attest-build-provenance@v2 to its full commit SHA e8998f949152b193b063cb0ec769d69d929409be with a # v2 comment preserved for readability.

