<!-- markdownlint-disable -->

# Hardening Report: johnbillion--action-wordpress-plugin-attestation/0.7.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **johnbillion--action-wordpress-plugin-attestation/0.7.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Fetch zip from the plugin directory' builds `zipurl` from untrusted inputs (inputs.zip-url, inputs.plugin, inputs.version, passed in through ZIP_URL/PLUGIN/VERSION env vars). It then writes values derived from it to special files with no `printf '%s' ... | tr -d '\n\r'` sanitization: `echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"` (line 72) and `echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"` (line 73). If a value contains a newline, it can add arbitrary env vars or outputs.

Locations:

- `action.yml:72`
- `action.yml:73`

### script-injection (severity: high)

Rule (b) in step 'Fetch zip from the plugin directory': workflow-controlled env vars are expanded unquoted. `max_attempts=$(( ${TIMEOUT} * $per_minute ))` (line 79) puts inputs.timeout into bash arithmetic unquoted. Bash evaluates arithmetic recursively, so a value like `a[$(cmd)]` runs commands. Also `zipurl=${zipurl//%plugin%/$PLUGIN}` (line 69) and `zipurl=${zipurl//%version%/$VERSION}` (line 70) expand inputs.plugin and inputs.version unquoted in the replacement string. Quote these expansions, and check that TIMEOUT is numeric before using it in arithmetic.

Locations:

- `action.yml:79`
- `action.yml:69`
- `action.yml:70`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index f70dda8..04436a6 100644
--- a/action.yml
+++ b/action.yml
@@ -55,6 +55,23 @@ outputs:
 runs:
   using: composite
   steps:
     # This fetches the zipped plugin from the plugin directory. The zip might not exist yet if the plugin uses release confirmation
     # and the release hasn't been confirmed. This will retry until the zip is available or the timeout is reached.
     - name: Fetch zip from the plugin directory
@@ -66,17 +83,25 @@ runs:
         TIMEOUT: ${{ inputs.timeout }}
       run: | #shell
         zipurl="$ZIP_URL"
-        zipurl=${zipurl//%plugin%/$PLUGIN}
-        zipurl=${zipurl//%version%/$VERSION}
+        zipurl="${zipurl//%plugin%/"$PLUGIN"}"
+        zipurl="${zipurl//%version%/"$VERSION"}"
+        zipurl="$(printf '%s' "$zipurl" | tr -d '\n\r')"
 
-        echo PLUGIN_HOST="$(echo "$zipurl" | awk -F/ '{print $3}')" >> "$GITHUB_ENV"
+        plugin_host="$(printf '%s' "$zipurl" | awk -F/ '{print $3}')"
+        plugin_host="$(printf '%s' "$plugin_host" | tr -d '\n\r')"
+        echo PLUGIN_HOST="$plugin_host" >> "$GITHUB_ENV"
         echo zip-url="$zipurl" >> "$GITHUB_OUTPUT"
 
+        if ! [[ "$TIMEOUT" =~ ^[0-9]+$ ]]; then
+          echo "Error: timeout must be a non-negative integer number of minutes."
+          exit 1
+        fi
+
         echo "Fetching plugin zip from $zipurl ..."
         attempts=0
         sleep=10
-        per_minute=$((60 / $sleep))
-        max_attempts=$(( ${TIMEOUT} * $per_minute ))
+        per_minute=$((60 / sleep))
+        max_attempts=$(( 10#$TIMEOUT * per_minute ))
 
         # The zip can return a 200 before every server behind the download host can serve it, so a download only
         # counts once it's a valid zip.
@@ -130,7 +155,7 @@ runs:
 
     - name: Upload the zip from the plugin directory for debugging
       if: ${{ failure() }}
       uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
       with:
         name: ${{ inputs.plugin }}-${{ inputs.version }}-plugin-directory-zip
         path: ${{ github.workspace }}/${{ inputs.plugin }}.zip
@@ -139,7 +164,7 @@ runs:
     - name: Generate attestation for the zip
       if: ${{ inputs.dry-run == 'false' }}
       id: generate-attestation
       uses: actions/attest@59d89421af93a897026c735860bf21b6eb4f7b26 # v4.1.0
       env:
         PLUGIN: ${{ inputs.plugin }}
         VERSION: ${{ inputs.version }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

In action.yml, the 'Fetch zip from the plugin directory' step changed in four ways, which re-applies the previous harden's patch. (1) The %plugin% and %version% substitutions are now quoted, so the plugin and version inputs no longer expand unquoted. (2) Newlines and carriage returns are stripped from zipurl, and separately from PLUGIN_HOST, before either is written to GITHUB_OUTPUT or GITHUB_ENV. (3) TIMEOUT is checked to be a plain whole number before the arithmetic; anything else stops the step with an error. (4) The arithmetic forces base 10 (10#) so the timeout value is never evaluated as an expression. Nothing was run to test this. Behaviour change: a non-integer timeout such as '1.5' used to fail later in the arithmetic and now stops at the new check.

