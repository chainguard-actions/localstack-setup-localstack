<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 12 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ inputs.ci-project }}` is interpolated directly inside a `run:` shell command string: `export CI_PROJECT=${{ inputs.ci-project }}`. This allows an attacker-controlled value to be injected into the shell. Additionally, `eval "${CONFIGURATION} localstack start -d"` executes a shell-constructed string that incorporates the `CONFIGURATION` env var (itself built from `${CONFIGURATION}` which comes from `inputs.configuration` via env), compounding the injection risk.

Locations:

- `startup/action.yml:68`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Create preview environment' step: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`, `source ${{ github.action_path }}/../retry-function.sh`, `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, and `lifetime="${{ inputs.lifetime }}"`.

Locations:

- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:62`
- `ephemeral/startup/action.yml:79`
- `ephemeral/startup/action.yml:80`
- `ephemeral/startup/action.yml:81`

### script-injection (severity: high)

Rule (a): `${{ inputs.preview-cmd }}` is interpolated directly as the body of a `run:` shell script in the 'Run preview deployment' step. This allows an attacker to supply arbitrary shell commands via the `preview-cmd` input, achieving full remote code execution on the runner.

Locations:

- `ephemeral/startup/action.yml:163`

### script-injection (severity: high)

Rule (a): In the 'Print logs of ephemeral instance' step, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside `run:` shell command strings: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

Locations:

- `ephemeral/startup/action.yml:170`
- `ephemeral/startup/action.yml:175`

### script-injection (severity: high)

Rule (a): In the 'Shutdown ephemeral instance' step, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside `run:` shell command strings: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh`.

Locations:

- `ephemeral/shutdown/action.yml:32`
- `ephemeral/shutdown/action.yml:37`

### script-injection (severity: high)

Rule (a): `${{ inputs.preview-url }}` is interpolated directly inside a `run:` shell command string in the 'Load the Ephemeral Instance URL' step: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled input value is embedded directly in the shell script.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### script-injection (severity: high)

Rule (a): `${{ github.event.number }}` is interpolated directly inside a `run:` shell command string: `echo ${{ github.event.number }} > ./pr-id.txt`. The PR number is attacker-controlled in pull_request events and is injected directly into the shell without quoting or env-var indirection.

Locations:

- `prepare/action.yml:20`

### script-injection (severity: high)

Rule (a): Multiple `${{ steps.* }}` and `${{ needs.* }}` expressions are interpolated directly inside `run:` shell command strings in ci.yml: `localstack pod list | grep ${{ steps.pod_name.outputs.name }}` (line 76), `run: localstack pod delete ${{ needs.cloud-pods-save-test.outputs.pod-name }}` (line 107), and `if localstack pod list | grep -q ${{ needs.cloud-pods-save-test.outputs.pod-name }}` / `echo "Cleanup failed! Pod ${{ needs.cloud-pods-save-test.outputs.pod-name }}"` (lines 115-116). These step/job outputs can be attacker-influenced via PR events.

Locations:

- `.github/workflows/ci.yml:76`
- `.github/workflows/ci.yml:107`
- `.github/workflows/ci.yml:115`

### script-injection (severity: high)

Rule (a): In ephemeral.yml, `${{ github.event.pull_request.number }}` and `${{ github.repository }}` are interpolated directly inside `run:` shell command strings: `PR_NUMBER=${{ github.event.pull_request.number }}` and `comments=$(gh api repos/${{ github.repository }}/issues/$PR_NUMBER/comments)`. Both values are attacker-controllable in pull_request events.

Locations:

- `.github/workflows/ephemeral.yml:47`
- `.github/workflows/ephemeral.yml:51`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled input containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `finish/action.yml:58`

### permissions (severity: medium)

missing-permissions: The workflow file ci.yml has no top-level `permissions:` key and none of its jobs define a `permissions:` block. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### permissions (severity: medium)

broad-permissions: The `preview-test` job in ephemeral.yml sets `permissions: write-all`, granting overly broad access to all GitHub API scopes. This should be replaced with specific minimal permissions (e.g., `pull-requests: write`, `contents: read`).

Locations:

- `.github/workflows/ephemeral.yml:8`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, permissions

**Notes:**

Fixed all 11 findings across 7 files:

1. startup/action.yml: Moved ${{ inputs.ci-project }} to env block as CI_PROJECT_INPUT; replaced eval "${CONFIGURATION} localstack start -d" with env ${CONFIGURATION} localstack start -d to eliminate eval of shell-constructed strings.

2. ephemeral/startup/action.yml (Create preview environment step): Moved ${{ inputs.localstack-api-key }}, ${{ github.action_path }}, ${{ inputs.auto-load-pod }}, ${{ inputs.extension-auto-install }}, and ${{ inputs.lifetime }} to env block.

3. ephemeral/startup/action.yml (Run preview deployment step): Moved ${{ inputs.preview-cmd }} to env block as PREVIEW_CMD and used eval "$PREVIEW_CMD".

4. ephemeral/startup/action.yml (Print logs step): Moved ${{ inputs.localstack-api-key }} and ${{ github.action_path }} to env block.

5. ephemeral/shutdown/action.yml: Moved ${{ inputs.localstack-api-key }} and ${{ github.action_path }} to env block.

6. finish/action.yml: Moved ${{ inputs.preview-url }} to env block as PREVIEW_URL_INPUT; sanitized with tr -d '\n\r' before writing to $GITHUB_ENV to fix github-env-injection.

7. prepare/action.yml: Moved ${{ github.event.number }} to env block as PR_NUMBER.

8. .github/workflows/ci.yml: Added top-level permissions: contents: read; moved ${{ steps.pod_name.outputs.name }} and ${{ needs.cloud-pods-save-test.outputs.pod-name }} to env blocks as POD_NAME in all three affected steps.

9. .github/workflows/ephemeral.yml: Replaced permissions: write-all with contents: read + pull-requests: write; moved ${{ github.event.pull_request.number }} and ${{ github.repository }} to env block as PR_NUMBER and GH_REPOSITORY.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:

1. ephemeral/startup/action.yml (line 155): Replaced `eval "$PREVIEW_CMD"` with safe xargs-based tokenization. The command is now parsed into a bash array using `printf '%s' "$PREVIEW_CMD" | xargs printf '%s\0'` and executed directly as `"${cmd[@]}"`, preventing shell metacharacter injection.

2. startup/action.yml (lines 74, 76): 
   - Quoted `${IMAGE_NAME}` → `"${IMAGE_NAME}"` in the `docker pull` command to prevent word splitting and glob expansion.
   - Replaced unquoted `env ${CONFIGURATION} localstack start -d` with safe xargs-based tokenization that parses CONFIGURATION into a `config_args` array and passes it as `env "${config_args[@]}" localstack start -d`, preventing injection of additional env var assignments or shell metacharacters.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings by quoting unquoted shell variable expansions:
1. cloud-pods/action.yml (lines 17, 21): Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`.
2. local/action.yml (lines 35, 39): Changed `localstack state export ${NAME}.zip` to `localstack state export "${NAME}.zip"` and `localstack state import ${NAME}.zip` to `localstack state import "${NAME}.zip"`.
3. startup/action.yml (line 72): Changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. All variables were already sourced from the step's env: block (or inherited env), and quoting them prevents shell metacharacter injection.

