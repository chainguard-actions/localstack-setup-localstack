<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 5 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ inputs.preview-cmd }}` is interpolated directly inside a `run:` shell command and executed verbatim as a shell command. This allows any caller to inject arbitrary shell commands. Offending line: `${{ inputs.preview-cmd }}`

Locations:

- `ephemeral/startup/action.yml:120`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings without going through env vars. Offending lines include: `export CI_PROJECT=${{ inputs.ci-project }}` (startup/action.yml), `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` (ephemeral/startup/action.yml and ephemeral/shutdown/action.yml), `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"` (ephemeral/startup/action.yml), and `source ${{ github.action_path }}/../retry-function.sh` (ephemeral/startup/action.yml and ephemeral/shutdown/action.yml). Any of these allow an attacker-controlled value to be injected into the shell before quoting is applied.

Locations:

- `startup/action.yml:57`
- `ephemeral/startup/action.yml:50`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:68`
- `ephemeral/startup/action.yml:69`
- `ephemeral/startup/action.yml:70`
- `ephemeral/shutdown/action.yml:36`
- `ephemeral/shutdown/action.yml:43`

### script-injection (severity: high)

Rule (a): `echo ${{ github.event.number }} > ./pr-id.txt` interpolates a github context value directly inside a `run:` shell command. Although `github.event.number` is typically numeric, it is still a template-substituted value that bypasses shell quoting and is a script-injection pattern.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

Rule (b): `eval "${CONFIGURATION} localstack start -d"` in startup/action.yml executes the CONFIGURATION env var (sourced from `${{ inputs.configuration }}`) via eval without quoting or sanitization. An attacker-controlled `configuration` input can inject arbitrary shell commands through the eval.

Locations:

- `startup/action.yml:58`

### github-env-injection (severity: high)

In finish/action.yml, the `run:` block writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. A newline injected into `inputs.preview-url` can define arbitrary environment variables for subsequent steps. The required `printf '%s' ... | tr -d '\n\r'` sanitization step is absent.

Locations:

- `finish/action.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across 5 files:

1. startup/action.yml: (a) Moved `${{ inputs.ci-project }}` to env var CI_PROJECT_INPUT; (b) Replaced dangerous `eval "${CONFIGURATION} localstack start -d"` with `env $(echo "$CONFIGURATION" | xargs) localstack start -d` to avoid eval executing attacker-controlled shell commands.

2. ephemeral/startup/action.yml: Moved all inline expressions to env vars: `${{ inputs.localstack-api-key }}` → LOCALSTACK_API_KEY_INPUT, `${{ github.action_path }}` → ACTION_PATH (used with quoted `source "$ACTION_PATH/../retry-function.sh"`), `${{ inputs.auto-load-pod }}` → AUTO_LOAD_POD_INPUT, `${{ inputs.extension-auto-install }}` → EXTENSION_AUTO_INSTALL_INPUT, `${{ inputs.lifetime }}` → LIFETIME_INPUT. For `${{ inputs.preview-cmd }}`, moved to PREVIEW_CMD env var and executed via temp file with `bash -eo pipefail` to preserve errexit semantics.

3. ephemeral/shutdown/action.yml: Moved `${{ inputs.localstack-api-key }}` → LOCALSTACK_API_KEY_INPUT and `${{ github.action_path }}` → ACTION_PATH to env block.

4. prepare/action.yml: Moved `${{ github.event.number }}` to env var PR_NUMBER and used `"$PR_NUMBER"` in the run script.

5. finish/action.yml: Moved `${{ inputs.preview-url }}` to env var PREVIEW_URL_INPUT, then sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV. Also sanitized the file-read path for completeness.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 4 findings across 4 files:

1. startup/action.yml (script-injection): (a) Quoted `${IMAGE_NAME}` in `docker pull` to prevent shell metacharacter interpretation. (b) Replaced `env $(echo "$CONFIGURATION" | xargs) localstack start -d` with a safe xargs-based array tokenization pattern that properly handles shell metacharacters in CONFIGURATION without allowing injection.

2. ephemeral/startup/action.yml (github-env-injection): Added `printf '%s' ... | tr -d '\n\r'` sanitization for `prId` (from pr-id.txt artifact) and `repoName` (from $GITHUB_REPOSITORY), and sanitized the resulting `previewName` before writing to both $GITHUB_ENV and $GITHUB_OUTPUT.

3. ephemeral/shutdown/action.yml (github-env-injection): (a) Fixed 'Load the PR ID' step to sanitize `pr_id` before writing to $GITHUB_OUTPUT. (b) Fixed 'Setup preview name' step to sanitize `prId` and `repoName` before constructing `previewName`, and sanitized `previewName` before writing to $GITHUB_ENV.

4. finish/action.yml (github-env-injection): Fixed 'Load the PR ID' step to sanitize `pr_id` (from pr-id.txt artifact) before writing to $GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in two files:
1. cloud-pods/action.yml: Quoted $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`
2. local/action.yml: Quoted ${NAME}.zip in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"`

The NAME variable was already correctly sourced via the env: block (not directly from ${{ inputs.name }} in the run: script), but the unquoted expansions allowed shell metacharacters in the value to cause word splitting and potential command injection. Adding double quotes around the variable references fixes both findings.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in hardened/action/startup/action.yml at line 57: changed unquoted `${LS_WAIT_TIMEOUT:-30}` to quoted `"${LS_WAIT_TIMEOUT:-30}"` in the `localstack wait -t` command. The variable is inherited from the calling workflow environment and was unquoted, allowing shell word-splitting and metacharacter injection. Quoting it prevents this attack vector.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed both _actions/* glob searches that would fail in a fork:
1. action.yml line 91: Replaced multi-line ls/grep/tail pipeline with `echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH")" >> $GITHUB_ENV`. Since action.yml is at the repo root, $GITHUB_ACTION_PATH IS the repo root — no /.. needed.
2. startup/action.yml line 44: Replaced the same pattern with `echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")" >> $GITHUB_ENV`. Since startup/action.yml is one directory below the repo root, $GITHUB_ACTION_PATH/.. gives the repo root. Both produce workspace-relative paths (starting with "./") as required for dynamic uses: references.

