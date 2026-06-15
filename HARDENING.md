<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Fetch ZIP from the plugin directory' step, the variable `zipurl` is constructed from caller-controlled inputs (`$ZIP_URL`, `$PLUGIN`, `$VERSION`, all sourced from `inputs.*`) and then written unsanitized to `$GITHUB_ENV` (as `PLUGIN_HOST`) and `$GITHUB_OUTPUT` (as `zip-url`). An attacker-supplied input containing newline characters could inject arbitrary key=value pairs into the runner's environment or output context. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is missing before both writes.

Locations:

- `action.yml:67`
- `action.yml:68`

### unpinned-uses (severity: high)

The composite action step 'Generate attestation for the ZIP' references `actions/attest-build-provenance@v2`, which uses a mutable version tag (`v2`) instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to a different (potentially malicious) commit.

Locations:

- `action.yml:121`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses

**Notes:**

Fixed two security findings in action.yml: (1) github-env-injection: Added sanitization of the `zipurl` variable (derived from caller-controlled inputs) before writing to $GITHUB_ENV and $GITHUB_OUTPUT using `printf '%s' "$VAR" | tr -d '\n\r'` to strip newline characters that could inject arbitrary key=value pairs. (2) unpinned-uses: Pinned `actions/attest-build-provenance@v2` to its full commit SHA `e8998f949152b193b063cb0ec769d69d929409be` with the original tag preserved as a comment.

