<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.2** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands.

1. prepare/action.yml line 16: `run: echo ${{ github.event.number }} > ./pr-id.txt` — github.event.number is interpolated directly into the shell command. An attacker controlling a PR number (e.g. via a crafted event) could inject shell metacharacters.

2. startup/action.yml line ~75: `export CI_PROJECT=${{ inputs.ci-project }}` — the inputs.ci-project expression is interpolated directly into the shell, allowing shell injection via the input value.

3. ephemeral/startup/action.yml (Create preview environment step): Multiple direct interpolations inside the run: block:
   - `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` — inputs.localstack-api-key injected directly.
   - `source ${{ github.action_path }}/../retry-function.sh` — github.action_path injected directly.
   - `autoLoadPod="${AUTO_LOAD_POD:-${{ inputs.auto-load-pod }}}"` — inputs.auto-load-pod injected directly.
   - `extensionAutoInstall="${EXTENSION_AUTO_INSTALL:-${{ inputs.extension-auto-install }}}"` — inputs.extension-auto-install injected directly.
   - `lifetime="${{ inputs.lifetime }}"` — inputs.lifetime injected directly.

4. ephemeral/startup/action.yml (Run preview deployment step, line ~148): `${{ inputs.preview-cmd }}` is the entire body of the run: block — the raw input value is executed as a shell command, giving full arbitrary code execution to whoever controls the input.

5. ephemeral/startup/action.yml (Print logs step): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh` — same pattern as above.

6. ephemeral/shutdown/action.yml (Shutdown ephemeral instance step): `AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"` and `source ${{ github.action_path }}/../retry-function.sh` — direct interpolation in run: block.

7. finish/action.yml (Load the Ephemeral Instance URL step): `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — inputs.preview-url interpolated directly into the shell command and into GITHUB_ENV.

Locations:

- `prepare/action.yml:16`
- `startup/action.yml:75`
- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:80`
- `ephemeral/startup/action.yml:81`
- `ephemeral/startup/action.yml:82`
- `ephemeral/startup/action.yml:148`
- `ephemeral/startup/action.yml:157`
- `ephemeral/shutdown/action.yml:36`
- `finish/action.yml:57`

### github-env-injection (severity: high)

Untrusted input values are written to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step).

1. finish/action.yml (Load the Ephemeral Instance URL step): `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV` — the inputs.preview-url expression is interpolated directly into the value written to GITHUB_ENV. A newline in the value would allow injecting arbitrary environment variables.

2. ephemeral/startup/action.yml (Setup preview name step): `echo "previewName=$previewName" >> $GITHUB_ENV` — previewName is derived from $GITHUB_REPOSITORY (an inherited env var) and the contents of pr-id.txt. While GITHUB_REPOSITORY is generally safe, it is a workflow-controlled value and is not sanitized before being written to GITHUB_ENV.

3. startup/action.yml (Start LocalStack step): The CONFIGURATION env var (sourced from inputs.configuration via env:) is used in `eval "${CONFIGURATION} localstack start -d"` — while this is not a direct GITHUB_ENV write, the unsanitized CONFIGURATION value is passed to eval, which is equivalent in severity.

Locations:

- `finish/action.yml:57`
- `ephemeral/startup/action.yml:52`
- `startup/action.yml:74`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. prepare/action.yml: Moved github.event.number to env block (PR_NUMBER).
2. startup/action.yml: Removed direct `export CI_PROJECT=${{ inputs.ci-project }}` injection; added CI_PROJECT to env block and referenced it as ${CI_PROJECT} in the eval command.
3. ephemeral/startup/action.yml (Setup preview name): Added sanitization with printf/tr before writing previewName to GITHUB_ENV and GITHUB_OUTPUT.
4. ephemeral/startup/action.yml (Create preview environment): Moved inputs.localstack-api-key, inputs.auto-load-pod, inputs.extension-auto-install, inputs.lifetime, and github.action_path to env block; replaced all inline ${{ }} with env var references.
5. ephemeral/startup/action.yml (Run preview deployment): Moved inputs.preview-cmd to env block (PREVIEW_CMD); wrote to temp file and executed with `bash -eo pipefail` to preserve errexit semantics.
6. ephemeral/startup/action.yml (Print logs): Moved inputs.localstack-api-key and github.action_path to env block.
7. ephemeral/shutdown/action.yml (Shutdown ephemeral instance): Moved inputs.localstack-api-key and github.action_path to env block.
8. finish/action.yml (Load Ephemeral Instance URL): Moved inputs.preview-url to env block; sanitized all values written to GITHUB_ENV with printf/tr -d '\n\r'.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed four security findings:

1. startup/action.yml (script-injection): Replaced dangerous `eval "${CONFIGURATION} CI_PROJECT=${CI_PROJECT} localstack start -d"` with safe xargs-based tokenization into a bash array followed by `env "${config_args[@]}" CI_PROJECT="$CI_PROJECT" localstack start -d`. Also added double-quotes around `${IMAGE_NAME}` in the `docker pull` command to prevent shell metacharacter injection.

2. local/action.yml (script-injection): Added double-quotes around `${NAME}.zip` in both `localstack state export` and `localstack state import` commands to prevent shell metacharacter injection from user-controlled input.

3. cloud-pods/action.yml (script-injection): Added double-quotes around `$NAME` in both `localstack pod save` and `localstack pod load --yes` commands to prevent shell metacharacter injection from user-controlled input.

4. ephemeral/shutdown/action.yml (github-env-injection): Added sanitization of `previewName` using `printf '%s' "$previewName" | tr -d '\n\r'` before writing to $GITHUB_ENV, matching the pattern already correctly used in ephemeral/startup/action.yml. This prevents newline injection attacks via attacker-controlled repository names.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Load the PR ID' step in both ephemeral/shutdown/action.yml and finish/action.yml. The unsanitized pattern `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` was replaced with a multi-line script that: (1) reads the raw value into $raw, (2) sanitizes it with `printf '%s' "$raw" | tr -d '\n\r'` into $safe, and (3) writes `pr_id=$safe` to $GITHUB_OUTPUT. The two-step sanitization (raw then safe) avoids swallowing exit codes under bash's errexit mode.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed both _actions directory glob searches by replacing them with $GITHUB_ACTION_PATH-based paths:

1. action.yml (line 91): Replaced the ls/grep/tail pipeline with `echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH")" >> $GITHUB_ENV`. Since action.yml is at the repo root, $GITHUB_ACTION_PATH points directly to the root.

2. startup/action.yml (line 44): Replaced the ls/grep/tail pipeline with `echo "GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")" >> $GITHUB_ENV`. Since startup/action.yml is one directory below the root, $GITHUB_ACTION_PATH/.. resolves to the repo root.

Both values are made relative to $GITHUB_WORKSPACE with the required "./" prefix, as they feed into jenseng/dynamic-uses `uses:` fields which require relative paths starting with "./".

