<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Start LocalStack' step breaks rule (a) and rule (b). (a) The expression is placed straight into the shell script: `export CI_PROJECT=${{ inputs.ci-project }}`. A ci-project value containing shell metacharacters (e.g. `; curl ... | sh`) runs as code. Fix: pass it through env (CI_PROJECT: ${{ inputs.ci-project }}) and drop the export, or use `export CI_PROJECT="$CI_PROJECT_INPUT"`. (b) Workflow-controlled variables are expanded without quotes: `docker pull ${IMAGE_NAME} &` (IMAGE_NAME comes from inputs.image-tag or an inherited IMAGE_NAME env var) and `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` (LS_WAIT_TIMEOUT is inherited from the calling workflow). Both should be double-quoted. Also, `eval "${CONFIGURATION} localstack start -d"` re-parses inputs.configuration and IMAGE_NAME as shell code, so quoting does not help there. Any metacharacter in the configuration input or image tag is executed.

Locations:

- `action.yml:80`
- `action.yml:79`
- `action.yml:81`
- `action.yml:84`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:80`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/startup/action.yml b/startup/action.yml
index ce39050..f921cb3 100644
--- a/startup/action.yml
+++ b/startup/action.yml
@@ -1,6 +1,5 @@
 name: 'Start up Localstack'
 description: 'Starts up Localstack'
-
 inputs:
   image-tag:
     description: 'Tag of the LocalStack Docker image to use'
@@ -27,10 +26,26 @@ inputs:
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
@@ -40,26 +55,17 @@ runs:
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
@@ -76,14 +82,18 @@ runs:
         fi
 
         CONFIGURATION="IMAGE_NAME=${IMAGE_NAME} ${CONFIGURATION}"
-        docker pull ${IMAGE_NAME} &
-        export CI_PROJECT=${{ inputs.ci-project }}
-        eval "${CONFIGURATION} localstack start -d"
+        docker pull "${IMAGE_NAME}" &
+        export CI_PROJECT="$CI_PROJECT_INPUT"
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
@@ -91,4 +101,4 @@ runs:
         USE_PRO: ${{ inputs.use-pro }}
         CONFIGURATION: ${{ inputs.configuration }}
         SKIP_WAIT: ${{ inputs.skip-wait }}
-
+        CI_PROJECT_INPUT: ${{ inputs.ci-project }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

I fixed the 'Start LocalStack' step and the GH_ACTION_ROOT lookup in action.yml (the startup sub-action). Nothing was run or tested.

Changes:
- inputs.ci-project now goes in through env (CI_PROJECT_INPUT) and is exported quoted as CI_PROJECT, so it is no longer pasted into the script.
- `${IMAGE_NAME}` and `${LS_WAIT_TIMEOUT:-30}` are now double-quoted.
- `eval "${CONFIGURATION} localstack start -d"` is replaced by `env "${cfg_args[@]}" localstack start -d`. The array is built by the quote-aware xargs split: it is skipped when the input is empty, an unmatched quote stops the step, and it uses the shell's built-in printf. Quoted values still split the way eval split them.
- Re-applied the previous harden's fix: GH_ACTION_ROOT no longer searches `_actions/*` for the upstream name. It is now `./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")`, a relative path to the repo root, which the generated `uses:` for tools needs.

Behaviour changes:
- Nothing inside the configuration value is evaluated any more. A configuration that relied on eval to expand `$VAR` or run `$(...)` will now pass that text through literally.
- `env` will read a configuration token that starts with '-' as its own option, which eval did not do.

