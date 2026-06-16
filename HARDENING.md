<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `$zipurl` is constructed from untrusted inputs (`inputs.zip-url` → `$ZIP_URL`, `inputs.plugin` → `$PLUGIN`, `inputs.version` → `$VERSION`) and then written to `$GITHUB_ENV` (as `PLUGIN_HOST`) and `$GITHUB_OUTPUT` (as `zip-url`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker controlling these inputs could inject newlines to set arbitrary environment variables or outputs. The offending lines are:
  - `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`
  - `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:75`
- `action.yml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in action.yml at lines 75-76. The `$zipurl` variable (derived from untrusted inputs `inputs.zip-url`, `inputs.plugin`, and `inputs.version`) was being written directly to `$GITHUB_ENV` (as `PLUGIN_HOST`) and `$GITHUB_OUTPUT` (as `zip-url`) without sanitization. The fix introduces two sanitized variables: `safe_zipurl` (zipurl with newlines/carriage-returns stripped via `printf '%s' | tr -d '\n\r'`) and `safe_host` (hostname extracted from safe_zipurl, also stripped of newlines). These sanitized values are then used when writing to `$GITHUB_ENV` and `$GITHUB_OUTPUT`, preventing newline injection attacks.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in the 'Fetch zip from the plugin directory' run block in action.yml. Changed `zipurl=${zipurl//%plugin%/$PLUGIN}` to `zipurl=${zipurl//%plugin%/"$PLUGIN"}` and `zipurl=${zipurl//%version%/$VERSION}` to `zipurl=${zipurl//%version%/"$VERSION"}`. The double quotes around the variables within the bash parameter substitution expressions prevent shell metacharacters in attacker-controlled input values from causing unexpected behavior.

