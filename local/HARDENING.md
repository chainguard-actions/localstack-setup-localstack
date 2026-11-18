<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: the env var NAME comes from inputs.name, which the caller controls, but it is expanded without quotes in the run: block: `localstack state export ${NAME}.zip` (line 38) and `localstack state import ${NAME}.zip` (line 41). Because it is unquoted, the shell splits the value on whitespace and expands glob characters, so the input can change the arguments passed to localstack. Fix: quote it as "${NAME}.zip".

Locations:

- `action.yml:38`
- `action.yml:41`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/local/action.yml b/local/action.yml
index c0fb62c..a49484c 100644
--- a/local/action.yml
+++ b/local/action.yml
@@ -1,6 +1,5 @@
 name: 'Save/Load LocalStack state'
 description: 'Save or load LocalStack state'
-
 inputs:
   name:
     description: 'Name of the artifact'
@@ -10,19 +9,34 @@ inputs:
     description: 'Action to perform (save or load)'
     required: true
     default: 'save'
-
 runs:
   using: "composite"
   steps:
     # Try to get pr artifact from current workflow
     - name: Download current workflow's Local State artifact
       id: get-state-artifact
       if: ${{ inputs.action == 'load' }}
       uses: actions/download-artifact@37930b1c2abaa49bbe596cd826c3c89aef350131 #v7
       continue-on-error: true
       with:
         name: ${{ inputs.name }}
-
     # If the above fails, try to get the latest artifact from given workflow
     - name: Download latest Local State artifact
       uses: dawidd6/action-download-artifact@0bd50d53a6d7fb5cb921e607957e9cc12b4ce392 #v12
@@ -31,14 +45,13 @@ runs:
         name: ${{ inputs.name }}
         workflow: ${{ env.LS_STATE_ARTIFACT_WORKFLOW }} # Can be ID or workflow file name, if empty falls back to the latest successful run of the current workflow
         if_no_artifact_found: warn
-
     - run: |
         if [ "$ACTION" = "save" ]; then
           echo "Saving State $NAME"
-          localstack state export ${NAME}.zip
+          localstack state export "${NAME}.zip"
         elif [ "$ACTION" = "load" ]; then
           echo "Loading State $NAME"
-          localstack state import ${NAME}.zip
+          localstack state import "${NAME}.zip"
         else
           echo "Invalid action: $ACTION"
           exit 1
@@ -47,9 +60,8 @@ runs:
       env:
         NAME: "${{ inputs.name }}"
         ACTION: "${{ inputs.action }}"
-
     - name: Upload LocalStack State
       uses: actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6
       if: ${{ inputs.action == 'save' }}
       with:
         name: ${{ inputs.name }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, prior-fix

**Notes:**

In action.yml I put double quotes around ${NAME}.zip in both the `localstack state export` and `localstack state import` commands. The input can no longer be split on spaces or expanded as a wildcard pattern into extra arguments. This is the same security change the previous harden made, so the prior-fix finding is covered too. The previous patch also removed some blank lines; I left those in because they make no security difference. I didn't run anything to test the change.

