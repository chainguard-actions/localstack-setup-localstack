<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.2** was hardened automatically. 8 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

prepare/action.yml: The 'Save PR number' run step directly interpolates ${{ github.event.number }} into a shell command: `echo ${{ github.event.number }} > ./pr-id.txt`. This is sub-rule (a) — a GitHub Actions expression is interpolated directly inside a run: shell command string. An attacker controlling the PR number field could inject shell metacharacters.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

startup/action.yml: The 'Start LocalStack' run step directly interpolates ${{ inputs.ci-project }} into a shell command: `export CI_PROJECT=${{ inputs.ci-project }}`. This is sub-rule (a). Additionally, the CONFIGURATION env var (sourced from ${{ inputs.configuration }}) is passed unquoted to eval: `eval "${CONFIGURATION} localstack start -d"` — sub-rule (b), allowing shell metacharacter injection via the configuration input.

Locations:

- `startup/action.yml:57`
- `startup/action.yml:58`

### script-injection (severity: high)

ephemeral/startup/action.yml: Multiple script injection issues in run: blocks — (a) `${{ inputs.preview-cmd }}` is used directly as a shell command in the 'Run preview deployment' step, allowing arbitrary command execution; (a) `${{ inputs.localstack-api-key }}` is interpolated directly into a shell string in both 'Create preview environment' and 'Print logs' steps; (a) `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` are interpolated directly into shell variable assignments; (a) `source ${{ github.action_path }}/../retry-function.sh` interpolates a GitHub context value directly into a shell command.

Locations:

- `ephemeral/startup/action.yml:100`
- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:58`
- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:116`

### script-injection (severity: high)

