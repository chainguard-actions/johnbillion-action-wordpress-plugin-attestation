<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `zipurl` is constructed from `$ZIP_URL` (which holds `inputs.zip-url`, a caller-controlled input) and is written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled `zip-url` input containing newline characters could inject arbitrary environment variables into subsequent steps via `$GITHUB_ENV`. The offending line is: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`

Locations:

- `action.yml:68`

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable `zipurl` (derived from `inputs.zip-url` via `$ZIP_URL`) is written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller-controlled `zip-url` input containing newline characters could inject arbitrary key-value pairs into `$GITHUB_OUTPUT`, potentially poisoning downstream step outputs. The offending line is: `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml at the 'Fetch zip from the plugin directory' step. Added sanitization using `printf '%s' ... | tr -d '\n\r'` before writing the caller-controlled zip-url value to both $GITHUB_ENV (as PLUGIN_HOST) and $GITHUB_OUTPUT (as zip-url). The sanitized variables safe_zipurl and safe_host are used in place of the raw $zipurl to prevent newline-based injection attacks.

