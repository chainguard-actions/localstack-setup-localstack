<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: the env var NAME holds the workflow-controlled value of inputs.name, and the run: block expands it without quotes in `localstack pod save $NAME` (line 21) and `localstack pod load --yes $NAME` (line 24). The shell splits the unquoted value on whitespace and expands glob characters, so the input can add extra arguments or options to the localstack command. Fix: quote it as "$NAME".

Locations:

- `action.yml:21`
- `action.yml:24`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/cloud-pods/action.yml b/cloud-pods/action.yml
index da73596..ee0d8d0 100644
--- a/cloud-pods/action.yml
+++ b/cloud-pods/action.yml
@@ -1,6 +1,5 @@
 name: 'Save/Load a LocalStack Cloud Pod'
 description: 'Save or load a LocalStack Cloud Pod'
-
 inputs:
   name:
     description: 'Name of the Cloud Pod'
@@ -10,18 +9,33 @@ inputs:
     description: 'Action to perform (save or load)'
     required: true
     default: 'save'
-
 runs:
   using: "composite"
   steps:
-    - 
       run: |
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
+    - run: |
         if [ "$ACTION" = "save" ]; then
           echo "Saving Cloud Pod $NAME"
-          localstack pod save $NAME
+          localstack pod save "$NAME"
         elif [ "$ACTION" = "load" ]; then
           echo "Loading Cloud Pod $NAME"
-          localstack pod load --yes $NAME
+          localstack pod load --yes "$NAME"
         else
           echo "Invalid action: $ACTION"
           exit 1

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, prior-fix

**Notes:**

I quoted $NAME in `localstack pod save` and `localstack pod load --yes` in action.yml. The shell no longer splits or glob-expands the cloud pod name, so the input can't add extra arguments to the localstack command. That quoting was the only security change in the previous harden, and it is now back in place. I did not copy that patch's best-effort OIDC/record step, the step that posts to actions.enforce.dev. It doesn't harden anything, and I'm assuming the publishing pipeline adds it separately. Nothing has been run or tested.

