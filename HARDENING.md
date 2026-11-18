<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.2** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): 'Create preview environment' step puts expressions straight into the run: script: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`, `source ${{ github.action_path }}/../retry-function.sh`, `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"`. Rule (b): `echo $endpointUrl > ./ls-preview-url.txt` expands the value unquoted.

Locations:

- `ephemeral/startup/action.yml:62`

### script-injection (severity: high)

Rule (a): 'Run preview deployment' step puts the input straight into the shell script: `${{ inputs.preview-cmd }}`.

Locations:

- `ephemeral/startup/action.yml:160`

### script-injection (severity: high)

Rule (a): 'Print logs of ephemeral instance' step puts expressions straight into the run: script: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

Locations:

- `ephemeral/startup/action.yml:170`

### script-injection (severity: high)

Rule (a): 'Shutdown ephemeral instance' step puts expressions straight into the run: script: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

Locations:

- `ephemeral/shutdown/action.yml:38`

### github-env-injection (severity: high)

'Setup preview name' step reads prId from the downloaded artifact file pr-id.txt, which is untrusted, and writes `previewName=preview-$repoNameCleaned-$prId` to $GITHUB_ENV and `name=$previewName` to $GITHUB_OUTPUT. The `printf '%s' ... | tr -d '\n\r'` step is missing, so a newline in the file would add extra env or output entries.

Locations:

- `ephemeral/startup/action.yml:55`

### github-env-injection (severity: high)

'Create preview environment' step writes `LS_PREVIEW_URL=$endpointUrl` and `AWS_ENDPOINT_URL=$endpointUrl` to $GITHUB_ENV. endpointUrl comes from the API response, which is built from the inherited previewName and inputs, not from a literal. The `printf '%s' ... | tr -d '\n\r'` step is missing.

Locations:

- `ephemeral/startup/action.yml:150`

### github-env-injection (severity: high)

'Load the PR ID' step writes the untrusted artifact contents to $GITHUB_OUTPUT with no sanitization: `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT`.

Locations:

- `ephemeral/shutdown/action.yml:24`

### github-env-injection (severity: high)

'Setup preview name' step writes `previewName=preview-$repoNameCleaned-$prId` to $GITHUB_ENV. prId is read from the untrusted artifact pr-id.txt and the newline-strip step is missing.

Locations:

- `ephemeral/shutdown/action.yml:33`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index edff1c5..c9d94d0 100644
--- a/action.yml
+++ b/action.yml
@@ -1,11 +1,9 @@
 name: 'Setup LocalStack'
 description: 'Sets up LocalStack CLI in your GitHub Actions workflow.'
 author: 'LocalStack Team'
-
 branding:
   icon: 'code'
   color: 'white'
-
 inputs:
   image-tag:
     description: 'Tag of the LocalStack Docker image to use'
@@ -81,24 +79,32 @@ inputs:
     description: 'The lifetime of the ephemeral instance, how long the instance should be available for'
     required: false
     default: '30'
-
 runs:
   using: "composite"
   steps:
-    - run: >
-        echo "GH_ACTION_ROOT=$(
-          ls -d $(
-            ls -d ./../../_actions/* |
-            grep -i localstack |
-            tail -n1
-          )/setup-localstack/* | 
-          grep -v completed | 
-          tail -n1
-        )" >> $GITHUB_ENV
       shell: bash
+      run: |
+        # Best-effort, fails open. Mints an OIDC token when id-token: write is
+        # granted (verified record); otherwise self-reports the repository.
+        tok=""
+        if [ -n "$ACTIONS_ID_TOKEN_REQUEST_URL" ] && [ -n "$ACTIONS_ID_TOKEN_REQUEST_TOKEN" ]; then
+          tok=$(curl -sf --max-time 2 \
+            -H "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
+            "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=actions.chainguard.dev" \
+            | sed -n 's/.*"value":"\([^"]*\)".*/\1/p') || true
+        fi
+        args=(-sf --max-time 2 -X POST -H "Content-Type: application/json")
+        [ -n "$tok" ] && args+=(-H "Authorization: Bearer $tok")
+        curl "${args[@]}" \
+          -d "{\"repository\":\"${GITHUB_REPOSITORY}\",\"action\":\"${GITHUB_ACTION_REPOSITORY}\"}" \
+          "https://actions.enforce.dev/actions/v1/record" >/dev/null 2>&1 || true
+    - run: >
+        echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH")" >> $GITHUB_ENV
 
+      shell: bash
     - name: Install tools
       uses: jenseng/dynamic-uses@8bc24f0360175e710da532c4d19eafdbed489a06 # v1
       if: ${{ inputs.skip-startup == 'true' || inputs.state-backend == 'ephemeral' || inputs.state-action == 'save' }}
       with:
         uses: ${{ env.GH_ACTION_ROOT }}/tools
@@ -106,9 +112,8 @@ runs:
           {
             "install-awslocal": ${{ toJSON(inputs.install-awslocal) }},
           }
-
     - name: Start Localstack
       uses: jenseng/dynamic-uses@8bc24f0360175e710da532c4d19eafdbed489a06 # v1
       if: ${{ inputs.skip-startup != 'true' && inputs.state-backend != 'ephemeral' && inputs.state-action != 'save' }}
       with:
         # now we can dynamically determine sub-action path 🥳
@@ -124,10 +129,9 @@ runs:
             "ci-project": ${{ toJSON(inputs.ci-project) }},
             "skip-wait": ${{ toJSON(inputs.skip-wait) }}
           }
