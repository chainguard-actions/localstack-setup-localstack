<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Save PR number' step puts a GitHub expression straight into the run: script: `run: echo ${{ github.event.number }} > ./pr-id.txt`. Fix: pass it in through an env: variable (e.g. PR_NUMBER) and use it double-quoted as "$PR_NUMBER".

Locations:

- `action.yml:17`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/prepare/action.yml b/prepare/action.yml
index 48ef59e..fb7f4de 100644
--- a/prepare/action.yml
+++ b/prepare/action.yml
@@ -1,6 +1,5 @@
 name: Start CI Build
 description: Start CI Build
-
 inputs:
   github-token:
     description: 'Github token used to create PR comments'
@@ -8,20 +7,36 @@ inputs:
   ci-project:
     description: 'Name of the CI project to track in LocalStack Cloud'
     required: false
-
 runs:
   using: composite
   steps:
     - name: Save PR number
       shell: bash
-      run: echo ${{ github.event.number }} > ./pr-id.txt
-
+      env:
+        PR_NUMBER: ${{ github.event.number }}
+      run: echo "$PR_NUMBER" > ./pr-id.txt
     - name: Upload PR number
       uses: actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6
       with:
         name: pr-id
         path: ./pr-id.txt
-
     - name: Create initial PR comment
       uses: actions-cool/maintain-one-comment@4b2dbf086015f892dcb5e8c1106f5fccd6c1476b # v3
       with:

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, prior-fix

**Notes:**

Moved github.event.number into env var PR_NUMBER in the 'Save PR number' step and referenced it as "$PR_NUMBER". Uses were already pinned.

