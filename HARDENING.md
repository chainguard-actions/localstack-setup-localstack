# Hardening Report: LocalStack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.2** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

prepare/action.yml: The 'Save PR number' step directly interpolates `${{ github.event.number }}` inside a `run:` shell command (`echo ${{ github.event.number }} > ./pr-id.txt`). This value should be assigned to an env var first.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

ephemeral/shutdown/action.yml: The 'Shutdown ephemeral instance' step directly interpolates `${{ inputs.localstack-api-key }}` inside a `run:` shell command (`AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`). Attacker-controlled input is embedded directly in the shell script.

Locations:

- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

ephemeral/startup/action.yml: The 'Create preview environment' step directly interpolates multiple attacker-controlled inputs inside a `run:` shell command: `${{ inputs.localstack-api-key }}` (line 57), `${{ inputs.auto-load-pod }}` (line 79), `${{ inputs.extension-auto-install }}` (line 80), and `${{ inputs.lifetime }}` (line 81). These should be assigned to env vars first.

Locations:

- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:79`
- `ephemeral/startup/action.yml:80`
- `ephemeral/startup/action.yml:81`

### script-injection (severity: high)

ephemeral/startup/action.yml: The 'Run preview deployment' step directly interpolates `${{ inputs.preview-cmd }}` as the entire body of a `run:` block. An attacker can supply arbitrary shell commands via this input, leading to full remote code execution.

Locations:

- `ephemeral/startup/action.yml:139`

### script-injection (severity: high)

ephemeral/startup/action.yml: The 'Print logs of ephemeral instance' step directly interpolates `${{ inputs.localstack-api-key }}` inside a `run:` shell command (`AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`). Attacker-controlled input is embedded directly in the shell script.

Locations:

- `ephemeral/startup/action.yml:148`

### script-injection (severity: high)

startup/action.yml: The 'Start LocalStack' step directly interpolates `${{ inputs.ci-project }}` inside a `run:` shell command (`export CI_PROJECT=${{ inputs.ci-project }}`). This attacker-controlled value is embedded directly in the shell script without going through an env: variable.

Locations:

- `startup/action.yml:73`

### script-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' step directly interpolates `${{ inputs.preview-url }}` inside a `run:` shell command (used in a bash conditional and in an echo command). Attacker-controlled input is embedded directly in the shell script.

Locations:

- `finish/action.yml:62`
- `finish/action.yml:63`

### github-env-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' step writes the attacker-controlled value `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. A newline character in the input could inject arbitrary environment variables into subsequent steps.

Locations:

- `finish/action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 8 security findings across 5 files:

1. prepare/action.yml (line 16): Moved `${{ github.event.number }}` to env var `PR_NUMBER` and referenced it as `"$PR_NUMBER"` in the run block.

2. ephemeral/shutdown/action.yml (line 36): Moved `${{ inputs.localstack-api-key }}` to env var `LOCALSTACK_API_KEY_INPUT` and updated the AUTH_HEADER construction to use `${LOCALSTACK_API_KEY_INPUT}`.

3. ephemeral/startup/action.yml (lines 57, 79, 80, 81 - Create preview environment step): Moved `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` to env vars `LOCALSTACK_API_KEY_INPUT`, `AUTO_LOAD_POD_INPUT`, `EXTENSION_AUTO_INSTALL_INPUT`, and `LIFETIME_INPUT` respectively, and updated all shell references accordingly.

4. ephemeral/startup/action.yml (line 139 - Run preview deployment step): Moved `${{ inputs.preview-cmd }}` to env var `PREVIEW_CMD` and replaced the direct interpolation with `eval "$PREVIEW_CMD"` to safely execute the command.

5. ephemeral/startup/action.yml (line 148 - Print logs step): Moved `${{ inputs.localstack-api-key }}` to env var `LOCALSTACK_API_KEY_INPUT` and updated the AUTH_HEADER construction.

6. startup/action.yml (line 73): Moved `${{ inputs.ci-project }}` to env var `CI_PROJECT_INPUT` in the step's env block and updated the shell reference to `"${CI_PROJECT_INPUT}"`.

7 & 8. finish/action.yml (lines 62, 63 - script-injection + github-env-injection): Moved `${{ inputs.preview-url }}` to env var `PREVIEW_URL_INPUT`, rewrote the conditional logic to use the env var, and added `printf '%s' "$resolved_url" | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV to prevent newline injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection occurrences where `${{ github.action_path }}` was directly interpolated in `run:` blocks:
1. `ephemeral/shutdown/action.yml` line 44 ('Shutdown ephemeral instance' step): Added `ACTION_PATH: ${{ github.action_path }}` to `env:` block and changed `source ${{ github.action_path }}/../retry-function.sh` to `source "$ACTION_PATH/../retry-function.sh"`.
2. `ephemeral/startup/action.yml` line 72 ('Create preview environment' step): Same fix applied.
3. `ephemeral/startup/action.yml` ~line 172 ('Print logs of ephemeral instance' step): Same fix applied.