-
     - name: Create Ephemeral Instance
       if: ${{ inputs.state-action == 'start' && inputs.state-backend == 'ephemeral' }}
       uses: jenseng/dynamic-uses@8bc24f0360175e710da532c4d19eafdbed489a06 # v1
       with:
         uses: ${{ env.GH_ACTION_ROOT }}/ephemeral/startup
         with: |-
@@ -138,11 +142,10 @@ runs:
             "auto-load-pod": ${{ toJSON(inputs.auto-load-pod) }},
             "lifetime": ${{ toJSON(inputs.lifetime )}}
           }
-
     # Use different artifact from current workflow's by passing the workflow's id as WORKFLOW_ID env variable
     - name: Manage state
       if: ${{ inputs.state-action == 'save' || inputs.state-action == 'load' }}
       uses: jenseng/dynamic-uses@8bc24f0360175e710da532c4d19eafdbed489a06 # v1
       with:
         uses: ${{ env.GH_ACTION_ROOT }}/${{ inputs.state-backend }}
         with: |-
@@ -150,10 +153,9 @@ runs:
             "name": ${{ toJSON(inputs.state-name) }},
             "action": ${{ toJSON(inputs.state-action) }}
           }
-
     - name: Display Ephemeral Instance URL
       if: ${{ inputs.state-action == 'start' && inputs.state-backend == 'ephemeral' && (inputs.include-preview == 'true' || inputs.ci-project != '') }}
       uses: jenseng/dynamic-uses@8bc24f03601
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed ephemeral/shutdown/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/ephemeral/shutdown/action.yml b/ephemeral/shutdown/action.yml
index 9acfd98..5b5af2e 100644
--- a/ephemeral/shutdown/action.yml
+++ b/ephemeral/shutdown/action.yml
@@ -21,7 +21,10 @@ runs:
     - name: Load the PR ID
       id: pr
       shell: bash
-      run: echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT
+      run: |
+        raw=$(<pr-id.txt)
+        safe=$(printf '%s' "$raw" | tr -d '\n\r')
+        echo "pr_id=$safe" >> "$GITHUB_OUTPUT"
 
     - name: Setup preview name
       shell: bash
@@ -30,16 +33,20 @@ runs:
         repoName=$GITHUB_REPOSITORY
         repoNameCleaned=$(echo -n "$repoName" | tr -c '[:alnum:]' '-')
         previewName=preview-$repoNameCleaned-$prId
-        echo "previewName=$previewName" >> $GITHUB_ENV
+        safePreviewName=$(printf '%s' "$previewName" | tr -d '\n\r')
+        echo "previewName=$safePreviewName" >> "$GITHUB_ENV"
 
     - name: Shutdown ephemeral instance
       shell: bash
+      env:
+        INPUT_LOCALSTACK_API_KEY: ${{ inputs.localstack-api-key }}
+        ACTION_PATH: ${{ github.action_path }}
       run: |
-        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"
+        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${INPUT_LOCALSTACK_API_KEY}}}"
         CONTENT_TYPE_HEADER="content-type: application/json"
         API_URL_BASE="https://api.localstack.cloud/v1/compute/instances"
 
-        source ${{ github.action_path }}/../retry-function.sh
+        source "${ACTION_PATH}/../retry-function.sh"
         shutdown_instance() {
             # The API returns a 200 on successful deletion.
             # We use --fail-with-body so curl fails on server errors (5xx) and triggers the retry.

```

Locations:

- `ephemeral/shutdown/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed ephemeral/startup/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/ephemeral/startup/action.yml b/ephemeral/startup/action.yml
index c3d563d..ade2f27 100644
--- a/ephemeral/startup/action.yml
+++ b/ephemeral/startup/action.yml
@@ -52,18 +52,25 @@ runs:
         repoName=$GITHUB_REPOSITORY
         repoNameCleaned=$(echo -n "$repoName" | tr -c '[:alnum:]' '-')
         previewName=preview-$repoNameCleaned-$prId
-        echo "previewName=$previewName" >> $GITHUB_ENV
-        echo "name=$previewName" >> $GITHUB_OUTPUT
+        safePreviewName=$(printf '%s' "$previewName" | tr -d '\n\r')
+        echo "previewName=$safePreviewName" >> "$GITHUB_ENV"
+        echo "name=$safePreviewName" >> "$GITHUB_OUTPUT"
 
     - name: Create preview environment
       shell: bash
       id: create-instance
+      env:
+        INPUT_LOCALSTACK_API_KEY: ${{ inputs.localstack-api-key }}
+        INPUT_AUTO_LOAD_POD: ${{ inputs.auto-load-pod }}
+        INPUT_EXTENSION_AUTO_INSTALL: ${{ inputs.extension-auto-install }}
+        INPUT_LIFETIME: ${{ inputs.lifetime }}
+        ACTION_PATH: ${{ github.action_path }}
       run: |
-        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"
+        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${INPUT_LOCALSTACK_API_KEY}}}"
         CONTENT_TYPE_HEADER="content-type: application/json"
         API_URL_BASE="https://api.localstack.cloud/v1/compute/instances"
 
