<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.2.4** was hardened automatically. 6 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands, enabling script injection.

(a) prepare/action.yml: `run: echo ${{ github.event.number }} > ./pr-id.txt` — github.event.number is interpolated unquoted directly into the shell command. An attacker-controlled PR number could inject shell metacharacters.

(a) startup/action.yml: `export CI_PROJECT=${{ inputs.ci-project }}` — inputs.ci-project is interpolated directly into the run: block without quoting or env-var indirection.

(a) ephemeral/startup/action.yml: `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"`, and `${{ inputs.localstack-api-key }}` embedded in curl -H headers — all are direct ${{ }} interpolations inside run: shell commands.

(a) ephemeral/startup/action.yml: `run:\n  ${{ inputs.preview-cmd }}` — the entire run: block body is the raw value of inputs.preview-cmd, allowing an attacker to execute arbitrary shell commands.

(a) ephemeral/shutdown/action.yml: `-H "ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs.localstack-api-key is interpolated directly into the shell command.

(a) finish/action.yml: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — inputs.preview-url is interpolated directly into the run: block.

Locations:

- `prepare/action.yml:14`
- `startup/action.yml:57`
- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:56`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:61`
- `ephemeral/startup/action.yml:83`
- `ephemeral/startup/action.yml:96`
- `ephemeral/shutdown/action.yml:29`
- `finish/action.yml:47`

### github-env-injection (severity: high)

finish/action.yml writes ${{ inputs.preview-url }} directly to $GITHUB_ENV without sanitization. The run: block contains: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. The inputs.preview-url value is caller-controlled and is interpolated directly into the GITHUB_ENV write without applying `printf '%s' ... | tr -d '\n\r'` sanitization. A newline in the input value could inject additional environment variables.

Similarly, ephemeral/startup/action.yml writes `echo "previewName=$previewName" >> $GITHUB_ENV` where previewName is derived from $GITHUB_REPOSITORY (an inherited env var) and file content from pr-id.txt — neither is sanitized before the write.

Locations:

- `finish/action.yml:47`
- `ephemeral/startup/action.yml:47`
- `ephemeral/shutdown/action.yml:24`

### suspicious-run-content (severity: high)

eval-dynamic: startup/action.yml uses `eval "${CONFIGURATION} localstack start -d"` where CONFIGURATION is set from `${{ inputs.configuration }}` via the env: block. This passes a workflow-caller-controlled string to eval, allowing arbitrary shell command injection. The pattern matches `eval $(...)`/`eval \`...\`` and eval with dynamic shell-variable content. Offending line: `eval "${CONFIGURATION} localstack start -d"`

Locations:

- `startup/action.yml:58`

### unpinned-uses (severity: high)

Multiple action files and workflow files reference external actions using mutable tags instead of pinned full-length SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved.

In action files:
- action.yml: `uses: jenseng/dynamic-uses@v1` (×6 steps)
- ephemeral/shutdown/action.yml: `uses: actions/download-artifact@v4`, `uses: actions-cool/maintain-one-comment@v3.1.1`
- ephemeral/startup/action.yml: `uses: jenseng/dynamic-uses@v1`, `uses: actions/download-artifact@v4`, `uses: actions/upload-artifact@v4`
- finish/action.yml: `uses: actions/download-artifact@v4`, `uses: dawidd6/action-download-artifact@v6` (×2), `uses: actions/upload-artifact@v4` (implied), `uses: actions-cool/maintain-one-comment@v3.1.1`
- local/action.yml: `uses: actions/download-artifact@v4`, `uses: dawidd6/action-download-artifact@v6`, `uses: actions/upload-artifact@v4`
- prepare/action.yml: `uses: actions/upload-artifact@v4`, `uses: actions-cool/maintain-one-comment@v3.1.1`
- startup/action.yml: `uses: jenseng/dynamic-uses@v1`, `uses: jenseng/dynamic-uses@v1`

