<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): step 'Load the Ephemeral Instance URL' puts the expression ${{ inputs.preview-url }} straight into the run: script, on two lines: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. The value is pasted into the script before the shell runs, so a caller-supplied value containing $(...), backticks or quotes runs as commands. Fix: pass the input through an env: variable (e.g. INPUT_PREVIEW_URL) and use it double-quoted.

Locations:

- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

Step 'Load the Ephemeral Instance URL' writes workflow-controlled data to $GITHUB_ENV without first running printf '%s' ... | tr -d '\n\r'. Two writes are affected: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`, whose value comes from the inherited LS_PREVIEW_URL env var or inputs.preview-url, and `echo "LS_PREVIEW_URL=$(<ls-preview-url.txt)" >> $GITHUB_ENV`, whose value is the contents of a downloaded artifact. A value containing a newline can add arbitrary environment variables (e.g. LD_PRELOAD, NODE_OPTIONS) for later steps.

Locations:

- `action.yml:66`
- `action.yml:68`

### github-env-injection (severity: high)

Step 'Load the PR ID' runs `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT`. pr-id.txt comes from an artifact, which the previous step may download from another PR-related workflow run, so its contents are outside this action's control. The value is written to $GITHUB_OUTPUT without stripping newlines (printf '%s' ... | tr -d '\n\r'), so newlines in the file can add extra step outputs.

Locations:

- `action.yml:42`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-url }}" appears directly in run: block of step "Load the Ephemeral Instance URL"; move to env: map

Locations:

- `action.yml:65`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-url }}" appears directly in run: block of step "Load the Ephemeral Instance URL"; move to env: map

Locations:

- `action.yml:66`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/finish/action.yml b/finish/action.yml
index a960837..d6a9ba1 100644
--- a/finish/action.yml
+++ b/finish/action.yml
@@ -1,6 +1,5 @@
 name: Finish CI Build
 description: Finish CI Build
-
 inputs:
   github-token:
     description: 'Github token used to create PR comments'
@@ -14,18 +13,33 @@ inputs:
   preview-url:
     description: 'The Ephmeral Instance URL to include in the PR comment'
     required: false
-
 runs:
   using: composite
   steps:
     # Try to get pr artifact from current workflow
     - name: Download current PR artifact
       id: get-pr-artifact
       uses: actions/download-artifact@37930b1c2abaa49bbe596cd826c3c89aef350131 # v7
       continue-on-error: true
       with:
         name: pr-id
-
     # If the above fails, try to get the latest pr artifact from the PR related workflows
     - name: Download latest PR artifact
       uses: dawidd6/action-download-artifact@0bd50d53a6d7fb5cb921e607957e9cc12b4ce392 # v12
@@ -35,19 +49,16 @@ runs:
         pr: ${{ github.event.pull_request.number }}
         # Can be ID or workflow file name, if empty falls back to the latest successful run of the current workflow
         workflow: ${{ env.PR_ARTIFACT_WORKFLOW }}
-
     - name: Load the PR ID
       id: pr
       shell: bash
       run: echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT
-
     - name: Download preview instance URL
       id: get-preview-instance-url-artifact
       if: inputs.include-preview
       uses: actions/download-artifact@37930b1c2abaa49bbe596cd826c3c89aef350131 # v7
       with:
         name: preview-instance-url
-
     # If the above fails, try to get the latest pr artifact from the PR related workflows
     - name: Download latest PR artifact
       uses: dawidd6/action-download-artifact@0bd50d53a6d7fb5cb921e607957e9cc12b4ce392 # v12
@@ -57,26 +68,26 @@ runs:
         pr: ${{ github.event.pull_request.number }}
         # Can be ID or workflow file name, if empty falls back to the latest successful run of the current workflow
         workflow: ${{ env.PR_ARTIFACT_WORKFLOW }}
-
     - name: Load the Ephemeral Instance URL
       shell: bash
       if: inputs.include-preview
+      env:
+        INPUT_PREVIEW_URL: ${{ inputs.preview-url }}
       run: |
-        if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then
-          echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV
+        resolved="${LS_PREVIEW_URL:-$INPUT_PREVIEW_URL}"
+        if [[ -n "$resolved" ]]; then
+          safe="$(printf '%s' "$resolved" | tr -d '\n\r')"
+          echo "LS_PREVIEW_URL=$safe" >> "$GITHUB_ENV"
         elif [[ -e ls-preview-url.txt ]]; then
-          echo "LS_PREVIEW_URL=$(<ls-preview-url.txt)" >> $GITHUB_ENV
+          safe="$(printf '%s' "$(<ls-preview-url.txt)" | tr -d '\n\r')"
+          echo "LS_PREVIEW_URL=$safe" >> "$GITHUB_ENV"
         else
-          echo "LS_PREVIEW_URL=Unable to determine preview URL" >> $GITHUB_ENV
+          echo "LS_PREVIEW_URL=Unable to determine preview URL" >> "$GITHUB_ENV"
         fi
-
     - name: Update status comment
       uses: actions-cool/maintain-one-comment@4b2dbf086015f892dcb5e8c1106f5fccd6c1476b # v3
       with:
         token: ${{ inputs.github-token }}
-        body: |
-          ${{ inputs.ci-project && format('{0}{1}', '🚀 LocalStack Stack Insights and Cloud Pod state for this CI run: https://app.localstack.cloud/ci/', inputs.ci-project) }}
-          ${{ inputs.include-preview && format('{0}{1}', '🚀 Preview for this PR: ', env.LS_PREVIEW_URL) }}
-          <!-- Sticky Pull Request Comment -->
+        body: "${{ inputs.ci-project && format('{0}{1}', '\U0001F680 LocalStack Stack Insights and Cloud Pod state for this CI run: https://app.localstack.cloud/ci/', inputs.ci-project) }}\n${{ inputs.include-preview && format('{0}{1}', '\U0001F680 Preview for this PR: ', env.LS_PREVIEW_URL) }}\n<!-- Sticky Pull Request Comment -->\n"
         body-include: '<!-- Sticky Pull Request Comment -->'
         number: ${{ steps.pr.outputs.pr_id }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

I fixed both run: steps in action.yml; nothing was run or tested. 'Load the Ephemeral Instance URL' now gets inputs.preview-url through an env: entry, INPUT_PREVIEW_URL, and uses "${LS_PREVIEW_URL:-$INPUT_PREVIEW_URL}" in place of the pasted expression. Before anything is written to $GITHUB_ENV or $GITHUB_OUTPUT, newlines are stripped with printf '%s' | tr -d '\n\r'. That covers the preview URL (from the input, the inherited variable or ls-preview-url.txt) and the PR ID from pr-id.txt. Upstream read each file inside echo's arguments, so a missing file did not stop the step. To keep that, each file is read with raw="$(<file)" || true before it is cleaned. I re-applied the security part of the previous harden. I left out its rewrite of the maintain-one-comment body, because that only changed formatting: one line with escaped emoji instead of the multi-line block. It was not a security fix.

