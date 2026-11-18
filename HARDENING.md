<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Create preview environment' step puts GitHub expressions straight into the run: script, so their values are pasted into the shell code before bash runs. Offending lines: AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"; source ${{ github.action_path }}/../retry-function.sh; autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"; extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"; lifetime="${{ inputs.lifetime }}". Fix: move each value into an env: variable and expand it in double quotes.

Locations:

- `ephemeral/startup/action.yml:62`
- `ephemeral/startup/action.yml:66`
- `ephemeral/startup/action.yml:84`
- `ephemeral/startup/action.yml:85`
- `ephemeral/startup/action.yml:86`

### script-injection (severity: high)

Rule (a): the 'Run preview deployment' step runs `${{ inputs.preview-cmd }}` as the whole run: script. The input is pasted into the shell script, so whatever the caller passes runs as shell code.

Locations:

- `ephemeral/startup/action.yml:155`

### script-injection (severity: high)

Rule (a): the 'Print logs of ephemeral instance' step puts expressions straight into the run: script: AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}" and source ${{ github.action_path }}/../retry-function.sh.

Locations:

- `ephemeral/startup/action.yml:163`
- `ephemeral/startup/action.yml:167`

### script-injection (severity: high)

Rule (a): the 'Shutdown ephemeral instance' step puts expressions straight into the run: script: AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}" and source ${{ github.action_path }}/../retry-function.sh.

Locations:

- `ephemeral/shutdown/action.yml:38`
- `ephemeral/shutdown/action.yml:42`

### github-env-injection (severity: high)

The 'Setup preview name' step builds previewName from prId=$(<pr-id.txt), which comes from a downloaded artifact. It writes the value to $GITHUB_ENV ('echo "previewName=$previewName" >> $GITHUB_ENV') and $GITHUB_OUTPUT ('echo "name=$previewName" >> $GITHUB_OUTPUT'). No printf '%s' | tr -d '\n\r' step runs first. Only the repository name part goes through tr -c '[:alnum:]'; prId does not, so a newline in the file adds extra entries.

Locations:

- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:56`

### github-env-injection (severity: high)

The 'Create preview environment' step writes endpointUrl to $GITHUB_ENV without stripping newlines first: 'echo "LS_PREVIEW_URL=$endpointUrl" >> $GITHUB_ENV' and 'echo "AWS_ENDPOINT_URL=$endpointUrl" >> $GITHUB_ENV'. endpointUrl is taken from an API response to a request built from inputs (auto-load-pod, extension-auto-install, lifetime).

Locations:

- `ephemeral/startup/action.yml:142`
- `ephemeral/startup/action.yml:143`

### github-env-injection (severity: high)

The 'Load the PR ID' step writes the raw contents of the downloaded pr-id.txt artifact to $GITHUB_OUTPUT without sanitizing: run: echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT.

Locations:

- `ephemeral/shutdown/action.yml:24`

### github-env-injection (severity: high)

The 'Setup preview name' step builds previewName from prId=$(<pr-id.txt), which comes from an artifact. It writes it with 'echo "previewName=$previewName" >> $GITHUB_ENV' and no printf '%s' | tr -d '\n\r' step first.

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
index 9acfd98..4246cdd 100644
--- a/ephemeral/shutdown/action.yml
+++ b/ephemeral/shutdown/action.yml
@@ -21,7 +21,10 @@ runs:
     - name: Load the PR ID
       id: pr
       shell: bash
-      run: echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT
+      run: |
+        raw_pr_id=$(<pr-id.txt)
+        safe_pr_id=$(printf '%s' "$raw_pr_id" | tr -d '\n\r')
+        echo "pr_id=$safe_pr_id" >> "$GITHUB_OUTPUT"
 
     - name: Setup preview name
       shell: bash
@@ -30,16 +33,20 @@ runs:
         repoName=$GITHUB_REPOSITORY
         repoNameCleaned=$(echo -n "$repoName" | tr -c '[:alnum:]' '-')
         previewName=preview-$repoNameCleaned-$prId
-        echo "previewName=$previewName" >> $GITHUB_ENV
+        safe_previewName=$(printf '%s' "$previewName" | tr -d '\n\r')
+        echo "previewName=$safe_previewName" >> "$GITHUB_ENV"
 
     - name: Shutdown ephemeral instance
       shell: bash
+      env:
+        LOCALSTACK_API_KEY_INPUT: ${{ inputs.localstack-api-key }}
+        ACTION_PATH: ${{ github.action_path }}
       run: |
-        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"
+        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-$LOCALSTACK_API_KEY_INPUT}}"
         CONTENT_TYPE_HEADER="content-type: application/json"
         API_URL_BASE="https://api.localstack.cloud/v1/compute/instances"
 
-        source ${{ github.action_path }}/../retry-function.sh
+        source "$ACTION_PATH/../retry-function.sh"
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
index c3d563d..8b139e6 100644
--- a/ephemeral/startup/action.yml
+++ b/ephemeral/startup/action.yml
@@ -52,18 +52,25 @@ runs:
         repoName=$GITHUB_REPOSITORY
         repoNameCleaned=$(echo -n "$repoName" | tr -c '[:alnum:]' '-')
         previewName=preview-$repoNameCleaned-$prId
-        echo "previewName=$previewName" >> $GITHUB_ENV
-        echo "name=$previewName" >> $GITHUB_OUTPUT
+        safe_previewName=$(printf '%s' "$previewName" | tr -d '\n\r')
+        echo "previewName=$safe_previewName" >> "$GITHUB_ENV"
+        echo "name=$safe_previewName" >> "$GITHUB_OUTPUT"
 
     - name: Create preview environment
       shell: bash
       id: create-instance
+      env:
+        LOCALSTACK_API_KEY_INPUT: ${{ inputs.localstack-api-key }}
+        ACTION_PATH: ${{ github.action_path }}
+        AUTO_LOAD_POD_INPUT: ${{ inputs.auto-load-pod }}
+        EXTENSION_AUTO_INSTALL_INPUT: ${{ inputs.extension-auto-install }}
+        LIFETIME_INPUT: ${{ inputs.lifetime }}
       run: |
-        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"
+        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-$LOCALSTACK_API_KEY_INPUT}}"
         CONTENT_TYPE_HEADER="content-type: application/json"
         API_URL_BASE="https://api.localstack.cloud/v1/compute/instances"
 
-        source ${{ github.action_path }}/../retry-function.sh
+        source "$ACTION_PATH/../retry-function.sh"
 
         fetch_instances() {
             local list_response
@@ -81,9 +88,9 @@ runs:
           exit 1
         fi
 
-        autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"
-        extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"
-        lifetime="${{ inputs.lifetime }}"
+        autoLoadPod="${AUTO_LOAD_POD:-$AUTO_LOAD_POD_INPUT}"
+        extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-$EXTENSION_AUTO_INSTALL_INPUT}"
+        lifetime="$LIFETIME_INPUT"
 
         instance_exists=$(echo "$list_response" | jq --arg NAME "$previewName" '.[] | select(.instance_name == $NAME)')
 
@@ -138,9 +145,10 @@ runs:
 
         echo "Created preview environment with endpoint URL: $endpointUrl"
 
-        echo $endpointUrl > ./ls-preview-url.txt
-        echo "LS_PREVIEW_URL=$endpointUrl" >> $GITHUB_ENV
-        echo "AWS_ENDPOINT_URL=$endpointUrl" >> $GITHUB_ENV
+        echo "$endpointUrl" > ./ls-preview-url.txt
+        safe_endpointUrl=$(printf '%s' "$endpointUrl" | tr -d '\n\r')
+        echo "LS_PREVIEW_URL=$safe_endpointUrl" >> "$GITHUB_ENV"
+        echo "AWS_ENDPOINT_URL=$safe_endpointUrl" >> "$GITHUB_ENV"
 
     - name: Upload preview instance URL
       uses: actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6
@@ -151,20 +159,26 @@ runs:
     - name: Run preview deployment
       if: ${{ inputs.preview-cmd != '' }}
       shell: bash
+      env:
+        PREVIEW_CMD: ${{ inputs.preview-cmd }}
       run: |
-        ${{ inputs.preview-cmd }}
+        _preview_script=$(mktemp)
+        printf '%s\n' "$PREVIEW_CMD" > "$_preview_script"
+        bash -eo pipefail "$_preview_script"
 
     - name: Print logs of ephemeral instance
       if: ${{ !cancelled() && steps.create-instance.outcome == 'success' }}
       shell: bash
       env:
         previewName: ${{ steps.preview-name.outputs.name }}
+        LOCALSTACK_API_KEY_INPUT: ${{ inputs.localstack-api-key }}
+        ACTION_PATH: ${{ github.action_path }}
       run: |
-        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"
+        AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-$LOCALSTACK_API_KEY_INPUT}}"
         CONTENT_TYPE_HEADER="content-type: application/json"
         API_URL_BASE="https://api.localstack.cloud/v1/compute/instances"
 
-        source ${{ github.action_path }}/../retry-function.sh
+        source "$ACTION_PATH/../retry-function.sh"
         fetch_logs() {
             loca
... (truncated)
```

Locations:

- `ephemeral/startup/action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

ephemeral/startup/action.yml: moved inputs.localstack-api-key, github.action_path, auto-load-pod, extension-auto-install and lifetime into step env vars and expanded them in double quotes (create-instance and print-logs steps). preview-cmd now goes through env PREVIEW_CMD, gets written to a temp file and runs with 'bash -eo pipefail', so a failing line still stops the step the way the inlined original did. Newlines are now stripped from previewName and endpointUrl before they are written to GITHUB_ENV/GITHUB_OUTPUT. ephemeral/shutdown/action.yml: same env-var move for the api key and action path, and newlines are stripped from pr_id and previewName before they are written out. Prior fix re-applied: action.yml and startup/action.yml no longer search ./../../_actions/* for the upstream name. GH_ACTION_ROOT is now built from $GITHUB_ACTION_PATH, relative to the workspace with a './' prefix (startup adds one '..'). The prior patch also contained a telemetry/OIDC record step, which is not a security fix, so I did not re-add it.

