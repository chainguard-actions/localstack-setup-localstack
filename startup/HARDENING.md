<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Start LocalStack' step puts an expression straight into the run script: `export CI_PROJECT=${{ inputs.ci-project }}`. The input is pasted into the shell text before bash runs, so a value like `x; malicious_cmd` runs as a command. Fix: add CI_PROJECT: ${{ inputs.ci-project }} to the step's env: block and drop this line, or use `export CI_PROJECT="$CI_PROJECT_INPUT"`.

Locations:

- `action.yml:77`

### script-injection (severity: high)

Rule (b): the 'Start LocalStack' step expands workflow-controlled variables without quotes. IMAGE_NAME can come from the calling workflow's env, and IMAGE_TAG comes from inputs.image-tag. LS_WAIT_TIMEOUT is inherited from the calling workflow. Offending lines: `docker pull ${IMAGE_NAME} &` (line 76) and `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` (line 81). Also, `eval "${CONFIGURATION} localstack start -d"` (line 78) runs inputs.configuration, and an inherited IMAGE_NAME, through eval, so shell syntax in those values gets executed. Fix: quote the expansions ("${IMAGE_NAME}", "${LS_WAIT_TIMEOUT:-30}"). Replace the eval: parse CONFIGURATION into KEY=VALUE pairs and pass them with `env` or `export`.

Locations:

- `action.yml:76`
- `action.yml:78`
- `action.yml:81`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/startup/action.yml b/startup/action.yml
index 740fccc..4408016 100644
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
+        export CI_PROJECT="$CI_PROJECT_INPUT"
+        cfg_args=()
+        if [ -n "$CONFIGURATION" ]; then
+          while IFS= read -r -d '' t; do cfg_args+=("$t"); done \
+            < <(printf '%s' "$CONFIGURATION" | xargs printf '%s\0')
+        fi
+        env "${cfg_args[@]}" localstack start -d
 
         if [ "$SKIP_WAIT" = false ]; then
-          localstack wait -t ${LS_WAIT_TIMEOUT:-30}  # Let users configure the timeout via env variable
+          localstack wait -t "${LS_WAIT_TIMEOUT:-30}"  # Let users configure the timeout via env variable
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
+        CI_PROJECT_INPUT: ${{ inputs.ci-project }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

I fixed all findings in action.yml (the startup manifest).

1. ci-project input: it is now passed through the step's env as CI_PROJECT_INPUT, and the script uses `export CI_PROJECT="$CI_PROJECT_INPUT"`, so the value is no longer pasted into the script text.
2. Unquoted variables: IMAGE_NAME and LS_WAIT_TIMEOUT are now in double quotes in `docker pull` and `localstack wait`.
3. eval: removed. CONFIGURATION is now split into words with xargs, which respects quotes the way the old eval did (an unmatched quote stops the step). The words are passed as `env "${cfg_args[@]}" localstack start -d`, so nothing in the value is run as shell code. One difference from upstream: a `$VAR` written inside the configuration value is no longer expanded.
4. Prior fix re-applied: the step that searched the runner's `_actions` folder for the upstream repo name has been replaced. GH_ACTION_ROOT is now `./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")`, a path relative to the workspace that points at the repo root, which is what the dynamic-uses `tools` step needs. `realpath --relative-to` only exists on Linux runners.