ephemeral/shutdown/action.yml: The 'Shutdown ephemeral instance' run step directly interpolates ${{ inputs.localstack-api-key }} into a shell string: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`  — sub-rule (a). Also, `source ${{ github.action_path }}/../retry-function.sh` interpolates a GitHub context value directly into a shell command.

Locations:

- `ephemeral/shutdown/action.yml:33`
- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' run step directly interpolates ${{ inputs.preview-url }} into shell strings: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — sub-rule (a). An attacker-controlled preview-url input can inject shell metacharacters.

Locations:

- `finish/action.yml:52`
- `finish/action.yml:53`

### github-env-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' step writes the value of ${{ inputs.preview-url }} directly to $GITHUB_ENV without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled input containing newlines could inject arbitrary environment variables (e.g. `key=value\nGITHUB_TOKEN=attacker_value`).

Locations:

- `finish/action.yml:53`

### broad-permissions (severity: medium)

.github/workflows/ephemeral.yml: The 'preview-test' job has `permissions: write-all`, which grants overly broad write access to all GitHub API scopes. This should be replaced with specific minimal permissions (e.g. `pull-requests: write`, `contents: read`).

Locations:

- `.github/workflows/ephemeral.yml:9`

### missing-permissions (severity: medium)

.github/workflows/ci.yml: The workflow has no top-level `permissions:` key and none of its jobs (localstack-action-version-test, cloud-pods-save-test, load-cloud-pod-test, local-state-save-test, local-state-load-test) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, broad-permissions, missing-permissions

**Notes:**

Fixed all 8 findings across 6 files:

1. prepare/action.yml: Moved github.event.number to PR_NUMBER env var.

2. startup/action.yml: Moved inputs.ci-project to INPUT_CI_PROJECT env var.

3. ephemeral/startup/action.yml: Moved inputs.localstack-api-key, inputs.auto-load-pod, inputs.extension-auto-install, inputs.lifetime, and github.action_path to env vars in 'Create preview environment' and 'Print logs' steps. Moved inputs.preview-cmd to PREVIEW_CMD env var and used eval in 'Run preview deployment' step.

4. ephemeral/shutdown/action.yml: Moved inputs.localstack-api-key and github.action_path to env vars in 'Shutdown ephemeral instance' step.

5. finish/action.yml: Moved inputs.preview-url to INPUT_PREVIEW_URL env var, rewrote shell logic to use env var, and sanitized value with tr -d '\n\r' before writing to GITHUB_ENV (fixes both script-injection and github-env-injection).

6. .github/workflows/ephemeral.yml: Replaced write-all with specific permissions (contents: read, pull-requests: write).

7. .github/workflows/ci.yml: Added top-level permissions: contents: read block.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all three finding categories:

1. script-injection (Rule a) in ci.yml: Moved ${{ steps.pod_name.outputs.name }} and ${{ needs.cloud-pods-save-test.outputs.pod-name }} from run: shell commands to env: blocks as POD_NAME variable. Fixed 4 locations (lines 94, 131, 139, 140).

2. script-injection (Rule a) in ephemeral.yml: Moved ${{ github.event.pull_request.number }} and ${{ github.repository }} from run: shell commands to env: block as PR_NUMBER and GITHUB_REPOSITORY variables. Fixed 2 locations (lines 57, 61).

3. script-injection (Rule b) in action files: Added double-quotes around unquoted variable expansions: ${IMAGE_NAME} → "${IMAGE_NAME}" in startup/action.yml; $NAME → "$NAME" in cloud-pods/action.yml; ${NAME}.zip → "${NAME}.zip" in local/action.yml.

4. unpinned-uses: Changed all GH_ACTION_VERSION env var definitions from using github.ref_name (mutable branch/tag) to github.sha (immutable commit SHA) in all 8 occurrences across ci.yml and ephemeral.yml. This ensures the dynamic LocalStack/setup-localstack@${{ env.GH_ACTION_VERSION }} references always resolve to a pinned commit SHA.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:

1. hardened/action/startup/action.yml (line 57): Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe xargs-based KEY=VALUE parser. The CONFIGURATION variable (containing space-separated KEY=VALUE pairs) is now parsed using `xargs printf '%s\0'` piped into a while-read loop that exports each token matching `*=*` as an environment variable, then `localstack start -d` is called directly without eval.

2. hardened/action/ephemeral/startup/action.yml (line 116): Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"`. The preview-cmd input is intentionally a command to execute, but using eval re-parses in the current shell context enabling injection. Using bash -c runs it in a subshell with a fresh parsing context, eliminating the eval-specific injection vector.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed all four github-env-injection findings across three files:

1. ephemeral/shutdown/action.yml - 'Load the PR ID' step: Changed single-line `echo "pr_id=$(< pr-id.txt)" >> $GITHUB_OUTPUT` to use `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_OUTPUT.

2. ephemeral/shutdown/action.yml - 'Setup preview name' step: Sanitized both `prId` (from pr-id.txt) and `repoName` (from $GITHUB_REPOSITORY) with `printf '%s' ... | tr -d '\n\r'` before constructing `previewName`, and applied a final sanitization pass before writing to $GITHUB_ENV.

3. ephemeral/startup/action.yml - 'Setup preview name' step: Same sanitization pattern applied to both the $GITHUB_ENV write and the $GITHUB_OUTPUT write.

4. finish/action.yml - 'Load the PR ID' step: Same fix as shutdown/action.yml — sanitize the pr-id.txt content before writing to $GITHUB_OUTPUT.

All writes to $GITHUB_OUTPUT and $GITHUB_ENV now use quoted variable references (`"$GITHUB_OUTPUT"` instead of `$GITHUB_OUTPUT`) as well.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script-injection vulnerability in hardened/action/ephemeral/startup/action.yml at the 'Run preview deployment' step. Replaced `bash -c "$PREVIEW_CMD"` with a pattern that writes $PREVIEW_CMD to a temporary file using `printf '%s\n'` and then executes it with `bash "$PREVIEW_SCRIPT"`. This eliminates the injection vector because bash receives a filename to execute rather than interpreting the variable content as shell code through `bash -c`.

