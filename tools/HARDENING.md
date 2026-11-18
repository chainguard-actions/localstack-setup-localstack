<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--tools/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--tools/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/tools/action.yml b/tools/action.yml
index b4b1c33..64866e4 100644
--- a/tools/action.yml
+++ b/tools/action.yml
@@ -1,15 +1,30 @@
 name: 'Install Localstack tools'
 description: 'Installs `localstack` and optionally `awslocal` CLI'
-
 inputs:
   install-awslocal:
     description: 'Whether to install the `awslocal` CLI into the environment'
     required: true
     default: 'true'
-
 runs:
   using: "composite"
   steps:
     - name: Start LocalStack
       run: |
         pip install --upgrade pip
@@ -20,4 +35,3 @@ runs:
       shell: bash
       env:
         INSTALL_AWSLOCAL: "${{ inputs.install-awslocal }}"
-

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Notes:**

I made no changes; tools/action.yml already has no security issue. The only changes the previous patch shows are deleted blank lines, which have no security effect. Its hunk header (+1,30) suggests it added lines that the diff does not show, so I could not tell what they were, and I did not invent any. The current file already passes inputs.install-awslocal through env: as $INSTALL_AWSLOCAL and quotes it in the shell. It has no uses: references, no piped downloads and no writes to GITHUB_ENV or GITHUB_OUTPUT. A composite action takes no permissions: block.

