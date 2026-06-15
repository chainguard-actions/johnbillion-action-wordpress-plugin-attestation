<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.3** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `zipurl` is constructed from untrusted inputs (`inputs.zip-url`, `inputs.plugin`, `inputs.version`) via env vars (`$ZIP_URL`, `$PLUGIN`, `$VERSION`). This value is then written unsanitized to both `$GITHUB_ENV` (as `PLUGIN_HOST`) and `$GITHUB_OUTPUT` (as `zip-url`) without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. An attacker-controlled input containing embedded newlines could inject arbitrary environment variables or output key-value pairs. The two failing lines are:
  1. `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`
  2. `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:68`
- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Fetch zip from the plugin directory' step in action.yml. The two unsanitized writes to $GITHUB_ENV and $GITHUB_OUTPUT were replaced with sanitized versions:
1. `PLUGIN_HOST`: now uses `printf '%s' ... | tr -d '\n\r'` to strip newlines from the extracted host before writing to $GITHUB_ENV.
2. `zip-url`: now uses `printf '%s' "$zipurl" | tr -d '\n\r'` to strip newlines from the zip URL before writing to $GITHUB_OUTPUT.
This prevents attacker-controlled inputs containing embedded newlines from injecting arbitrary environment variables or output key-value pairs.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in action.yml at lines 68-69. The $PLUGIN and $VERSION environment variables (sourced from inputs.plugin and inputs.version) were used unquoted inside bash parameter substitution expressions. Added double quotes around both variables: changed `zipurl=${zipurl//%plugin%/$PLUGIN}` to `zipurl=${zipurl//%plugin%/"$PLUGIN"}` and `zipurl=${zipurl//%version%/$VERSION}` to `zipurl=${zipurl//%version%/"$VERSION"}`. This prevents shell metacharacters in attacker-controlled input values from being interpreted by the shell.

