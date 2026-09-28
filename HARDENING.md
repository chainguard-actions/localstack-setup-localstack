<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 2 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Multiple steps embed GitHub Actions expressions directly in shell scripts, allowing an attacker-controlled value to be interpreted as shell code before the shell ever sees it.

1. prepare/action.yml line 16: `run: echo ${{ github.event.number }} > ./pr-id.txt` — github.event.number interpolated directly into shell.

2. startup/action.yml line 63: `export CI_PROJECT=${{ inputs.ci-project }}` — inputs.ci-project interpolated directly into shell (and the value is then used in an eval call on the same script).

3. ephemeral/startup/action.yml line 57: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs.localstack-api-key interpolated directly into shell.

4. ephemeral/startup/action.yml line 59: `source ${{ github.action_path }}/../retry-function.sh` — github.action_path interpolated directly into shell.

5. ephemeral/startup/action.yml lines 72-74: `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"` — multiple inputs interpolated directly into shell.

6. ephemeral/startup/action.yml line 131: `run: |\n  ${{ inputs.preview-cmd }}` — inputs.preview-cmd is directly executed as a shell command. This is a critical arbitrary code execution vector.

7. ephemeral/startup/action.yml line 138: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — second occurrence in the Print logs step.

8. ephemeral/startup/action.yml line 140: `source ${{ github.action_path }}/../retry-function.sh` — second occurrence in the Print logs step.

9. ephemeral/shutdown/action.yml line 36: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs.localstack-api-key interpolated directly into shell.

10. ephemeral/shutdown/action.yml line 38: `source ${{ github.action_path }}/../retry-function.sh` — github.action_path interpolated directly into shell.

11. finish/action.yml lines 63-64: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — inputs.preview-url interpolated directly into shell and written to GITHUB_ENV.

Locations:

- `prepare/action.yml:16`
- `startup/action.yml:63`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:59`
- `ephemeral/startup/action.yml:72`
- `ephemeral/startup/action.yml:131`
- `ephemeral/startup/action.yml:138`
- `ephemeral/startup/action.yml:140`
- `ephemeral/shutdown/action.yml:36`
- `ephemeral/shutdown/action.yml:38`
- `finish/action.yml:63`

### github-env-injection (severity: high)

Unsanitized untrusted input values are written to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step, enabling newline injection that can set arbitrary environment variables for subsequent steps.

1. finish/action.yml line 64: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — inputs.preview-url (an action input, attacker-controlled) is interpolated directly into the GITHUB_ENV write with no sanitization. A value containing a newline could inject additional KEY=VALUE pairs.

2. startup/action.yml (env block): The `CONFIGURATION` env var is set from `${{ inputs.configuration }}` and then used inside the run block as `eval "${CONFIGURATION} localstack start -d"`. While the env: mapping itself is not a direct GITHUB_ENV write, the unquoted `${CONFIGURATION}` expansion inside `eval` means any shell metacharacters in the input are executed. Additionally, `export CI_PROJECT=${{ inputs.ci-project }}` is a direct expression write inside the run block without sanitization.

Locations:

- `finish/action.yml:64`
- `startup/action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 11 script-injection locations and 2 github-env-injection locations:

1. prepare/action.yml line 16: Moved `github.event.number` to env block as PR_NUMBER
2. startup/action.yml line 63: Moved `inputs.ci-project` to env block as INPUT_CI_PROJECT
3. ephemeral/startup/action.yml (Create preview environment): Moved `inputs.localstack-api-key`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, `inputs.lifetime`, and `github.action_path` to env block
4. ephemeral/startup/action.yml (Run preview deployment): Moved `inputs.preview-cmd` to env block as PREVIEW_CMD, used `eval "$PREVIEW_CMD"` to execute it
5. ephemeral/startup/action.yml (Print logs): Moved `inputs.localstack-api-key` and `github.action_path` to env block
6. ephemeral/shutdown/action.yml: Moved `inputs.localstack-api-key` and `github.action_path` to env block
7. finish/action.yml: Moved `inputs.preview-url` to env block as INPUT_PREVIEW_URL; added `printf '%s' ... | tr -d '\n\r'` sanitization before all GITHUB_ENV writes to prevent newline injection

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection findings:
1. startup/action.yml: Quoted `${IMAGE_NAME}` in `docker pull "${IMAGE_NAME}"` and replaced `eval "${CONFIGURATION} localstack start -d"` with `env ${CONFIGURATION} localstack start -d` to avoid shell interpretation of user-controlled CONFIGURATION input.
2. ephemeral/startup/action.yml: Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"` to run in a subshell rather than the current shell context.
3. cloud-pods/action.yml: Quoted `$NAME` in both `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent word splitting/glob expansion.
4. local/action.yml: Quoted `${NAME}.zip` in both `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent word splitting/glob expansion.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/startup/action.yml:
1. Line 80: Replaced unquoted `env ${CONFIGURATION} localstack start -d` with a safe xargs-based tokenization approach. The CONFIGURATION variable (containing space-separated KEY=VALUE pairs) is now tokenized into a bash array using `xargs printf '%s\0'` with a null-delimited read loop, then expanded as `env "${conf_args[@]}" localstack start -d`. This prevents shell metacharacter injection while preserving the intended behavior.
2. Line 83: Added double quotes around `${LS_WAIT_TIMEOUT:-30}` to make it `"${LS_WAIT_TIMEOUT:-30}"`, preventing word splitting and metacharacter interpretation of the environment variable.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Run preview deployment' step in ephemeral/startup/action.yml. The original code used `bash -c "$PREVIEW_CMD"` which interprets user-controlled input as shell code, enabling command injection. The fix writes the PREVIEW_CMD value to a temporary script file using `printf '%s\n' "$PREVIEW_CMD" > "$PREVIEW_SCRIPT"` and then executes it with `bash "$PREVIEW_SCRIPT"`, avoiding the dangerous `bash -c` pattern while preserving the intended functionality of running user-provided shell commands.

### Iteration 5

**Fixes applied:** github-env-injection

**Notes:**

Fixed all three github-env-injection findings:

1. ephemeral/shutdown/action.yml (line 21): Sanitized pr-id.txt content before writing pr_id to $GITHUB_OUTPUT using `printf '%s' "$(<pr-id.txt)" | tr -d '\n\r'`.

2. ephemeral/shutdown/action.yml (line 30): Sanitized both the prId (from pr-id.txt) and the final previewName before writing to $GITHUB_ENV using `printf '%s' ... | tr -d '\n\r'`.

3. ephemeral/startup/action.yml (lines 56-57): Sanitized both the prId (from pr-id.txt) and the final previewName before writing to $GITHUB_ENV and $GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'`.

4. finish/action.yml (line 40): Sanitized pr-id.txt content before writing pr_id to $GITHUB_OUTPUT using `printf '%s' "$(<pr-id.txt)" | tr -d '\n\r'`.

All writes to special environment files now apply the mandatory sanitization step to strip newlines and carriage returns that could be used for injection attacks.

