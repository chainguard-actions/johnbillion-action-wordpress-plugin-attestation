<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `zipurl` is constructed from `$ZIP_URL`, `$PLUGIN`, and `$VERSION` — all sourced from `inputs.*` (attacker-controllable). Two unsanitized writes occur:
1. `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` — writes a value derived from inputs into GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization. A newline embedded in any of the inputs could inject arbitrary environment variables.
2. `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"` — similarly writes an unsanitized input-derived value to GITHUB_OUTPUT, enabling output injection.

Locations:

- `action.yml:64`
- `action.yml:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two unsanitized writes in the 'Fetch zip from the plugin directory' step:
1. `PLUGIN_HOST` write to `$GITHUB_ENV`: replaced `echo PLUGIN_HOST="$(echo "$zipurl" | awk ...)"` with a two-step approach that captures the awk output, strips newlines via `tr -d '\n\r'`, then writes using `printf` to GITHUB_ENV.
2. `zip-url` write to `$GITHUB_OUTPUT`: replaced `echo zip-url="$zipurl"` with a sanitized version that strips newlines via `tr -d '\n\r'` before writing using `printf` to GITHUB_OUTPUT.
Both values are derived from attacker-controllable inputs (inputs.plugin, inputs.version, inputs.zip-url), so stripping newlines prevents injection of arbitrary environment variables or step outputs.

