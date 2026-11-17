<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Fetch zip from the plugin directory' writes values built from untrusted inputs to special environment files without stripping newlines first. `zipurl` comes from inputs.zip-url, inputs.plugin and inputs.version. It is written with `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` and `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"`. A newline in any of these inputs would add extra environment variables or outputs. Fix: run each value through `printf '%s' ... | tr -d '\n\r'` before writing it.

Locations:

- `action.yml:72`
- `action.yml:73`

### script-injection (severity: high)

Rule (b) violation in step 'Fetch zip from the plugin directory': the env var TIMEOUT (from inputs.timeout) is expanded unquoted inside arithmetic: `max_retries=$(( ${TIMEOUT} * $per_minute - 1 ))`. Bash evaluates the contents of arithmetic expansion, and a value like `a[$(cmd)]` runs a command. PLUGIN and VERSION are also expanded unquoted in `zipurl=${zipurl//%plugin%/$PLUGIN}` and `zipurl=${zipurl//%version%/$VERSION}`. Fix: check that TIMEOUT is numeric (e.g. `[[ "$TIMEOUT" =~ ^[0-9]+$ ]]`) before using it, and double-quote the replacement expansions.

Locations:

- `action.yml:79`
- `action.yml:69`
- `action.yml:70`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 35882c0..de9a46c 100644
--- a/action.yml
+++ b/action.yml
@@ -4,7 +4,6 @@ author: johnbillion
 branding:
   icon: check-circle
   color: green
-
 inputs:
   zip-path:
     description: The local path to the plugin zip file.
@@ -51,10 +50,26 @@ outputs:
   zip-sha-256:
     description: SHA-256 hash of the downloaded plugin zip file.
     value: ${{ steps.fetch-zip.outputs.zip-sha-256 }}
-
 runs:
   using: composite
   steps:
     # This fetches the zipped plugin from the plugin directory. The zip might not exist yet if the plugin uses release confirmation
     # and the release hasn't been confirmed. This will retry until the zip is available or the timeout is reached.
     - name: Fetch zip from the plugin directory
@@ -69,8 +84,10 @@ runs:
         zipurl=${zipurl//%plugin%/$PLUGIN}
         zipurl=${zipurl//%version%/$VERSION}
 
-        echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"
-        echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"
+        safe_zipurl=$(printf '%s' "$zipurl" | tr -d '\n\r')
+        safe_host=$(printf '%s' "$(echo "$safe_zipurl" | awk -F/ '{print $3}')" | tr -d '\n\r')
+        echo PLUGIN_HOST="$safe_host" >> "$GITHUB_ENV"
+        echo zip-url="$safe_zipurl" >> "$GITHUB_OUTPUT"
 
         echo "Fetching plugin zip from $zipurl ..."
         elapsed=0
@@ -101,7 +118,6 @@ runs:
           exit 1
         fi
       shell: bash
-
     # Now compare the contents of the generated zip and the plugin directory zip to ensure they match.
     # Only then should an attestation for the plugin directory zip be generated.
     - name: Unzip the zip from the plugin directory
@@ -110,30 +126,26 @@ runs:
       run: | #shell
         unzip -q -d zip-deployed "${PLUGIN}.zip"
       shell: bash
-
     - name: Unzip the generated zip
       env:
         ZIP_PATH: ${{ inputs.zip-path }}
       run: | #shell
         unzip -q -d zip-generated "${ZIP_PATH}"
       shell: bash
-
     - name: Ensure the contents are identical
       run: | #shell
         diff --recursive zip-generated zip-deployed
       shell: bash
-
     - name: Generate attestation for the zip
       if: ${{ inputs.dry-run == 'false' }}
       id: generate-attestation
       uses: actions/attest-build-provenance@c074443f1aee8d4aeeae555aebba3282517141b2 # v2.2.3
       env:
         PLUGIN: ${{ inputs.plugin }}
         VERSION: ${{ inputs.version }}
       with:
         subject-path: "${{ github.workspace }}/${{ env.PLUGIN }}.zip"
         subject-name: "${{ env.PLUGIN_HOST }}-${{ env.PLUGIN }}-${{ env.VERSION }}"
-
     - name: Verify the attestation
       if: ${{ inputs.dry-run == 'false' }}
       env:

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

I hardened the 'Fetch zip from the plugin directory' step in action.yml. The plugin and version values are now double-quoted where they replace %plugin% and %version% in the zip URL. The timeout input must now be a whole number (checked against ^[0-9]+$) before it is used in arithmetic; anything else stops the step with an error. The zip URL and the host taken from it now have newlines removed (printf '%s' | tr -d '\n\r') before they are written to GITHUB_OUTPUT and GITHUB_ENV. The awk output is saved to a variable first, so its exit status is not hidden by tr. This re-applies the previous harden's newline fix. Not run or tested here.

