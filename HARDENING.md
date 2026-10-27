<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, zipurl is built from inputs.zip-url, inputs.plugin and inputs.version (passed in as the ZIP_URL, PLUGIN and VERSION env vars). The step then writes it to the special files without removing newlines first: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` and `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"`. A value containing a newline could inject extra env vars or outputs. Clean the value with `printf '%s' ... | tr -d '\n\r'` before each write.

Locations:

- `action.yml:72`
- `action.yml:73`

### script-injection (severity: high)

Sub-rule (b): in the 'Fetch zip from the plugin directory' step, the TIMEOUT env var comes from inputs.timeout and is expanded without quotes inside shell arithmetic: `max_attempts=$(( ${TIMEOUT} * $per_minute ))`. Bash evaluates the arithmetic, so a crafted value such as `a[$(cmd)]` would run a command. Check that TIMEOUT is numeric (for example `[[ "$TIMEOUT" =~ ^[0-9]+$ ]]`) before using it. The same step also expands `$PLUGIN` and `$VERSION` without quotes in `zipurl=${zipurl//%plugin%/$PLUGIN}` and `zipurl=${zipurl//%version%/$VERSION}`.

Locations:

- `action.yml:79`
- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

action.yml 'Fetch zip' step: put PLUGIN/VERSION in double quotes inside the pattern substitutions; stripped \n\r from zipurl and from the derived PLUGIN_HOST before writing them to GITHUB_ENV/GITHUB_OUTPUT; checked that TIMEOUT matches ^[0-9]+$ before the arithmetic and used it with base 10 (10#$TIMEOUT) so a leading zero isn't read as octal.

