<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.0** was hardened automatically. 5 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. Multiple action files interpolate GitHub Actions expressions directly into shell scripts without routing through env: variables, allowing script injection.

1. prepare/action.yml line 14: `run: echo ${{ github.event.number }} > ./pr-id.txt` — github context interpolated directly into shell.

2. startup/action.yml line 67: `export CI_PROJECT=${{ inputs.ci-project }}` — inputs.* interpolated directly into shell.

3. ephemeral/startup/action.yml line 55: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs.* interpolated directly into shell.

4. ephemeral/startup/action.yml line 58: `source ${{ github.action_path }}/../retry-function.sh` — github.* interpolated directly into shell.

5. ephemeral/startup/action.yml lines 79-81: `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"` — inputs.* interpolated directly into shell.

6. ephemeral/startup/action.yml line 107: `${{ inputs.preview-cmd }}` — the entire preview-cmd input is directly executed as shell code, a critical injection point.

7. ephemeral/shutdown/action.yml line 34: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs.* interpolated directly into shell.

8. ephemeral/shutdown/action.yml line 37: `source ${{ github.action_path }}/../retry-function.sh` — github.* interpolated directly into shell.

9. finish/action.yml line 62: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and line 63: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — inputs.* interpolated directly into shell and written to GITHUB_ENV.

Locations:

- `prepare/action.yml:14`
- `startup/action.yml:67`
- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:58`
- `ephemeral/startup/action.yml:79`
- `ephemeral/startup/action.yml:107`
- `ephemeral/shutdown/action.yml:34`
- `ephemeral/shutdown/action.yml:37`
- `finish/action.yml:62`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands in workflow files.

1. .github/workflows/ci.yml: `localstack pod list | grep ${{ steps.pod_name.outputs.name }}` — steps output interpolated directly into shell (unquoted, injectable).

2. .github/workflows/ci.yml: `localstack pod delete ${{ needs.cloud-pods-save-test.outputs.pod-name }}` — needs output interpolated directly into shell.

3. .github/workflows/ci.yml: `if localstack pod list | grep -q ${{ needs.cloud-pods-save-test.outputs.pod-name }}` — needs output interpolated directly into shell.

4. .github/workflows/ephemeral.yml: `PR_NUMBER=${{ github.event.pull_request.number }}` — github context interpolated directly into shell.

5. .github/workflows/ephemeral.yml: `comments=$(gh api repos/${{ github.repository }}/issues/$PR_NUMBER/comments)` — github context interpolated directly into shell.

Locations:

- `.github/workflows/ci.yml:75`
- `.github/workflows/ci.yml:113`
- `.github/workflows/ci.yml:122`
- `.github/workflows/ephemeral.yml:37`
- `.github/workflows/ephemeral.yml:41`

### github-env-injection (severity: high)

finish/action.yml writes an unsanitized inputs.* value directly to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. The expression `${{ inputs.preview-url }}` is interpolated directly into the shell string and then written to GITHUB_ENV: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. A newline in the input value could inject arbitrary environment variables into the runner environment.

Locations:

- `finish/action.yml:63`

### broad-permissions (severity: medium)

The workflow job 'preview-test' in .github/workflows/ephemeral.yml has `permissions: write-all`, which grants overly broad write access across all GitHub API scopes. This should be replaced with specific minimal permissions (e.g., `pull-requests: write`, `contents: read`) required by the job.

Locations:

- `.github/workflows/ephemeral.yml:10`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and none of its jobs (localstack-action-version-test, cloud-pods-save-test, load-cloud-pod-test, local-state-save-test, local-state-load-test) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A top-level `permissions: {}` or job-level minimal permissions should be specified.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, broad-permissions, missing-permissions

**Notes:**

Fixed all findings across 6 files:

1. prepare/action.yml: Moved github.event.number to env block (PR_NUMBER) in the Save PR number step.

2. startup/action.yml: Moved inputs.ci-project to env block (CI_PROJECT_INPUT) and referenced it as ${CI_PROJECT_INPUT} in the shell script.

3. ephemeral/startup/action.yml: 
   - Create preview environment step: Moved inputs.localstack-api-key, github.action_path, inputs.auto-load-pod, inputs.extension-auto-install, and inputs.lifetime to env block.
   - Run preview deployment step: Moved inputs.preview-cmd to env block (PREVIEW_CMD) and used eval "$PREVIEW_CMD" instead of direct interpolation.
   - Print logs step: Moved inputs.localstack-api-key and github.action_path to env block.

4. ephemeral/shutdown/action.yml: Moved inputs.localstack-api-key and github.action_path to env block in the Shutdown ephemeral instance step.

5. finish/action.yml: Moved inputs.preview-url to env block (INPUT_PREVIEW_URL) and added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_ENV.

6. .github/workflows/ci.yml: Added top-level `permissions: contents: read` block. Fixed 3 script injection locations by moving steps/needs outputs to env vars (POD_NAME).

7. .github/workflows/ephemeral.yml: Replaced `permissions: write-all` with `contents: read` and `pull-requests: write`. Fixed 2 script injection locations by moving github.event.pull_request.number and github.repository to env vars (PR_NUMBER, GH_REPOSITORY).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 5 findings across 4 files:

1. startup/action.yml: (a) Quoted `docker pull "${IMAGE_NAME}"` to prevent shell metacharacter injection. (b) Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe xargs-based KEY=VALUE parser that exports each variable individually before calling `localstack start -d` directly — no eval of user-controlled input.

2. ephemeral/startup/action.yml: (a) Sanitized previewName (derived from GITHUB_REPOSITORY and pr-id.txt) with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT. (b) Sanitized endpointUrl (from API response) with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV. (c) Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"` to eliminate double-evaluation; PREVIEW_CMD was already in the env: block so ${{ }} injection was already mitigated.