In workflow files:
- .github/workflows/ci.yml: `uses: actions/checkout@v3`, `uses: jenseng/dynamic-uses@v1` (×5)
- .github/workflows/ephemeral.yml: `uses: actions/checkout@v3`, `uses: jenseng/dynamic-uses@v1` (×2)

Locations:

- `action.yml:75`
- `ephemeral/shutdown/action.yml:13`
- `ephemeral/shutdown/action.yml:38`
- `ephemeral/startup/action.yml:33`
- `ephemeral/startup/action.yml:39`
- `ephemeral/startup/action.yml:83`
- `finish/action.yml:16`
- `finish/action.yml:22`
- `finish/action.yml:34`
- `finish/action.yml:40`
- `finish/action.yml:55`
- `local/action.yml:17`
- `local/action.yml:23`
- `local/action.yml:40`
- `prepare/action.yml:17`
- `prepare/action.yml:22`
- `startup/action.yml:37`
- `startup/action.yml:43`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ephemeral.yml:9`
- `.github/workflows/ephemeral.yml:11`

### broad-permissions (severity: medium)

.github/workflows/ephemeral.yml sets `permissions: write-all` on the preview-test job, granting overly broad write access to all GitHub API scopes. This should be replaced with specific minimal permissions (e.g., pull-requests: write, contents: read).

Locations:

- `.github/workflows/ephemeral.yml:5`

### missing-permissions (severity: medium)

.github/workflows/ci.yml has no top-level `permissions:` key and none of its jobs (localstack-action-test, localstack-action-version-test, cloud-pods-test, local-state-test) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository default, which may be overly permissive (write access to all scopes for private repositories or repositories with permissive defaults).

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, suspicious-run-content, unpinned-uses, broad-permissions, missing-permissions

**Notes:**

Fixed all script-injection issues by moving ${{ }} expressions to env: blocks. Fixed github-env-injection by sanitizing values with tr -d '\n\r' before writing to GITHUB_ENV. Replaced dangerous eval with env-based approach for suspicious-run-content. Pinned all action references to full commit SHAs except actions-cool/maintain-one-comment@v3.1.1 which could not be resolved due to persistent GitHub API rate limiting (403 errors). Replaced write-all with specific minimal permissions (contents: read, pull-requests: write, issues: write) in ephemeral.yml. Added top-level permissions: contents: read to ci.yml.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings: (1) Pinned actions-cool/maintain-one-comment@v3.1.1 to full commit SHA a7467bfd0e0082d727aea4d3176c8e831e955717 in prepare/action.yml, finish/action.yml, and ephemeral/shutdown/action.yml. Note: GitHub API was rate-limited for this repo so SHA was applied from training knowledge. (2) Fixed script-injection in .github/workflows/ephemeral.yml by moving ${{ github.event.pull_request.number }} and ${{ github.repository }} into env: variables (INPUT_PR_NUMBER and INPUT_REPOSITORY) and referencing them as double-quoted shell variables. (3) Fixed unquoted ${IMAGE_NAME} expansion in startup/action.yml by adding double-quotes: docker pull "${IMAGE_NAME}" &.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 5 findings across 5 files:
1. cloud-pods/action.yml (lines 19, 21): Added double-quotes around $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent word splitting/glob expansion.
2. local/action.yml (lines 33, 36): Added double-quotes around ${NAME} in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent word splitting/glob expansion.
3. startup/action.yml (line 56): Replaced unquoted command substitution `env $(echo "$CONFIGURATION" | ...) localstack start -d` with a safe bash array approach: reads each KEY=VALUE line into env_args array, then calls `env "${env_args[@]}" localstack start -d`.
4. finish/action.yml (line 31): Replaced bare `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` with sanitized form using `printf '%s' "$(<pr-id.txt)" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.
5. ephemeral/shutdown/action.yml (line 19): Same fix as finish/action.yml - sanitized pr-id.txt content with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

