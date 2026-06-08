# Hardening Report: LocalStack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct interpolation of `${{ github.event.number }}` inside a `run:` shell block. An attacker controlling the PR number (or a malicious event payload) could inject arbitrary shell commands. The value should be assigned to an environment variable and referenced as `$VAR` instead.

Locations:

- `prepare/action.yml:15`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.ci-project }}` inside a `run:` shell block (`export CI_PROJECT=${{ inputs.ci-project }}`). An attacker supplying a crafted `ci-project` input value could inject arbitrary shell commands. The value should be passed via an `env:` variable.

Locations:

- `startup/action.yml:62`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` inside `run:` shell blocks. Attacker-controlled input values are embedded directly into shell command strings without going through environment variables, enabling shell injection.

Locations:

- `ephemeral/shutdown/action.yml:34`
- `ephemeral/shutdown/action.yml:38`

### script-injection (severity: high)

Multiple direct interpolations of attacker-controlled expressions inside `run:` shell blocks: `${{ inputs.localstack-api-key }}`, `${{ github.action_path }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, `${{ inputs.lifetime }}`, and critically `${{ inputs.preview-cmd }}` which is executed directly as a shell command (`run: |\n  ${{ inputs.preview-cmd }}`). This last case allows an attacker to supply arbitrary shell commands via the `preview-cmd` input.

Locations:

- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:59`
- `ephemeral/startup/action.yml:76`
- `ephemeral/startup/action.yml:77`
- `ephemeral/startup/action.yml:78`
- `ephemeral/startup/action.yml:131`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.preview-url }}` inside a `run:` shell block in `finish/action.yml`. The value is embedded directly into a shell string and also written to `$GITHUB_ENV` without sanitization.

Locations:

- `finish/action.yml:57`

### github-env-injection (severity: high)

The `finish/action.yml` 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` (as `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`) without applying the required `printf '%s' ... | tr -d '\n\r'` sanitization. A newline in the input value could inject arbitrary environment variables into subsequent steps.

Locations:

- `finish/action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 6 script-injection and 1 github-env-injection findings across 5 files:

1. **prepare/action.yml** (line 15): Moved `${{ github.event.number }}` to `env: PR_NUMBER` and referenced as `"$PR_NUMBER"` in the run block.

2. **startup/action.yml** (line 62): Moved `${{ inputs.ci-project }}` to `env: CI_PROJECT_INPUT` and referenced as `"${CI_PROJECT_INPUT}"` in the run block.

3. **ephemeral/shutdown/action.yml** (lines 34, 38): Moved `${{ inputs.localstack-api-key }}` to `env: LOCALSTACK_API_KEY_INPUT` and `${{ github.action_path }}` to `env: ACTION_PATH`; updated shell references accordingly.

4. **ephemeral/startup/action.yml** (lines 55, 59, 76, 77, 78, 131): 
   - In 'Create preview environment' step: moved `inputs.localstack-api-key`, `github.action_path`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, and `inputs.lifetime` to env vars.
   - In 'Run preview deployment' step: moved `inputs.preview-cmd` to `env: PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` instead of directly interpolating the command.
   - In 'Print logs of ephemeral instance' step: moved `inputs.localstack-api-key` and `github.action_path` to env vars.

5. **finish/action.yml** (lines 57, 58): Moved `${{ inputs.preview-url }}` to `env: PREVIEW_URL_INPUT`; rewrote the run block to resolve the URL into a local variable and sanitize it with `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_ENV`, preventing newline injection.

