<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.0** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

prepare/action.yml: The 'Save PR number' run: block directly interpolates `${{ github.event.number }}` into the shell command string (sub-rule a). An attacker-controlled PR number could inject shell metacharacters. Offending line: `run: echo ${{ github.event.number }} > ./pr-id.txt`

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

startup/action.yml: The 'Start LocalStack' run: block directly interpolates `${{ inputs.ci-project }}` into the shell command string (sub-rule a), and then passes the `$CONFIGURATION` env var (sourced from `inputs.configuration`) directly to `eval` without quoting (sub-rule b). Both allow arbitrary shell command injection. Offending lines: `export CI_PROJECT=${{ inputs.ci-project }}` and `eval "${CONFIGURATION} localstack start -d"`

Locations:

- `startup/action.yml:68`
- `startup/action.yml:69`

### script-injection (severity: high)

ephemeral/startup/action.yml: Multiple run: blocks directly interpolate ${{ ... }} expressions into shell command strings (sub-rule a): (1) 'Create preview environment' step interpolates `${{ inputs.localstack-api-key }}`, `${{ github.action_path }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` directly in the shell script. (2) 'Run preview deployment' step executes `${{ inputs.preview-cmd }}` directly as a shell command — a critical arbitrary command injection. (3) 'Print logs' step interpolates `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` directly in the shell script.

Locations:

- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:76`
- `ephemeral/startup/action.yml:77`
- `ephemeral/startup/action.yml:78`
- `ephemeral/startup/action.yml:121`
- `ephemeral/startup/action.yml:128`
- `ephemeral/startup/action.yml:131`

### script-injection (severity: high)

ephemeral/shutdown/action.yml: The 'Shutdown ephemeral instance' run: block directly interpolates `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` into the shell command string (sub-rule a). Offending lines: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`

Locations:

- `ephemeral/shutdown/action.yml:34`
- `ephemeral/shutdown/action.yml:37`

### script-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' run: block directly interpolates `${{ inputs.preview-url }}` into the shell command string (sub-rule a). Offending lines: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' step writes `inputs.preview-url` directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The value `${{ inputs.preview-url }}` is interpolated directly into the shell string and then echoed to $GITHUB_ENV, allowing newline injection to set arbitrary environment variables. Offending line: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `finish/action.yml:58`

### github-env-injection (severity: high)

startup/action.yml: The 'Start LocalStack' step writes the `CONFIGURATION` env var (sourced from `inputs.configuration`) to the shell environment via `eval "${CONFIGURATION} localstack start -d"`, and `inputs.ci-project` is directly interpolated as `export CI_PROJECT=${{ inputs.ci-project }}`. The `CONFIGURATION` env var is set from `inputs.configuration` and passed unsanitized to `eval`, which can inject arbitrary environment variables and commands. Additionally, `inputs.configuration` is written to the process environment without newline sanitization.

Locations:

- `startup/action.yml:68`
- `startup/action.yml:69`

### github-env-injection (severity: high)

ephemeral/startup/action.yml: The 'Create preview environment' step writes `endpointUrl` (derived from an API response) to $GITHUB_ENV without sanitization: `echo "LS_PREVIEW_URL=$endpointUrl" >> $GITHUB_ENV` and `echo "AWS_ENDPOINT_URL=$endpointUrl" >> $GITHUB_ENV`. Additionally, the 'Setup preview name' step writes `previewName` (derived from `$GITHUB_REPOSITORY` and file content) to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization. These are inherited process env vars treated as untrusted.

Locations:

- `ephemeral/startup/action.yml:50`
- `ephemeral/startup/action.yml:51`
- `ephemeral/startup/action.yml:113`
- `ephemeral/startup/action.yml:114`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. **prepare/action.yml**: Moved `${{ github.event.number }}` to `PR_NUMBER` env var; shell now uses `"$PR_NUMBER"`.

2. **startup/action.yml**: Moved `${{ inputs.ci-project }}` to `CI_PROJECT_INPUT` env var; replaced `eval "${CONFIGURATION} localstack start -d"` with `env $(echo "$CONFIGURATION" | xargs) localstack start -d` to avoid eval-based command injection (CONFIGURATION was already in env block).

3. **ephemeral/startup/action.yml**:
   - 'Create preview environment' step: Moved `inputs.localstack-api-key`, `github.action_path`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, `inputs.lifetime` to env vars; sanitized `endpointUrl` with `tr -d '\n\r'` before writing to GITHUB_ENV.
   - 'Setup preview name' step: Sanitized `previewName` with `tr -d '\n\r'` before writing to GITHUB_ENV and GITHUB_OUTPUT.
   - 'Run preview deployment' step: Moved `inputs.preview-cmd` to `PREVIEW_CMD` env var; runs via `bash -c "$PREVIEW_CMD"`.
   - 'Print logs' step: Moved `inputs.localstack-api-key` and `github.action_path` to env vars.

4. **ephemeral/shutdown/action.yml**: Moved `inputs.localstack-api-key` and `github.action_path` to env vars in the 'Shutdown ephemeral instance' step.

5. **finish/action.yml**: Moved `inputs.preview-url` to `PREVIEW_URL_INPUT` env var; sanitized all values written to GITHUB_ENV with `printf '%s' ... | tr -d '\n\r'`.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

1. ephemeral/shutdown/action.yml: Added newline sanitization before writing previewName to GITHUB_ENV using `safe_preview_name=$(printf '%s' "$previewName" | tr -d '\n\r')`, matching the pattern already used in ephemeral/startup/action.yml.
2. startup/action.yml: (a) Quoted `${IMAGE_NAME}` in `docker pull` to prevent word-splitting/glob expansion. (b) Replaced `env $(echo "$CONFIGURATION" | xargs) localstack start -d` with a safe xargs-based array tokenization pattern (`config_vars=()` + guarded while/read loop + `env "${config_vars[@]}" localstack start -d`) to prevent shell metacharacter injection from attacker-controlled configuration input.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings:
1. cloud-pods/action.yml (lines 21, 24): Added double quotes around $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent unquoted variable expansion from allowing shell metacharacter injection.
2. local/action.yml (lines 34, 37): Added double quotes around ${NAME}.zip in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent unquoted variable expansion from allowing shell metacharacter injection.
3. ephemeral/startup/action.yml (line 163): Replaced `bash -c "$PREVIEW_CMD"` with writing the command to a temp script file using `printf '%s\n' "$PREVIEW_CMD" > "$PREVIEW_SCRIPT"` and then executing `bash "$PREVIEW_SCRIPT"`. This prevents bash from interpreting shell metacharacters in the preview-cmd input as shell syntax during the -c evaluation, while still supporting multi-line/multi-command inputs.

