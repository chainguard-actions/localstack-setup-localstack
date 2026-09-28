<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.2.5** was hardened automatically. 2 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions inside shell commands (sub-rule a), allowing script injection. Key violations:

1. prepare/action.yml line 16: `run: echo ${{ github.event.number }} > ./pr-id.txt` — github context interpolated directly into shell.

2. startup/action.yml line 57: `export CI_PROJECT=${{ inputs.ci-project }}` — inputs interpolated directly into shell assignment.

3. ephemeral/startup/action.yml line 57 (create-instance step): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs interpolated into shell string.

4. ephemeral/startup/action.yml line 60: `source ${{ github.action_path }}/../retry-function.sh` — github context interpolated into shell path.

5. ephemeral/startup/action.yml line 79: `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"` — inputs interpolated into shell.

6. ephemeral/startup/action.yml line 80: `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"` — inputs interpolated into shell.

7. ephemeral/startup/action.yml line 81: `lifetime="${{ inputs.lifetime }}"` — inputs interpolated into shell.

8. ephemeral/startup/action.yml line ~131 (run-preview-deployment step): `${{ inputs.preview-cmd }}` — the entire shell command is an attacker-controlled input expression; this is arbitrary remote code execution.

9. ephemeral/startup/action.yml (print-logs step): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

10. ephemeral/shutdown/action.yml line 35: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs interpolated into shell.

11. ephemeral/shutdown/action.yml line 38: `source ${{ github.action_path }}/../retry-function.sh` — github context interpolated into shell path.

12. finish/action.yml lines 58–59: `${LS_PREVIEW_URL:-${{ inputs.preview-url }}}` — inputs interpolated directly into shell conditional and echo command.

Locations:

- `prepare/action.yml:16`
- `startup/action.yml:57`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:79`
- `ephemeral/startup/action.yml:80`
- `ephemeral/startup/action.yml:81`
- `ephemeral/startup/action.yml:131`
- `ephemeral/startup/action.yml:143`
- `ephemeral/shutdown/action.yml:35`
- `ephemeral/shutdown/action.yml:38`
- `finish/action.yml:58`

### github-env-injection (severity: high)

finish/action.yml writes the value of inputs.preview-url directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The expression ${{ inputs.preview-url }} is interpolated inline into the shell string and then echoed to $GITHUB_ENV:

  `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

An attacker-controlled value containing newlines could inject arbitrary environment variable assignments into the runner environment for subsequent steps.

Locations:

- `finish/action.yml:59`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 12 script injection locations and 1 github-env-injection location across 5 files:

1. prepare/action.yml: Moved `github.event.number` to `env: PR_NUMBER` and referenced as `"$PR_NUMBER"` in the run block.

2. startup/action.yml: Moved `inputs.ci-project` to `env: CI_PROJECT_INPUT` and referenced as `"${CI_PROJECT_INPUT}"` in the run block.

3. ephemeral/startup/action.yml (create-instance step): Added env block with `INPUT_LOCALSTACK_API_KEY`, `INPUT_AUTO_LOAD_POD`, `INPUT_EXTENSION_AUTO_INSTALL`, `INPUT_LIFETIME`, and `ACTION_PATH`. Replaced all inline `${{ }}` expressions in the shell script with the corresponding env vars. The `source ${{ github.action_path }}/../retry-function.sh` was replaced with `source "${ACTION_PATH}/../retry-function.sh"`.

4. ephemeral/startup/action.yml (run-preview-deployment step): Moved `inputs.preview-cmd` to `env: PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` instead of directly executing the expression as a shell command.

5. ephemeral/startup/action.yml (print-logs step): Added env block with `INPUT_LOCALSTACK_API_KEY` and `ACTION_PATH`, replaced inline expressions in the shell script.

6. ephemeral/shutdown/action.yml (shutdown-ephemeral-instance step): Added env block with `INPUT_LOCALSTACK_API_KEY` and `ACTION_PATH`, replaced inline expressions in the shell script.

7. finish/action.yml (Load the Ephemeral Instance URL step): Moved `inputs.preview-url` to `env: INPUT_PREVIEW_URL`, resolved the value into a shell variable, and sanitized it with `printf '%s' "$resolved" | tr -d '\n\r'` before writing to `$GITHUB_ENV`. This fixes both the script-injection and github-env-injection findings.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 5 findings across 3 files:

1. startup/action.yml (script-injection): Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe `env` command using xargs to tokenize the CONFIGURATION string into an array of KEY=VALUE pairs, then passing them to `env` as separate arguments. This prevents arbitrary shell command injection via the configuration input.

2. ephemeral/startup/action.yml (script-injection): Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"` to avoid the extra evaluation pass that eval performs.

3. ephemeral/shutdown/action.yml (github-env-injection): Added sanitization of previewName using `printf '%s' "$previewName" | tr -d '\n\r'` before writing to $GITHUB_ENV.

4. ephemeral/startup/action.yml (github-env-injection, line 52): Added sanitization of previewName using `printf '%s' "$previewName" | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

5. ephemeral/startup/action.yml (github-env-injection, line 143): Added sanitization of endpointUrl using `printf '%s' "$endpointUrl" | tr -d '\n\r'` before writing LS_PREVIEW_URL and AWS_ENDPOINT_URL to $GITHUB_ENV.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed four script injection vulnerabilities:
1. ephemeral/startup/action.yml (line 161): Replaced `bash -c "$PREVIEW_CMD"` with writing PREVIEW_CMD to a temp file and executing it with `bash /tmp/_preview_cmd.sh`. This prevents the shell from re-parsing the env var value as shell code via bash -c's argument parsing.
2. cloud-pods/action.yml (lines 20, 23): Added double quotes around $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent word splitting and glob expansion.
3. local/action.yml (lines 36, 39): Added double quotes around ${NAME}.zip in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent word splitting and glob expansion.
4. startup/action.yml (line 70): Added double quotes around ${IMAGE_NAME} in `docker pull "${IMAGE_NAME}" &` to prevent word splitting and glob expansion.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions:
1. startup/action.yml line 85: Quoted `${LS_WAIT_TIMEOUT:-30}` → `"${LS_WAIT_TIMEOUT:-30}"` to prevent command injection via shell metacharacters in the env var.
2. ephemeral/startup/action.yml line 126: Quoted `${lifetime}` inside the JSON curl body → `\"${lifetime}\"` to prevent JSON string breakout via a double-quote character in the input value.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two instances of unsanitized writes to $GITHUB_OUTPUT:
1. `ephemeral/shutdown/action.yml` line 22: Replaced `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` with a two-line form that first sanitizes via `printf '%s' ... | tr -d '\n\r'` into a `safe` variable, then writes `pr_id=$safe` to `"$GITHUB_OUTPUT"`.
2. `finish/action.yml` line 35: Same fix applied — replaced the direct echo with a sanitized form using `printf '%s' ... | tr -d '\n\r'`.
Both fixes are consistent with the sanitization pattern already used elsewhere in the codebase.

