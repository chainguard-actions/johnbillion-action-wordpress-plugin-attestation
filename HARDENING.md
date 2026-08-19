<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'fetch-zip' step writes values derived from untrusted inputs to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). The variable `zipurl` is constructed from `inputs.zip-url`, `inputs.plugin`, and `inputs.version` — all caller-controlled. A value containing embedded newlines could inject arbitrary key=value pairs into the runner environment or output context.

Offending lines:
  echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"
  echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"

Fix: sanitize each value before writing, e.g.:
  safe=$(printf '%s' "$zipurl" | tr -d '\n\r')
  echo "zip-url=$safe" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:62`
- `action.yml:63`

### script-injection (severity: high)

Rule (b): Multiple unquoted shell variable expansions of env vars that hold untrusted, caller-controlled values. This allows shell metacharacters (spaces, globs, semicolons, etc.) in the input to be interpreted by the shell.

1. `zipurl=${zipurl//%plugin%/$PLUGIN}` — `$PLUGIN` (from inputs.plugin) is unquoted inside a parameter substitution (action.yml, fetch-zip step).
2. `zipurl=${zipurl//%version%/$VERSION}` — `$VERSION` (from inputs.version) is unquoted inside a parameter substitution (action.yml, fetch-zip step).
3. `curl --silent --output "${PLUGIN}.zip" "$zipurl"` — `${PLUGIN}` is not surrounded by double-quotes; only the `.zip` suffix is quoted (action.yml, fetch-zip step).
4. `unzip -q -d zip-deployed "${PLUGIN}.zip"` — same unquoted `${PLUGIN}` pattern (action.yml, unzip-deployed step).
5. `unzip -q -d zip-generated "${ZIP_PATH}"` — `${ZIP_PATH}` (from inputs.zip-path) is not surrounded by double-quotes (action.yml, unzip-generated step).

Fix: wrap each expansion in double-quotes, e.g. `"${PLUGIN}"`, `"${VERSION}"`, `"${ZIP_PATH}"`.

Locations:

- `action.yml:59`
- `action.yml:60`
- `action.yml:78`
- `action.yml:88`
- `action.yml:93`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed both findings in hardened/action/action.yml:

1. github-env-injection: Added sanitization before writing to $GITHUB_ENV and $GITHUB_OUTPUT. Introduced `safe_zipurl=$(printf '%s' "$zipurl" | tr -d '\n\r')` and `safe_host=$(printf '%s' "$zipurl" | awk -F/ '{print $3}' | tr -d '\n\r')`, then used these sanitized variables in the echo statements instead of the raw `$zipurl`.

2. script-injection: Fixed unquoted variable expansions in parameter substitutions — changed `${zipurl//%plugin%/$PLUGIN}` to `${zipurl//%plugin%/"$PLUGIN"}` and `${zipurl//%version%/$VERSION}` to `${zipurl//%version%/"$VERSION"}`. The other reported locations (`${PLUGIN}.zip` and `${ZIP_PATH}`) were already inside double-quoted strings in the original file.

