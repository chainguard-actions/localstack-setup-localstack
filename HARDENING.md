<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.2.5** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions inside shell command strings, violating sub-rule (a). The most critical instance is in ephemeral/startup/action.yml where `${{ inputs.preview-cmd }}` is used as the entire body of a run: step, allowing an attacker to execute arbitrary shell commands. Other violations include: `echo ${{ github.event.number }}` (prepare/action.yml), `export CI_PROJECT=${{ inputs.ci-project }}` and `eval "${CONFIGURATION} localstack start -d"` where CONFIGURATION holds `${{ inputs.configuration }}` (startup/action.yml — also sub-rule (b): unquoted expansion of untrusted env var passed to eval), `AUTH_HEADER="...${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}"`, `source ${{ github.action_path }}/../retry-function.sh`, `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"` (ephemeral/startup/action.yml), and `${LS_PREVIEW_URL:-${{ inputs.preview-url }}}` (finish/action.yml). All of these allow expression values to be interpreted by the shell before quoting.

Locations:

- `prepare/action.yml:16`
- `startup/action.yml:56`
- `startup/action.yml:57`
- `ephemeral/startup/action.yml:51`
- `ephemeral/startup/action.yml:54`
- `ephemeral/startup/action.yml:67`
- `ephemeral/startup/action.yml:68`
- `ephemeral/startup/action.yml:69`
- `ephemeral/startup/action.yml:131`
- `ephemeral/startup/action.yml:148`
- `ephemeral/startup/action.yml:151`
- `ephemeral/shutdown/action.yml:33`
- `ephemeral/shutdown/action.yml:36`
- `finish/action.yml:57`

### github-env-injection (severity: high)

Unsanitized values are written to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. In finish/action.yml, `${{ inputs.preview-url }}` is interpolated directly into the run: block and then written to GITHUB_ENV via `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — an attacker-controlled input value can inject newlines to set arbitrary environment variables. In ephemeral/startup/action.yml, `endpointUrl` (derived from an external API response whose content is influenced by attacker-controlled inputs like `previewName`) is written to GITHUB_ENV without sanitization via `echo "LS_PREVIEW_URL=$endpointUrl" >> $GITHUB_ENV` and `echo "AWS_ENDPOINT_URL=$endpointUrl" >> $GITHUB_ENV`.

Locations:

- `finish/action.yml:58`
- `ephemeral/startup/action.yml:121`
- `ephemeral/startup/action.yml:122`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env block as PR_NUMBER.

2. startup/action.yml: Added CI_PROJECT_INPUT to env block; replaced direct `${{ inputs.ci-project }}` interpolation; replaced `eval "${CONFIGURATION} localstack start -d"` with safe xargs-based tokenization using `env "${localstack_env[@]}" localstack start -d`.

3. ephemeral/startup/action.yml: Added env block with INPUT_LOCALSTACK_API_KEY, INPUT_AUTO_LOAD_POD, INPUT_EXTENSION_AUTO_INSTALL, INPUT_LIFETIME, ACTION_PATH; replaced all direct ${{ }} interpolations in run blocks; fixed `source ${{ github.action_path }}` to use `source "${ACTION_PATH}/../retry-function.sh"`; sanitized endpointUrl before writing to GITHUB_ENV with `printf '%s' "$endpointUrl" | tr -d '\n\r'`; moved preview-cmd to env block and used `eval "$PREVIEW_CMD"`.

4. ephemeral/shutdown/action.yml: Added env block with INPUT_LOCALSTACK_API_KEY and ACTION_PATH; replaced direct ${{ }} interpolations in AUTH_HEADER and source command.

5. finish/action.yml: Added INPUT_PREVIEW_URL to env block; replaced `${LS_PREVIEW_URL:-${{ inputs.preview-url }}}` with env var reference; added `printf '%s' ... | tr -d '\n\r'` sanitization before all writes to GITHUB_ENV.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities: (1) In startup/action.yml line 68, added double quotes around ${IMAGE_NAME} in the `docker pull` command to prevent shell metacharacter injection from the user-controlled `inputs.image-tag` value. (2) In ephemeral/startup/action.yml line 163, replaced `eval "$PREVIEW_CMD"` with `sh -c "$PREVIEW_CMD"` to execute the user-provided preview command in an isolated subshell rather than the current shell context, preventing it from modifying the current shell's environment and state. The PREVIEW_CMD variable was already correctly placed in the env block.

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 7 security findings across 5 files:

1. ephemeral/startup/action.yml - 'Run preview deployment': Replaced `sh -c "$PREVIEW_CMD"` with writing PREVIEW_CMD to a temp file and executing with `bash "$_preview_script"` to prevent shell metacharacter injection.

2. ephemeral/startup/action.yml - 'Setup preview name': Added sanitization of previewName with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

3. startup/action.yml - 'Start LocalStack': Restructured CONFIGURATION handling to build localstack_env array directly with IMAGE_NAME and DNS_ADDRESS as separate elements, then tokenize user-provided CONFIGURATION separately using xargs with a proper empty-value guard.

4. cloud-pods/action.yml: Quoted `$NAME` → `"$NAME"` in `localstack pod save` and `localstack pod load --yes` commands.

5. local/action.yml: Quoted `${NAME}.zip` → `"${NAME}.zip"` in `localstack state export` and `localstack state import` commands.

6. ephemeral/shutdown/action.yml - 'Load the PR ID': Sanitized pr-id.txt content with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

7. ephemeral/shutdown/action.yml - 'Setup preview name': Added sanitization of previewName before writing to $GITHUB_ENV.

8. finish/action.yml - 'Load the PR ID': Sanitized pr-id.txt content with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed both _actions directory glob searches by replacing them with $GITHUB_ACTION_PATH-based paths:
1. action.yml:90 - Replaced the multi-line ls/_actions glob with a simple `echo "GH_ACTION_ROOT=$GITHUB_ACTION_PATH" >> $GITHUB_ENV` since action.yml is at the action root.
2. startup/action.yml:44 - Replaced the multi-line ls/_actions glob with `GH_ACTION_ROOT="$(cd "$GITHUB_ACTION_PATH/.." && pwd)"` since startup/action.yml is one directory level deep, so the action root is one level up from $GITHUB_ACTION_PATH.

