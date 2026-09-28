<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable $zipurl is constructed from inputs.plugin, inputs.version, and inputs.zip-url (all untrusted caller-controlled inputs) and then written directly to $GITHUB_ENV without sanitization: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`. An attacker can inject newlines into these inputs to write arbitrary key=value pairs into the runner's environment, potentially overriding environment variables used by subsequent steps. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is missing before this write.

Locations:

- `action.yml:72`

### github-env-injection (severity: high)

In the 'Fetch zip from the plugin directory' step, the variable $zipurl is constructed from inputs.plugin, inputs.version, and inputs.zip-url (all untrusted caller-controlled inputs) and then written directly to $GITHUB_OUTPUT without sanitization: `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"`. An attacker can inject newlines into these inputs to write arbitrary key=value pairs into the step output file, potentially poisoning outputs consumed by downstream steps or jobs. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is missing before this write.

Locations:

- `action.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml at lines 72-73. The `$zipurl` value (derived from untrusted caller-controlled inputs: inputs.plugin, inputs.version, and inputs.zip-url) was being written directly to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization. Fixed by introducing `safe_zipurl` and `safe_plugin_host` variables that strip newlines and carriage returns using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to the environment and output files.

