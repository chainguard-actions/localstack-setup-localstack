<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.2** was hardened automatically. 8 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ github.event.number }}` is interpolated directly inside a `run:` shell command. An attacker who controls the event payload can inject arbitrary shell commands. Offending line: `run: echo ${{ github.event.number }} > ./pr-id.txt`

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

Rule (a): `${{ inputs.ci-project }}` is interpolated directly inside a `run:` shell command (`export CI_PROJECT=${{ inputs.ci-project }}`). A caller can supply a value containing shell metacharacters to inject arbitrary commands. Additionally, `eval "${CONFIGURATION} localstack start -d"` executes the `CONFIGURATION` env var (sourced from `inputs.configuration`) as a shell command prefix, allowing arbitrary command injection via rule (b) — the env var is eval'd without sanitization.

Locations:

- `startup/action.yml:57`
- `startup/action.yml:58`

### script-injection (severity: high)

Rule (a): `${{ inputs.preview-cmd }}` is interpolated directly as the entire body of a `run:` shell step (`run: |\n  ${{ inputs.preview-cmd }}`). This allows a caller to execute arbitrary shell commands by supplying a malicious value for `preview-cmd`.

Locations:

- `ephemeral/startup/action.yml:130`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` expressions are interpolated directly inside `run:` shell commands in the 'Create preview environment' step: (1) `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"`, (2) `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"`, (3) `lifetime="${{ inputs.lifetime }}"`. A caller can inject shell metacharacters through any of these inputs.

Locations:

- `ephemeral/startup/action.yml:72`
- `ephemeral/startup/action.yml:73`
- `ephemeral/startup/action.yml:74`

### script-injection (severity: high)

Rule (a): `${{ inputs.localstack-api-key }}` is interpolated directly inside a `run:` shell command in the 'Create preview environment' step: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`. A caller can inject shell metacharacters through the `localstack-api-key` input. The same pattern also appears in the 'Print logs of ephemeral instance' step.

Locations:

- `ephemeral/startup/action.yml:51`
- `ephemeral/startup/action.yml:148`

### script-injection (severity: high)

Rule (a): `${{ inputs.localstack-api-key }}` is interpolated directly inside a `run:` shell command in the 'Shutdown ephemeral instance' step: `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`. A caller can inject shell metacharacters through the `localstack-api-key` input.

Locations:

- `ephemeral/shutdown/action.yml:33`

### script-injection (severity: high)

Rule (a): `${{ inputs.preview-url }}` is interpolated directly inside a `run:` shell command in the 'Load the Ephemeral Instance URL' step: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. A caller can inject shell metacharacters or newlines through the `preview-url` input.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes `inputs.preview-url` directly to `$GITHUB_ENV` without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. A caller can supply a newline-containing value to inject arbitrary environment variables into subsequent steps (e.g., `EVIL_VAR=injected\nLS_PREVIEW_URL=...`). The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `finish/action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 8 security findings across 5 files:

1. prepare/action.yml (line 16): Moved `${{ github.event.number }}` to env var `PR_NUMBER`.

2. startup/action.yml (lines 57-58): Moved `${{ inputs.ci-project }}` to env var `INPUT_CI_PROJECT`, referenced as `${INPUT_CI_PROJECT}` in the shell script.

3. ephemeral/startup/action.yml (lines 51, 72-74): In 'Create preview environment' step, moved `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` to env vars (`INPUT_LOCALSTACK_API_KEY`, `INPUT_AUTO_LOAD_POD`, `INPUT_EXTENSION_AUTO_INSTALL`, `INPUT_LIFETIME`).

4. ephemeral/startup/action.yml (line 130): In 'Run preview deployment' step, moved `${{ inputs.preview-cmd }}` to env var `PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` to execute it.

5. ephemeral/startup/action.yml (line 148): In 'Print logs' step, moved `${{ inputs.localstack-api-key }}` to env var `INPUT_LOCALSTACK_API_KEY`.

6. ephemeral/shutdown/action.yml (line 33): Moved `${{ inputs.localstack-api-key }}` to env var `INPUT_LOCALSTACK_API_KEY`.

7. finish/action.yml (lines 57-58): Moved `${{ inputs.preview-url }}` to env var `INPUT_PREVIEW_URL`, resolved URL via shell expansion, and sanitized with `tr -d '\n\r'` before writing to `$GITHUB_ENV` (fixes both script-injection and github-env-injection findings).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all findings across three files:

1. ephemeral/startup/action.yml:
   - Moved `github.action_path` into env: blocks as ACTION_PATH in two steps (Create preview environment, Print logs), replacing direct `${{ github.action_path }}` interpolation in run: blocks.
   - Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"` in the Run preview deployment step.
   - Sanitized previewName with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_ENV.
   - Sanitized endpointUrl with `printf '%s' ... | tr -d '\n\r'` before writing LS_PREVIEW_URL and AWS_ENDPOINT_URL to GITHUB_ENV.

2. ephemeral/shutdown/action.yml:
   - Moved `github.action_path` into env: block as ACTION_PATH in the Shutdown ephemeral instance step.
   - Sanitized previewName with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_ENV.

3. startup/action.yml:
   - Replaced `eval "${CONFIGURATION} localstack start -d"` with safe KEY=VALUE parsing using xargs+while loop to export configuration as environment variables, then running `localstack start -d` directly.
   - Quoted `docker pull "${IMAGE_NAME}"` properly.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 4 findings across 5 files:
1. ephemeral/startup/action.yml: Replaced `bash -c "$PREVIEW_CMD"` with writing PREVIEW_CMD to a temp script file and executing it with `bash "$PREVIEW_SCRIPT"`, eliminating the bash -c injection vector.
2. cloud-pods/action.yml: Added double quotes around $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent word splitting/glob injection.
3. local/action.yml: Added double quotes around ${NAME}.zip in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent word splitting/glob injection.
4. ephemeral/shutdown/action.yml: Sanitized pr-id.txt content with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
5. finish/action.yml: Same sanitization fix for the Load the PR ID step.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted shell expansion in `startup/action.yml` line 73: changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. This prevents shell metacharacters in the inherited `LS_WAIT_TIMEOUT` environment variable from being interpreted by the shell while preserving the default value of 30 when the variable is unset.