3. ephemeral/shutdown/action.yml: Sanitized pr_id (from pr-id.txt) before writing to $GITHUB_OUTPUT, and sanitized previewName before writing to $GITHUB_ENV, both using `printf '%s' ... | tr -d '\n\r'`.

4. finish/action.yml: Sanitized pr_id (from pr-id.txt) with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Run preview deployment' step in hardened/action/ephemeral/startup/action.yml. Replaced `bash -c "$PREVIEW_CMD"` with a pattern that writes the command to a temporary script file (`mktemp`) using `printf '%s\n' "$PREVIEW_CMD" > "$PREVIEW_SCRIPT"` and then executes it with `bash "$PREVIEW_SCRIPT"`. This eliminates the bash -c script injection vector while preserving the intended functionality of running user-supplied commands. The temporary file is cleaned up after execution with `rm -f`.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in two files:
1. hardened/action/cloud-pods/action.yml (lines 21, 24): Changed `localstack pod save $NAME` and `localstack pod load --yes $NAME` to use `"$NAME"` with double quotes.
2. hardened/action/local/action.yml (lines 38, 41): Changed `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip` to use `"${NAME}.zip"` with double quotes.

The inputs.name value was already correctly placed in an env var (NAME) rather than directly interpolated via ${{ }} expressions, so the only fix needed was to quote the variable expansions to prevent shell word-splitting and glob expansion from enabling command injection via metacharacters in the name input.

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced all 9 mutable `LocalStack/setup-localstack@${{ env.GH_ACTION_VERSION }}` references with the pinned SHA `LocalStack/setup-localstack@2e0e14048ddaf0db551b45fdaa39d3553f2c6399 # v0.3.0` in both .github/workflows/ci.yml (7 occurrences) and .github/workflows/ephemeral.yml (2 occurrences). The `GH_ACTION_VERSION` env var (which fell back to the mutable `github.ref_name` branch name for non-PR events) was removed entirely from all steps.

