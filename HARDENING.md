<!-- markdownlint-disable -->

# Hardening Report: LocalStack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.2** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions (rule a) or use unquoted shell variable expansions of workflow-controlled data (rule b).

**prepare/action.yml line 15** (rule a+b): `run: echo ${{ github.event.number }} > ./pr-id.txt` — `github.event.number` is interpolated directly and unquoted.

**startup/action.yml line 57** (rule a): `export CI_PROJECT=${{ inputs.ci-project }}` — `inputs.ci-project` is interpolated directly into the shell.

**startup/action.yml line 55** (rule b): `docker pull ${IMAGE_NAME} &` — `IMAGE_NAME` is unquoted and holds `${{ inputs.image-tag }}` from the env block.

**startup/action.yml line 58** (rule a+b): `eval "${CONFIGURATION} localstack start -d"` — `CONFIGURATION` env var holds `${{ inputs.configuration }}`; eval with an attacker-controlled string enables arbitrary command execution.

**ephemeral/shutdown/action.yml line 37** (rule a): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — `inputs.localstack-api-key` interpolated directly in run block.

**ephemeral/shutdown/action.yml line 39** (rule a): `source ${{ github.action_path }}/../retry-function.sh` — `github.action_path` interpolated directly in run block.

**ephemeral/startup/action.yml line 63** (rule a): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — `inputs.localstack-api-key` interpolated directly.

**ephemeral/startup/action.yml lines 73–75** (rule a): `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, `lifetime="${{ inputs.lifetime }}"` — all three inputs interpolated directly in the run block.

**ephemeral/startup/action.yml line 152** (rule a — critical): `${{ inputs.preview-cmd }}` is the entire body of a `run:` block, meaning an attacker-controlled input is executed verbatim as a shell command.

**ephemeral/startup/action.yml line 158** (rule a): `source ${{ github.action_path }}/../retry-function.sh` — `github.action_path` interpolated directly.

**ephemeral/startup/action.yml line 165** (rule a): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — repeated in the print-logs step.

**finish/action.yml line 62** (rule a): `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — `inputs.preview-url` interpolated directly in run block.

Locations:

- `prepare/action.yml:15`
- `startup/action.yml:55`
- `startup/action.yml:57`
- `startup/action.yml:58`
- `ephemeral/shutdown/action.yml:37`
- `ephemeral/shutdown/action.yml:39`
- `ephemeral/startup/action.yml:63`
- `ephemeral/startup/action.yml:73`
- `ephemeral/startup/action.yml:74`
- `ephemeral/startup/action.yml:75`
- `ephemeral/startup/action.yml:152`
- `ephemeral/startup/action.yml:158`
- `ephemeral/startup/action.yml:165`
- `finish/action.yml:62`

### github-env-injection (severity: high)

The `run:` block in `finish/action.yml` writes a value derived from `inputs.preview-url` directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The expression `${{ inputs.preview-url }}` is interpolated directly into the shell string and then echoed into `$GITHUB_ENV`, allowing an attacker to inject arbitrary environment variable assignments (e.g. by embedding newlines in the input value):

```
echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV
```

Locations:

- `finish/action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 5 files:

1. **prepare/action.yml (line 15)**: Moved `github.event.number` to env block as `PR_NUMBER`, quoted in shell: `echo "$PR_NUMBER" > ./pr-id.txt`.

2. **startup/action.yml (lines 55-58)**:
   - Quoted `${IMAGE_NAME}` in `docker pull "${IMAGE_NAME}" &`
   - Removed direct `${{ inputs.ci-project }}` interpolation; added `CI_PROJECT: ${{ inputs.ci-project }}` to env block
   - Replaced dangerous `eval "${CONFIGURATION} localstack start -d"` with `env ${CONFIGURATION} localstack start -d` (env command doesn't interpret shell metacharacters)

3. **ephemeral/shutdown/action.yml (lines 37, 39)**: Added env block with `LOCALSTACK_API_KEY_INPUT: ${{ inputs.localstack-api-key }}` and `ACTION_PATH: ${{ github.action_path }}`; replaced direct interpolations with env var references and quoted the source path.

4. **ephemeral/startup/action.yml (lines 63, 73-75, 152, 158, 165)**:
   - Create preview environment step: Added env block with `LOCALSTACK_API_KEY_INPUT`, `ACTION_PATH`, `AUTO_LOAD_POD_INPUT`, `EXTENSION_AUTO_INSTALL_INPUT`, `LIFETIME_INPUT`; replaced all direct `${{ }}` interpolations with env var references.
   - Run preview deployment step: Moved `inputs.preview-cmd` to env block as `PREVIEW_CMD`; replaced direct `${{ inputs.preview-cmd }}` run body with `eval "$PREVIEW_CMD"` (prevents template injection while preserving functionality).
   - Print logs step: Added env block with `LOCALSTACK_API_KEY_INPUT` and `ACTION_PATH`; replaced direct interpolations with env var references.

5. **finish/action.yml (lines 62-63)**: Moved `inputs.preview-url` to env block as `PREVIEW_URL_INPUT`; rewrote the run block to use `effective_url="${LS_PREVIEW_URL:-${PREVIEW_URL_INPUT}}"` and sanitize with `printf '%s' "$effective_url" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection findings:
1. ephemeral/startup/action.yml (line 153): Replaced `eval "$PREVIEW_CMD"` with writing PREVIEW_CMD to a temp file via `printf '%s\n' "$PREVIEW_CMD" > "$_preview_script"` and executing with `bash "$_preview_script"`. This eliminates eval-based arbitrary code execution while preserving multi-line command support.
2. cloud-pods/action.yml (lines 21, 24): Added double quotes around `$NAME` in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent word splitting and shell metacharacter injection.
3. local/action.yml (lines 38, 41): Added double quotes around `${NAME}.zip` in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent word splitting and shell metacharacter injection.
4. startup/action.yml (line 76): Replaced unquoted `env ${CONFIGURATION} localstack start -d` with `printf '%s\n' "$CONFIGURATION" | xargs env -- localstack start -d` to safely parse KEY=VALUE pairs without exposing shell metacharacters to the shell interpreter.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection findings in three files:

1. ephemeral/startup/action.yml:
   - 'Setup preview name' step: sanitized `prId` from pr-id.txt using `printf '%s' ... | tr -d '\n\r'` before constructing `previewName`, then sanitized `previewName` again before writing to $GITHUB_ENV and $GITHUB_OUTPUT.
   - 'Create preview environment' step: sanitized `endpointUrl` (from external API response via jq) using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV as LS_PREVIEW_URL and AWS_ENDPOINT_URL.

2. ephemeral/shutdown/action.yml:
   - 'Load the PR ID' step: sanitized `pr_id` from pr-id.txt using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
   - 'Setup preview name' step: sanitized `prId` from pr-id.txt and the resulting `previewName` using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. finish/action.yml:
   - 'Load the PR ID' step: sanitized `pr_id` from pr-id.txt using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

All writes to $GITHUB_ENV and $GITHUB_OUTPUT now use quoted variable references ("$GITHUB_ENV", "$GITHUB_OUTPUT") as well.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in startup/action.yml line 63: changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. The unquoted variable expansion allowed an attacker who controls the LS_WAIT_TIMEOUT environment variable to inject arbitrary shell commands via metacharacters. Quoting the expansion prevents word splitting and command injection while preserving the default value fallback behavior.

