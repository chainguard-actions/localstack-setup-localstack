<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.0** was hardened automatically. 8 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.number }}` is interpolated directly inside a `run:` shell command string. An attacker-controlled PR number is passed through YAML template substitution before the shell sees it. Offending line: `run: echo ${{ github.event.number }} > ./pr-id.txt`

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.ci-project }}` is interpolated directly inside a `run:` shell command string: `export CI_PROJECT=${{ inputs.ci-project }}`. A caller-controlled input value is injected into the shell before quoting. Additionally, sub-rule (b): `docker pull ${IMAGE_NAME} &` uses IMAGE_NAME unquoted; IMAGE_NAME is derived from IMAGE_TAG which comes from `${{ inputs.image-tag }}` via env var. Also, `eval "${CONFIGURATION} localstack start -d"` executes CONFIGURATION which is set from the caller-controlled `${{ inputs.configuration }}` env var, allowing arbitrary shell command injection.

Locations:

- `startup/action.yml:57`
- `startup/action.yml:56`
- `startup/action.yml:58`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Create preview environment' step. Offending lines include: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` (line 55), `source ${{ github.action_path }}/../retry-function.sh` (line 57), `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"` (line 71), `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"` (line 72), `lifetime="${{ inputs.lifetime }}"` (line 73). All of these allow caller-controlled values to be injected into the shell script before execution.

Locations:

- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:71`
- `ephemeral/startup/action.yml:72`
- `ephemeral/startup/action.yml:73`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.preview-cmd }}` is interpolated directly as the body of a `run:` shell script in the 'Run preview deployment' step: `run: |\n  ${{ inputs.preview-cmd }}`. This directly executes arbitrary caller-supplied shell commands with no quoting or sanitization — the most severe form of script injection.

Locations:

- `ephemeral/startup/action.yml:118`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Print logs of ephemeral instance' step. Offending lines: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` (line 124) and `source ${{ github.action_path }}/../retry-function.sh` (line 126). Caller-controlled values are injected into the shell before execution.

Locations:

- `ephemeral/startup/action.yml:124`
- `ephemeral/startup/action.yml:126`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Shutdown ephemeral instance' step. Offending lines: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` (line 35) and `source ${{ github.action_path }}/../retry-function.sh` (line 37). Caller-controlled values are injected into the shell before execution.

Locations:

- `ephemeral/shutdown/action.yml:35`
- `ephemeral/shutdown/action.yml:37`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.preview-url }}` is interpolated directly inside a `run:` shell command string in the 'Load the Ephemeral Instance URL' step. Offending lines: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then` (line 57) and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` (line 58). A caller-controlled input is injected into the shell before execution.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes a value derived from the caller-controlled input `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Offending line: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker can inject newlines into the input to set arbitrary environment variables for subsequent steps.

Locations:

- `finish/action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 7 script-injection findings and 1 github-env-injection finding across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env var PR_NUMBER.

2. startup/action.yml: Fixed _actions glob to use GITHUB_ACTION_PATH with python3 relpath; moved `${{ inputs.ci-project }}` to env var INPUT_CI_PROJECT; quoted IMAGE_NAME in docker pull.

3. ephemeral/startup/action.yml: 'Create preview environment' step - moved inputs.localstack-api-key, inputs.auto-load-pod, inputs.extension-auto-install, inputs.lifetime to env vars; replaced `${{ github.action_path }}` with $GITHUB_ACTION_PATH. 'Run preview deployment' step - moved inputs.preview-cmd to env var PREVIEW_CMD and used eval. 'Print logs' step - moved inputs.localstack-api-key to env var; replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH.

4. ephemeral/shutdown/action.yml: Moved inputs.localstack-api-key to env var INPUT_LOCALSTACK_API_KEY; replaced ${{ github.action_path }} with $GITHUB_ACTION_PATH.

5. finish/action.yml: Moved inputs.preview-url to env var INPUT_PREVIEW_URL; sanitized with printf/tr before writing to GITHUB_ENV (fixes both script-injection and github-env-injection).

6. action.yml (root): Fixed _actions glob to use GITHUB_ACTION_PATH with python3 relpath.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 5 findings across 3 files:

1. startup/action.yml: Replaced `eval "${CONFIGURATION} localstack start -d"` with safe xargs-based tokenization into an array, then `env "${env_args[@]}" localstack start -d`. This avoids eval with user-controlled CONFIGURATION input.

2. ephemeral/startup/action.yml (preview-cmd): Replaced `eval "$PREVIEW_CMD"` with writing PREVIEW_CMD to a temp script file and executing it with `bash`, avoiding eval re-parsing of user-controlled input.

3. ephemeral/startup/action.yml (previewName): Added `safe_previewName=$(printf '%s' "$previewName" | tr -d '\n\r')` before writing to GITHUB_ENV and GITHUB_OUTPUT.

4. ephemeral/startup/action.yml (endpointUrl): Split into `raw_endpointUrl=$(jq ...)` then `endpointUrl=$(printf '%s' "$raw_endpointUrl" | tr -d '\n\r')` before writing to GITHUB_ENV.

5. ephemeral/shutdown/action.yml (previewName): Added `safe_previewName=$(printf '%s' "$previewName" | tr -d '\n\r')` before writing to GITHUB_ENV.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in two files:
1. hardened/action/cloud-pods/action.yml (lines 20, 23): Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`.
2. hardened/action/local/action.yml (lines 34, 37): Changed `localstack state export ${NAME}.zip` to `localstack state export "${NAME}.zip"` and `localstack state import ${NAME}.zip` to `localstack state import "${NAME}.zip"`.
In both cases, the NAME variable was already safely set via the env: block from inputs.name, but the unquoted expansion allowed word-splitting and glob expansion on values containing shell metacharacters. Double-quoting prevents this.

### Iteration 4

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three issues across three files:
1. ephemeral/shutdown/action.yml (line 24): Changed 'echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT' to a multi-line run that reads the file into a variable, sanitizes with 'printf | tr -d \n\r', then writes the safe value to $GITHUB_OUTPUT.
2. finish/action.yml (line 42): Same fix applied to the 'Load the PR ID' step.
3. startup/action.yml (line 83): Quoted the unquoted shell expansion '${LS_WAIT_TIMEOUT:-30}' to '"${LS_WAIT_TIMEOUT:-30}"' to prevent shell metacharacter injection from a workflow-controllable environment variable.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed GH_ACTION_ROOT assignment in both action.yml and startup/action.yml. The variable was being set as `GH_ACTION_ROOT=$(./$(python3 ...))` which tried to execute the relative path as a command. Changed to `GH_ACTION_ROOT=./$(python3 ...)` so the `./` prefix is part of the variable value, making it a workspace-relative path (e.g., `./path/to/action`) that `uses:` accepts. This fixes all 7 reported locations: 6 in action.yml (lines 96, 107, 124, 139, 150, 162) and 1 in startup/action.yml (line 49).

