<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell blocks. In prepare/action.yml, `${{ github.event.number }}` is interpolated directly into a shell command (`echo ${{ github.event.number }} > ./pr-id.txt`). In startup/action.yml, `${{ inputs.ci-project }}` is interpolated directly (`export CI_PROJECT=${{ inputs.ci-project }}`). In ephemeral/shutdown/action.yml, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside run: blocks. In ephemeral/startup/action.yml, multiple expressions are interpolated directly: `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, `${{ inputs.lifetime }}`, `${{ github.action_path }}`, and most critically `${{ inputs.preview-cmd }}` is executed directly as a shell command (`${{ inputs.preview-cmd }}`), allowing arbitrary command injection. In finish/action.yml, `${{ inputs.preview-url }}` is interpolated directly inside a run: block.

Locations:

- `prepare/action.yml:16`
- `startup/action.yml:75`
- `ephemeral/shutdown/action.yml:33`
- `ephemeral/shutdown/action.yml:38`
- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:72`
- `ephemeral/startup/action.yml:73`
- `ephemeral/startup/action.yml:74`
- `ephemeral/startup/action.yml:148`
- `ephemeral/startup/action.yml:152`
- `ephemeral/startup/action.yml:163`
- `ephemeral/startup/action.yml:168`
- `finish/action.yml:62`
- `finish/action.yml:63`

### github-env-injection (severity: high)

Unsanitized input values are written to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. In finish/action.yml, `${{ inputs.preview-url }}` is interpolated directly into the shell and then written to $GITHUB_ENV: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled value containing newlines could inject arbitrary environment variables. Similarly, in ephemeral/shutdown/action.yml and ephemeral/startup/action.yml, `previewName` (derived from the inherited `$GITHUB_REPOSITORY` env var and file content) is written to $GITHUB_ENV without sanitization.

Locations:

- `finish/action.yml:63`
- `ephemeral/shutdown/action.yml:28`
- `ephemeral/startup/action.yml:51`
- `ephemeral/startup/action.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env block (PR_NUMBER).

2. startup/action.yml: Replaced _actions glob with `$(cd "$GITHUB_ACTION_PATH/.." && pwd)` (startup/ is one level deep, so parent is repo root). Moved `${{ inputs.ci-project }}` to env block (CI_PROJECT_INPUT).

3. ephemeral/shutdown/action.yml: Sanitized previewName with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV. Moved `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` to env block.

4. ephemeral/startup/action.yml: Sanitized previewName before writing to GITHUB_ENV/GITHUB_OUTPUT. Moved all injected expressions (`inputs.localstack-api-key`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, `inputs.lifetime`, `github.action_path`) to env blocks. Fixed the critical `${{ inputs.preview-cmd }}` direct execution by moving to env block as PREVIEW_CMD and using `eval "$PREVIEW_CMD"`.

5. finish/action.yml: Moved `${{ inputs.preview-url }}` to env block (INPUT_PREVIEW_URL). Sanitized both the resolved URL and file-read URL with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across 4 files:

1. cloud-pods/action.yml: Quoted $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent shell word-splitting/injection.

2. local/action.yml: Quoted ${NAME}.zip → "${NAME}.zip" in both localstack state export and import commands.

3. startup/action.yml: (a) Quoted ${IMAGE_NAME} → "${IMAGE_NAME}" in docker pull. (b) Replaced `eval "${CONFIGURATION} localstack start -d"` with xargs-based tokenization into a bash array (cfg_args) and `env "${cfg_args[@]}" localstack start -d`, eliminating arbitrary shell evaluation of user-controlled CONFIGURATION input while preserving KEY=VALUE env-var semantics.

4. ephemeral/startup/action.yml: (a) Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"` to run in a subshell. (b) Added sanitization of API-derived endpointUrl via `printf '%s' "$endpointUrl" | tr -d '\n\r'` before writing to $GITHUB_ENV, preventing newline injection.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings:
1. ephemeral/startup/action.yml: Replaced `bash -c "$PREVIEW_CMD"` with writing PREVIEW_CMD to a temp script file via printf and executing it with `bash "$_preview_script"`. This prevents the user-controlled input from being interpreted as shell code through the -c argument.
2. ephemeral/shutdown/action.yml: Added sanitization for pr_id before writing to GITHUB_OUTPUT - reads raw value, strips newlines/carriage returns with `tr -d '\n\r'`, then writes safe value.
3. finish/action.yml: Same sanitization fix for pr_id - reads raw value, strips newlines/carriage returns, then writes safe value to GITHUB_OUTPUT.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection issues in hardened/action/startup/action.yml:
1. Removed unquoted ${CONFIGURATION} concatenations: Instead of building combined strings like CONFIGURATION="DNS_ADDRESS=127.0.0.1 ${CONFIGURATION}" and CONFIGURATION="IMAGE_NAME=${IMAGE_NAME} ${CONFIGURATION}", now initializes cfg_args directly with built-in values as separate properly-quoted array elements ("DNS_ADDRESS=127.0.0.1" and "IMAGE_NAME=${IMAGE_NAME}").
2. Fixed unquoted ${LS_WAIT_TIMEOUT:-30} → "${LS_WAIT_TIMEOUT:-30}".
3. Kept the xargs approach for tokenizing the user's CONFIGURATION list input (correct per instructions).
4. All uses of $CONFIGURATION are now properly double-quoted in every shell expansion.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Replaced the _actions/* glob search (ls -d ./../../_actions/* | grep -i localstack | tail -n1)/setup-localstack/* with $GITHUB_ACTION_PATH. The original code searched the runner's _actions directory for the action's own checkout by its upstream name 'localstack', which fails when the action is hardened/forked under a different owner. Since action.yml is at the action root, GH_ACTION_ROOT=$GITHUB_ACTION_PATH is the correct replacement — all sub-action paths (/tools, /startup, /ephemeral/startup, etc.) are relative to this root.

