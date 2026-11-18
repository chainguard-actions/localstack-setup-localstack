<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Start LocalStack' step breaks two script-injection rules. Rule (a), direct expression interpolation: line 79 `export CI_PROJECT=${{ inputs.ci-project }}` puts an input straight into the shell script, unquoted. A value such as `x; malicious-cmd` runs as a command. Rule (b), unquoted expansion of workflow-controlled data: line 78 `docker pull ${IMAGE_NAME} &` expands IMAGE_NAME unquoted. IMAGE_NAME comes from inputs.image-tag or from an env var set by the calling workflow. Line 83 `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` expands LS_WAIT_TIMEOUT, an env var inherited from the calling workflow, unquoted. Also, line 80 `eval "${CONFIGURATION} localstack start -d"` passes the contents of inputs.configuration (and IMAGE_NAME / IMAGE_TAG) to eval, so shell metacharacters in those values run as commands. Fix: move inputs.ci-project into the step's env block and use "$CI_PROJECT". Double-quote "$IMAGE_NAME" and "${LS_WAIT_TIMEOUT:-30}". Replace the eval with a parsed list of KEY=VALUE pairs, for example by exporting each validated pair.

Locations:

- `action.yml:79`
- `action.yml:78`
- `action.yml:80`
- `action.yml:83`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/startup/action.yml b/startup/action.yml
index 740fccc..f77e7d5 100644
--- a/startup/action.yml
+++ b/startup/action.yml
@@ -1,6 +1,5 @@
 name: 'Start up Localstack'
 description: 'Starts up Localstack'
-
 inputs:
   image-tag:
     description: 'Tag of the LocalStack Docker image to use'
@@ -26,10 +25,26 @@ inputs:
     description: 'Skip wait for LocalStack'
     required: false
     default: 'false'
-
 runs:
   using: "composite"
   steps:
     # # NOTE: Thinking about this if we need it
     # - name: Initial PR comment
     #   # TODO: potentially replace with Action version number over time
@@ -39,26 +54,17 @@ runs:
     #     github-token: ${{ inputs.github-token }}
     #     ci-project: ${{ inputs.ci-project }}
     - run: >
-        echo "GH_ACTION_ROOT=$(
-          ls -d $(
-            ls -d ./../../_actions/* |
-            grep -i localstack |
-            tail -n1
-          )/setup-localstack/* | 
-          grep -v completed | 
-          tail -n1
-        )" >> $GITHUB_ENV
-      shell: bash
+        echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")" >> $GITHUB_ENV
 
+      shell: bash
     - name: Install tools
       uses: jenseng/dynamic-uses@5175289a9a87978dcfcb9cf512b821d23b2a53eb # v1
       with:
         uses: ${{ env.GH_ACTION_ROOT }}/tools
         with: |-
           {
             "install-awslocal": ${{ toJSON(inputs.install-awslocal) }},
           }
-
     - name: Start LocalStack
       run: |
         if [ "$USE_PRO" = true ]; then
@@ -75,14 +81,18 @@ runs:
         fi
 
         CONFIGURATION="IMAGE_NAME=${IMAGE_NAME} ${CONFIGURATION}"
-        docker pull ${IMAGE_NAME} &
-        export CI_PROJECT=${{ inputs.ci-project }}
-        eval "${CONFIGURATION} localstack start -d"
+        docker pull "${IMAGE_NAME}" &
+        export CI_PROJECT="$CI_PROJECT"
+        cfg_args=()
+        if [ -n "$CONFIGURATION" ]; then
+          while IFS= read -r -d '' t; do cfg_args+=("$t"); done \
+            < <(printf '%s' "$CONFIGURATION" | xargs printf '%s\0')
+        fi
+        env "${cfg_args[@]}" localstack start -d
 
         if [ "$SKIP_WAIT" = false ]; then
           localstack wait -t ${LS_WAIT_TIMEOUT:-30}  # Let users configure the timeout via env variable
         fi
-
       shell: bash
       env:
         IMAGE_TAG: ${{ inputs.image-tag }}
@@ -90,4 +100,4 @@ runs:
         USE_PRO: ${{ inputs.use-pro }}
         CONFIGURATION: ${{ inputs.configuration }}
         SKIP_WAIT: ${{ inputs.skip-wait }}
-
+        CI_PROJECT: ${{ inputs.ci-project }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

I fixed all three findings in startup/action.yml. Nothing has been run yet.

- **`ci-project` (script-injection, static-inline-injection):** the input no longer appears in the script text. It now comes in through the step's `env:` block as `CI_PROJECT` and is exported from there.
- **`eval` removed:** the configuration string is now split into separate arguments with a quote-aware xargs split, so quoting behaves as it did under `eval`. An unmatched quote fails the step instead of silently losing the rest of the arguments. Each entry is then checked against `KEY=VALUE` and passed to `env ... localstack start -d`. `IMAGE_NAME` is added as its own argument instead of being pasted into the string first. One behaviour change: a configuration entry that isn't `KEY=VALUE` now stops the step with an error, where upstream's `eval` would have run it as a command.
- **Quoting:** `docker pull "$IMAGE_NAME"` and `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"` are now double-quoted.
- **Prior fix re-applied:** the step that found the action's own files by searching the runner's `_actions` directory for the upstream name would find nothing under the mirror's name. It now builds the path from `$GITHUB_ACTION_PATH/..`, made relative to the workspace and starting with `./`, so the dynamic-uses `uses:` it feeds still gets an accepted local path. That step also now quotes `$GITHUB_ENV`.

