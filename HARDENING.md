<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 8 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.number }}` is interpolated directly inside a `run:` shell command string without going through an env: variable. An attacker who controls the PR number field (e.g. via a crafted event payload) can inject arbitrary shell commands.

Offending line: `run: echo ${{ github.event.number }} > ./pr-id.txt`

Locations:

- `prepare/action.yml:14`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` and `${{ github.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Create preview environment' step:
- `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`  — inputs.localstack-api-key injected directly into shell
- `source ${{ github.action_path }}/../retry-function.sh`  — github.action_path injected directly
- `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"` — inputs.auto-load-pod injected directly
- `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"` — inputs.extension-auto-install injected directly
- `lifetime="${{ inputs.lifetime }}"` — inputs.lifetime injected directly

Any of these inputs can contain shell metacharacters that execute arbitrary commands before the shell ever sees the variable assignment.

Locations:

- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:65`
- `ephemeral/startup/action.yml:83`
- `ephemeral/startup/action.yml:84`
- `ephemeral/startup/action.yml:85`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.preview-cmd }}` is interpolated directly as the body of a `run:` shell command in the 'Run preview deployment' step. This means the entire value of the `preview-cmd` input is executed verbatim as a shell command. A calling workflow that passes attacker-controlled content (e.g. from a PR event) to this input achieves direct remote code execution.

Offending line: `${{ inputs.preview-cmd }}`

Locations:

- `ephemeral/startup/action.yml:155`

### script-injection (severity: high)

Sub-rule (a): In the 'Print logs of ephemeral instance' step, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside a `run:` shell command string:
- `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`
- `source ${{ github.action_path }}/../retry-function.sh`

These allow shell metacharacter injection via the inputs or the action_path context.

Locations:

- `ephemeral/startup/action.yml:163`
- `ephemeral/startup/action.yml:165`

### script-injection (severity: high)

Sub-rule (a): In the 'Shutdown ephemeral instance' step, `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` are interpolated directly inside a `run:` shell command string:
- `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`
- `source ${{ github.action_path }}/../retry-function.sh`

An attacker who controls the `localstack-api-key` input can inject shell metacharacters to execute arbitrary commands.

Locations:

- `ephemeral/shutdown/action.yml:33`
- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.ci-project }}` is interpolated directly inside a `run:` shell command string in the 'Start LocalStack' step:

`export CI_PROJECT=${{ inputs.ci-project }}`

The `inputs.ci-project` value is substituted by the Actions runner before the shell sees the line, so shell metacharacters in the input (e.g. `; malicious-cmd`) execute immediately. Additionally, `eval "${CONFIGURATION} localstack start -d"` is called on the same line, and `CONFIGURATION` is sourced from `${{ inputs.configuration }}` via the env: block — the unquoted `eval` of an env var holding attacker-controlled data is a sub-rule (b) violation.

Locations:

- `startup/action.yml:66`
- `startup/action.yml:67`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.preview-url }}` is interpolated directly inside a `run:` shell command string in the 'Load the Ephemeral Instance URL' step:

`if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
`  echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

The expression is substituted before the shell parses the line, so a value containing `}}` or shell metacharacters can break out of the string context and inject commands.

Locations:

- `finish/action.yml:59`
- `finish/action.yml:60`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes `inputs.preview-url` — an untrusted input — directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in the value allows an attacker to inject arbitrary environment variable definitions into the runner environment for all subsequent steps.

Offending line: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `finish/action.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 8 findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env block as PR_NUMBER.

2. startup/action.yml: Moved `${{ inputs.ci-project }}` to env block as CI_PROJECT_INPUT.

3. ephemeral/startup/action.yml (Create preview environment): Moved localstack-api-key, github.action_path, auto-load-pod, extension-auto-install, and lifetime inputs to env block; updated all references in the script.

4. ephemeral/startup/action.yml (Run preview deployment): Moved preview-cmd to env block as PREVIEW_CMD, wrote to temp file, executed with `bash -eo pipefail` to preserve errexit semantics.

5. ephemeral/startup/action.yml (Print logs): Moved localstack-api-key and github.action_path to env block.

6. ephemeral/shutdown/action.yml: Moved localstack-api-key and github.action_path to env block.

7. finish/action.yml: Moved preview-url to env block as PREVIEW_URL_INPUT, resolved URL via shell variable expansion, and sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_ENV (fixes both script-injection and github-env-injection findings).

### Iteration 2

**Fixes applied:** script-injection, suspicious-run-content, github-env-injection

**Notes:**

Fixed three security findings:

1. startup/action.yml - script-injection & suspicious-run-content: Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe xargs-based tokenization approach that parses CONFIGURATION into individual env var tokens and passes them to `env` command, avoiding arbitrary command execution. Also quoted `${IMAGE_NAME}` in `docker pull "${IMAGE_NAME}"` to prevent word-splitting.

2. ephemeral/shutdown/action.yml - github-env-injection: Added `safe_previewName=$(printf '%s' "$previewName" | tr -d '\n\r')` sanitization before writing previewName to $GITHUB_ENV.

3. ephemeral/startup/action.yml - github-env-injection: Added sanitization for previewName before writing to $GITHUB_ENV and $GITHUB_OUTPUT (lines 52-53), and added sanitization for endpointUrl before writing LS_PREVIEW_URL and AWS_ENDPOINT_URL to $GITHUB_ENV (lines 148-149).

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 4 locations across 4 files:
1. cloud-pods/action.yml (lines 19, 21): Added double-quotes around $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent shell metacharacter injection.
2. local/action.yml (lines 33, 36): Added double-quotes around ${NAME}.zip in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent shell metacharacter injection.
3. ephemeral/shutdown/action.yml (line 22): Replaced single-line `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` with a multi-line script that reads the raw value, sanitizes it with `printf '%s' "$raw_pr_id" | tr -d '\n\r'`, then writes the safe value to GITHUB_OUTPUT.
4. finish/action.yml (line 38): Same sanitization fix as ephemeral/shutdown/action.yml.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed both _actions directory glob searches by replacing them with $GITHUB_ACTION_PATH-based paths:
1. action.yml:91 - Replaced multi-line ls/grep/tail glob with: echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH")" >> $GITHUB_ENV (manifest is at repo root, so $GITHUB_ACTION_PATH IS the repo root)
2. startup/action.yml:44 - Replaced multi-line ls/grep/tail glob with: echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")" >> $GITHUB_ENV (manifest is one directory below repo root, so $GITHUB_ACTION_PATH/.. is the repo root)
Both use realpath --relative-to to produce workspace-relative paths starting with './' as required by jenseng/dynamic-uses for local uses: references.