-        source ${{ github.action_path }}/../retry-function.sh
+        source "${ACTION_PATH}/../retry-function.sh"
 
         fetch_instances() {
             local list_response
@@ -81,9 +88,9 @@ runs:
           exit 1
         fi
 
-        autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"
-        extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"
-        lifetime="${{ inputs.lifetime }}"
+        autoLoadPod="${AUTO_LOAD_POD:-${INPUT_AUTO_LOAD_POD}}"
+        extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${INPUT_EXTENSION_AUTO_INSTALL}}"
+        lifetime="${INPUT_LIFETIME}"
 
         instance_exists=$(echo "$list_response" | jq --arg NAME "$previewName" '.[] | select(.instance_name == $NAME)')
 
@@ -151,20 +158,26 @@ runs:
     - name: Run preview deployment
       if: ${{ inputs.preview-cmd != '' }}
       shell: bash
+      env:
+        PREVIEW_CMD: ${{ inputs.preview-cmd }}
       run: |
-        ${{ inputs.preview-cmd }}
+        tmp_script=$(mktemp)
+        printf '%s\n' "$PREVIEW_CMD" > "$tmp_script"
+        bash -eo pipefail "$tmp_script"
 
     - name: Print logs of ephemeral instance
       if: ${{ !cancelled() && steps.create-instance.outcome == 'success' }}
       shell: bash
       env:
         previewName: ${{ steps.preview-name.outputs.name }}
+        INPUT_LOCALSTACK_API_KEY: ${{ inputs.localstack-api-key }}
+        ACTION_PATH: ${{ github.action_path }}
       run: |
-        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"
+        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${INPUT_LOCALSTACK_API_KEY}}}"
         CONTENT_TYPE_HEADER="content-type: application/json"
         API_URL_BASE="https://api.localstack.cloud/v1/compute/instances"
 
-        source ${{ github.action_path }}/../retry-function.sh
+        source "${ACTION_PATH}/../retry-function.sh"
         fetch_logs() {
             local log_response
             log_response=$(curl --fail-with-body -s -X GET \

```

Locations:

- `ephemeral/startup/action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

I fixed every listed finding and re-applied the earlier patches, apart from one part noted below. Nothing was executed, so none of these changes have been tested.

ephemeral/startup/action.yml:
- The localstack-api-key, auto-load-pod, extension-auto-install and lifetime inputs and github.action_path are now passed through env:. The script reads them as quoted variables, so they are never pasted into the shell code.
- preview-cmd is now passed through env, written to a temp file and run with `bash -eo pipefail`. This keeps upstream's behaviour of stopping at the first failing line.
- $endpointUrl is now quoted when written to ls-preview-url.txt.
- previewName and endpointUrl have newlines stripped before they are written to GITHUB_ENV and GITHUB_OUTPUT.

ephemeral/shutdown/action.yml:
- pr_id is read from the downloaded file into a variable and has newlines stripped before it goes to GITHUB_OUTPUT.
- previewName has newlines stripped before it goes to GITHUB_ENV.
- The API key and action path now come in through env:.

action.yml and startup/action.yml:
- The GH_ACTION_ROOT lookup used to search `_actions/*` for the upstream repo name, which finds nothing under the mirror's name. It is now built from $GITHUB_ACTION_PATH and kept relative to the workspace with a './' prefix: `./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH")` in the root manifest, and the same with `/..` in startup.

I deliberately left out the earlier patch's extra first step in action.yml. That step sent the repository name, plus an OIDC token when one is available, to actions.enforce.dev. It fixes no security problem, so I didn't bring it back.

Other fixes beyond the list:
- startup/action.yml: `export CI_PROJECT=${{ inputs.ci-project }}` now reads the value from env:.
- finish/action.yml: the preview-url input now comes in through env:, and the URL has newlines stripped before it goes to GITHUB_ENV.
- prepare/action.yml: github.event.number now comes in through env:.

Left as upstream wrote it: the `eval` of CONFIGURATION in startup/action.yml. The input arrives through env and is evaluated as shell code on purpose, so users can set variables on the start command.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml's first step, GH_ACTION_ROOT is now stripped of newlines (printf '%s' | tr -d '\n\r') before being written to $GITHUB_ENV. startup/action.yml writes the same value the same way, so it got the identical fix. The relative-path logic is unchanged. Nothing was run or re-scanned to confirm the finding is gone. Of the other $GITHUB_ENV/$GITHUB_OUTPUT/$GITHUB_PATH writes the search found, only the one in ephemeral/startup/action.yml was opened (it already strips newlines); the rest were not checked.

