<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.2.5** was hardened automatically. 10 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.number }}` is directly interpolated inside a `run:` shell command: `echo ${{ github.event.number }} > ./pr-id.txt`. An attacker-controlled PR number is injected directly into the shell command string before the shell parses it.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are directly interpolated inside `run:` shell command strings in the 'Start LocalStack' step. Specifically: `export CI_PROJECT=${{ inputs.ci-project }}` injects the ci-project input directly into the shell, and `source ${{ github.action_path }}/../retry-function.sh` injects the action_path. Additionally, sub-rule (b): `eval "${CONFIGURATION} localstack start -d"` executes the CONFIGURATION env var (sourced from `inputs.configuration`) via eval, allowing shell metacharacter injection.

Locations:

- `startup/action.yml:55`
- `startup/action.yml:56`

### script-injection (severity: high)

Sub-rule (a): In the 'Shutdown ephemeral instance' step, `${{ inputs.localstack-api-key }}` is directly interpolated into the run block shell string: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`; and `source ${{ github.action_path }}/../retry-function.sh` injects github.action_path directly into the shell.

Locations:

- `ephemeral/shutdown/action.yml:34`
- `ephemeral/shutdown/action.yml:37`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are directly interpolated inside `run:` shell command strings across several steps. In 'Create preview environment': `${{ inputs.localstack-api-key }}`, `${{ github.action_path }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` are all injected directly into shell strings. In 'Run preview deployment': `${{ inputs.preview-cmd }}` is executed directly as a shell command — this is a critical arbitrary command injection. In 'Print logs': `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are again injected.

Locations:

- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:76`
- `ephemeral/startup/action.yml:77`
- `ephemeral/startup/action.yml:78`
- `ephemeral/startup/action.yml:138`
- `ephemeral/startup/action.yml:155`
- `ephemeral/startup/action.yml:158`

### script-injection (severity: high)

Sub-rule (a): In the 'Load the Ephemeral Instance URL' step, `${{ inputs.preview-url }}` is directly interpolated inside the run block shell string in two places: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. This allows an attacker-controlled input to be injected into the shell command.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

The 'Run preview deployment' step executes `${{ inputs.preview-cmd }}` directly as a shell command without any sanitization. This is both a script injection and an environment injection risk — the preview-cmd input is fully attacker-controlled and is executed verbatim as shell code, which can write arbitrary content to $GITHUB_ENV, $GITHUB_OUTPUT, or $GITHUB_PATH.

Locations:

- `ephemeral/startup/action.yml:138`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` directly to $GITHUB_ENV without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled newline in inputs.preview-url can inject arbitrary key=value pairs into the GitHub environment.

Locations:

- `finish/action.yml:58`

### github-env-injection (severity: high)

The 'Create preview environment' step writes `$endpointUrl` (derived from an external API response, not sanitized) to $GITHUB_ENV: `echo "LS_PREVIEW_URL=$endpointUrl" >> $GITHUB_ENV` and `echo "AWS_ENDPOINT_URL=$endpointUrl" >> $GITHUB_ENV`. The value is not passed through `printf '%s' ... | tr -d '\n\r'` before the write, allowing newline injection if the API response is malicious.

Locations:

- `ephemeral/startup/action.yml:130`
- `ephemeral/startup/action.yml:131`

### github-env-injection (severity: high)

The 'Setup preview name' step in ephemeral/shutdown/action.yml writes `previewName` to $GITHUB_ENV without sanitization: `echo "previewName=$previewName" >> $GITHUB_ENV`. The previewName is derived from $GITHUB_REPOSITORY (an inherited process env var set by the calling workflow) and the contents of pr-id.txt (an artifact). Neither source is sanitized with `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `ephemeral/shutdown/action.yml:28`

### github-env-injection (severity: high)

The 'Load the PR ID' step writes file contents directly to $GITHUB_OUTPUT without sanitization: `echo "pr_id=$(< pr-id.txt)" >> $GITHUB_OUTPUT` in both ephemeral/shutdown/action.yml and finish/action.yml. The pr-id.txt artifact content is not sanitized before being written to the special environment file, allowing newline injection.

Locations:

- `ephemeral/shutdown/action.yml:22`
- `finish/action.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. prepare/action.yml: Moved github.event.number to PR_NUMBER env var, used printf+tr to write safely.

2. startup/action.yml: Fixed GH_ACTION_ROOT glob to use GITHUB_ACTION_PATH with python3 relpath; moved inputs.ci-project to CI_PROJECT env var; replaced eval "${CONFIGURATION} localstack start -d" with safe xargs-tokenized env array approach.

3. ephemeral/shutdown/action.yml: Sanitized pr_id and previewName before writing to GITHUB_OUTPUT/GITHUB_ENV; moved inputs.localstack-api-key to INPUT_LOCALSTACK_API_KEY env var; fixed source with github.action_path to use $GITHUB_ACTION_PATH.

4. ephemeral/startup/action.yml: Sanitized previewName before writing to GITHUB_ENV/GITHUB_OUTPUT; moved inputs.localstack-api-key, inputs.auto-load-pod, inputs.extension-auto-install, inputs.lifetime to env vars; fixed source with github.action_path; sanitized endpointUrl from API response before writing to GITHUB_ENV; fixed critical preview-cmd injection by moving to PREVIEW_CMD env var and executing via bash -c.

5. finish/action.yml: Sanitized pr_id before writing to GITHUB_OUTPUT; moved inputs.preview-url to INPUT_PREVIEW_URL env var; sanitized all values before writing to GITHUB_ENV.

6. action.yml: Fixed GH_ACTION_ROOT _actions glob to use GITHUB_ACTION_PATH with python3 relpath.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions that could allow command injection via attacker-controlled inputs:
1. hardened/action/cloud-pods/action.yml (lines 19, 22): Quoted `$NAME` → `"$NAME"` in `localstack pod save` and `localstack pod load --yes` commands.
2. hardened/action/local/action.yml (lines 34, 37): Quoted `${NAME}.zip` → `"${NAME}.zip"` in `localstack state export` and `localstack state import` commands.
3. hardened/action/startup/action.yml (line 47): Quoted `${IMAGE_NAME}` → `"${IMAGE_NAME}"` in `docker pull` command.
All variables were already sourced from inputs via the env: block; only the quoting in the shell commands was missing.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/startup/action.yml line 72: Quoted the `${LS_WAIT_TIMEOUT:-30}` expansion to `"${LS_WAIT_TIMEOUT:-30}"` to prevent shell metacharacter injection from an attacker-controlled inherited environment variable.
2. hardened/action/ephemeral/startup/action.yml line 141: Replaced `bash -c "$PREVIEW_CMD"` (which executes the input value as inline shell code) with writing the command to a temporary script file via `printf '%s\n' "$PREVIEW_CMD" > "$_preview_script"` and then executing it with `bash "$_preview_script"`. This separates the data from the code execution mechanism and prevents the value from being parsed as shell code in the -c argument context.

