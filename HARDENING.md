<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.6.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Debug' step directly interpolates `${{ toJSON(inputs) }}` inside a `run:` shell command. The Actions runner substitutes this expression before the shell executes it, so an attacker-controlled input value (e.g. containing a single quote or shell metacharacter) can break out of the surrounding single-quote context and inject arbitrary shell commands.

Locations:

- `action.yml:34`

### script-injection (severity: high)

Rule (b): In the 'Fetch ZIP from the plugin directory' step, the env vars `$PLUGIN` (inputs.plugin), `$VERSION` (inputs.version), and `$ZIP_URL` (inputs.zip-url) are used unquoted inside shell parameter substitution expressions: `zipurl=${zipurl//%plugin%/$PLUGIN}` and `zipurl=${zipurl//%version%/$VERSION}`. Additionally, `$TIMEOUT` (inputs.timeout) is used unquoted in arithmetic. Unquoted expansion of workflow-controllable env vars allows shell metacharacter injection.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Fetch ZIP from the plugin directory' step writes `PLUGIN_HOST` to `$GITHUB_ENV` using: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"`. The value of `$zipurl` is derived from the untrusted inputs `$ZIP_URL` (inputs.zip-url), `$PLUGIN` (inputs.plugin), and `$VERSION` (inputs.version). No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing an attacker to inject arbitrary environment variable assignments into `$GITHUB_ENV` via newline characters in the input values.

Locations:

- `action.yml:55`

### unpinned-uses (severity: high)

The step 'Generate attestation for the ZIP' uses `actions/attest-build-provenance@v1`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling a supply-chain attack.

Locations:

- `action.yml:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. script-injection (Debug step): Moved `${{ toJSON(inputs) }}` to env var `ALL_INPUTS` and referenced it as `"$ALL_INPUTS"` in the shell script.
2. script-injection (Fetch ZIP step): Quoted `$PLUGIN` and `$VERSION` inside bash parameter substitution expressions; used bare variable names in arithmetic context for `TIMEOUT` and `sleep`.
3. github-env-injection: Sanitized `PLUGIN_HOST` value with `tr -d '\n\r'` before writing to `$GITHUB_ENV` to prevent newline injection.
4. unpinned-uses: Pinned `actions/attest-build-provenance@v1` to full commit SHA `ef244123eb79f2f7a7e75d99086184180e6d0018` with `# v1` comment.

