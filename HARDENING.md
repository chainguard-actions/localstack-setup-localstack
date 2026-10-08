<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 8 finding(s) were identified and resolved across 6 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ github.event.number }}` is interpolated directly inside a `run:` shell command string. An attacker-controlled PR number is passed directly to the shell before quoting. Offending line: `run: echo ${{ github.event.number }} > ./pr-id.txt`

Locations:

- `prepare/action.yml:15`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` expressions are interpolated directly inside a `run:` shell command string in the 'Start LocalStack' step. Specifically: `export CI_PROJECT=${{ inputs.ci-project }}` injects an attacker-controlled value directly into the shell. Additionally, rule (b): `docker pull ${IMAGE_NAME}` uses an unquoted shell variable where IMAGE_NAME is derived from `inputs.image-tag` via env, and `eval "${CONFIGURATION} localstack start -d"` evaluates a shell string built from `inputs.configuration` (via the CONFIGURATION env var), allowing arbitrary command injection.

Locations:

- `startup/action.yml:57`
- `startup/action.yml:56`
- `startup/action.yml:58`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Create preview environment' step: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`, `source ${{ github.action_path }}/../retry-function.sh`, `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, and `lifetime="${{ inputs.lifetime }}"`.

Locations:

- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:63`
- `ephemeral/startup/action.yml:77`
- `ephemeral/startup/action.yml:78`
- `ephemeral/startup/action.yml:79`

### script-injection (severity: high)

Rule (a): `${{ inputs.preview-cmd }}` is interpolated directly as the entire body of a `run:` shell command string in the 'Run preview deployment' step. This allows an attacker to supply arbitrary shell commands via the `preview-cmd` input. Offending line: `${{ inputs.preview-cmd }}`

Locations:

- `ephemeral/startup/action.yml:131`

### script-injection (severity: high)

Rule (a): In the 'Print logs of ephemeral instance' step, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside a `run:` shell command string. Offending lines: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

Locations:

- `ephemeral/startup/action.yml:140`
- `ephemeral/startup/action.yml:143`

### script-injection (severity: high)

Rule (a): In the 'Shutdown ephemeral instance' step, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside a `run:` shell command string. Offending lines: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

Locations:

- `ephemeral/shutdown/action.yml:33`
- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

Rule (a): In the 'Load the Ephemeral Instance URL' step, `${{ inputs.preview-url }}` is interpolated directly inside a `run:` shell command string in both a conditional test and an echo command. Offending lines: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled `preview-url` input containing newlines could inject arbitrary environment variables. Offending line: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `finish/action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 8 findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env block as PR_NUMBER.

2. startup/action.yml: (a) Replaced _actions glob with GITHUB_ACTION_PATH-based path; (b) Moved `${{ inputs.ci-project }}` to env block as CI_PROJECT_INPUT; (c) Quoted `docker pull "${IMAGE_NAME}"`; (d) Replaced `eval "${CONFIGURATION} localstack start -d"` with `env $(echo "$CONFIGURATION" | xargs) localstack start -d`.

3. ephemeral/startup/action.yml: (a) 'Create preview environment' step: moved localstack-api-key, github.action_path, auto-load-pod, extension-auto-install, and lifetime to env block; (b) 'Run preview deployment' step: moved preview-cmd to env block and executed via `bash -c "$PREVIEW_CMD"`; (c) 'Print logs' step: moved localstack-api-key and github.action_path to env block.

4. ephemeral/shutdown/action.yml: Moved localstack-api-key and github.action_path to env block in 'Shutdown ephemeral instance' step.

5. finish/action.yml: Moved preview-url to env block as PREVIEW_URL_INPUT and added `printf '%s' "$raw" | tr -d '\n\r'` sanitization before writing to GITHUB_ENV.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection findings:
1. cloud-pods/action.yml (lines 20, 22): Quoted `$NAME` → `"$NAME"` in `localstack pod save` and `localstack pod load --yes` commands.
2. local/action.yml (lines 33, 36): Quoted `${NAME}` → `"${NAME}"` in `localstack state export` and `localstack state import` commands (as `"${NAME}.zip"`).
3. startup/action.yml (line 57): Replaced unsafe `env $(echo "$CONFIGURATION" | xargs) localstack start -d` with a safe bash array + xargs NUL-delimited tokenization pattern: `cfg_args=(); while IFS= read -r -d '' t; do cfg_args+=("$t"); done < <(printf '%s' "$CONFIGURATION" | xargs printf '%s\0'); env "${cfg_args[@]}" localstack start -d`.
4. ephemeral/startup/action.yml (line 155): Replaced `bash -c "$PREVIEW_CMD"` with writing PREVIEW_CMD to a mktemp file and executing `bash "$_preview_script"`, eliminating the bash -c injection vector while preserving functionality.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings: (1) startup/action.yml line 57: quoted `${LS_WAIT_TIMEOUT:-30}` as `"${LS_WAIT_TIMEOUT:-30}"` to prevent shell metacharacter injection. (2) ephemeral/startup/action.yml lines 131-132: sanitized `endpointUrl` (from external API response) with `printf '%s' "$endpointUrl" | tr -d '\n\r'` before writing to GITHUB_ENV and ls-preview-url.txt. (3) finish/action.yml line 72: sanitized content read from ls-preview-url.txt with `printf '%s' "$raw_file" | tr -d '\n\r'` before writing to GITHUB_ENV.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed two instances of github-env-injection: (1) ephemeral/shutdown/action.yml: replaced `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` with a multi-line script that reads the file into `raw`, sanitizes with `printf '%s' "$raw" | tr -d '\n\r'`, and writes the safe value; (2) finish/action.yml: same fix applied to `echo "pr_id=$(< pr-id.txt)" >> $GITHUB_OUTPUT`. Both fixes prevent newline injection into $GITHUB_OUTPUT via malicious artifact content.

### Iteration 5

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in two files:
1. ephemeral/startup/action.yml: Added `safePreviewName=$(printf '%s' "$previewName" | tr -d '\n\r')` and used `safePreviewName` when writing to both $GITHUB_ENV and $GITHUB_OUTPUT. Also properly quoted the env file paths.
2. ephemeral/shutdown/action.yml: Added the same sanitization step and used `safePreviewName` when writing to $GITHUB_ENV. Also properly quoted the env file path.
Both fixes follow the required pattern: compute the raw composed value first, then sanitize into a separate variable, then write the sanitized variable to the GitHub environment files.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Replaced the _actions directory glob search (`ls -d ./../../_actions/* | grep -i localstack | tail -n1)/setup-localstack/*`) with `$GITHUB_ACTION_PATH`. Since action.yml is at the root of the action and all sub-actions reference paths relative to the root (e.g., /tools, /startup, /ephemeral/startup), setting GH_ACTION_ROOT=$GITHUB_ACTION_PATH is the correct and portable replacement that works under any owner/repo name.

