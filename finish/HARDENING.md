<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Load the Ephemeral Instance URL' step puts the expression ${{ inputs.preview-url }} directly inside its run: script. Offending lines: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. The value is pasted into the script before bash runs it, so shell metacharacters in it get executed. Fix: pass the input in through an env: variable and refer to it in double quotes.

Locations:

- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes values to $GITHUB_ENV without first running printf '%s' ... | tr -d '\n\r' on them. The values written are: the LS_PREVIEW_URL env var inherited from the calling workflow, inputs.preview-url, and the contents of ls-preview-url.txt, which comes from a downloaded artifact that can be produced by a PR workflow. A value containing a newline can set any extra environment variable, such as LD_PRELOAD or NODE_OPTIONS, for later steps. Lines: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` and `echo "LS_PREVIEW_URL=$(<ls-preview-url.txt)" >> $GITHUB_ENV`.

Locations:

- `action.yml:66`
- `action.yml:68`

### github-env-injection (severity: high)

The 'Load the PR ID' step writes the contents of pr-id.txt to $GITHUB_OUTPUT without stripping newlines: `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT`. The file comes from a downloaded artifact. The fallback downloads it from the latest run of a PR-related workflow, which a PR can control. A newline in the file would let it add extra step outputs. Fix: run printf '%s' ... | tr -d '\n\r' on the value first, and ideally also check that it is a number.

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
index a960837..3fccb5f 100644
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
@@ -57,26 +68,27 @@ runs:
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
+        _resolved="${LS_PREVIEW_URL:-$INPUT_PREVIEW_URL}"
+        if [[ -n "$_resolved" ]]; then
+          safe="$(printf '%s' "$_resolved" | tr -d '\n\r')"
+          echo "LS_PREVIEW_URL=$safe" >> "$GITHUB_ENV"
         elif [[ -e ls-preview-url.txt ]]; then
-          echo "LS_PREVIEW_URL=$(<ls-preview-url.txt)" >> $GITHUB_ENV
+          raw="$(<ls-preview-url.txt)"
+          safe="$(printf '%s' "$raw" | tr -d '\n\r')"
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

All changes are in finish/action.yml. I haven't run the action to check them.

1. 'Load the Ephemeral Instance URL' step: inputs.preview-url now comes in through env as INPUT_PREVIEW_URL instead of being pasted into the script. The step still picks the same value (LS_PREVIEW_URL first, then the input) and strips newlines with printf '%s' | tr -d '\n\r' before writing to $GITHUB_ENV.
2. Same step, ls-preview-url.txt fallback: the file's contents are also stripped of newlines before writing to $GITHUB_ENV.
3. 'Load the PR ID' step: the contents of pr-id.txt are stripped of newlines before writing to $GITHUB_OUTPUT. The file reads end in '|| true' because upstream read them inside echo's arguments, so a missing file never stopped the step. I kept that.

The previous harden also turned the comment body into one quoted line. I didn't repeat that: it's formatting only, not a security fix. The finding also suggested checking that the PR ID is a number. I didn't add that check, so a non-numeric value with its newlines removed still goes through to the comment step.

