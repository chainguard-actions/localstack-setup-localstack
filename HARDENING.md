<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.0** was hardened automatically. 7 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: block. `prepare/action.yml` interpolates `${{ github.event.number }}` directly into a shell command: `run: echo ${{ github.event.number }} > ./pr-id.txt`. An attacker-controlled PR number could inject shell metacharacters.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: block. `startup/action.yml` interpolates `${{ inputs.ci-project }}` directly into a shell command: `export CI_PROJECT=${{ inputs.ci-project }}`. A caller-controlled input value is injected into the shell before quoting, enabling command injection.

Locations:

- `startup/action.yml:65`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: block. `ephemeral/startup/action.yml` interpolates multiple inputs directly into shell commands: (1) `${{ inputs.preview-cmd }}` is used as a raw shell command (line ~148) — this is a direct arbitrary command execution vector; (2) `${{ inputs.localstack-api-key }}` is embedded in AUTH_HEADER assignment (line ~57); (3) `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` are interpolated into shell variable assignments (lines ~72-74); (4) `${{ github.action_path }}` is used in a `source` command (line ~60); (5) the same pattern repeats in the 'Print logs' step (lines ~155-160).

Locations:

- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:72`
- `ephemeral/startup/action.yml:73`
- `ephemeral/startup/action.yml:74`
- `ephemeral/startup/action.yml:148`
- `ephemeral/startup/action.yml:155`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: block. `ephemeral/shutdown/action.yml` interpolates `${{ inputs.localstack-api-key }}` directly into the AUTH_HEADER shell variable assignment and `${{ github.action_path }}` into a `source` command inside a run: block. Both allow a caller to inject shell content.

Locations:

- `ephemeral/shutdown/action.yml:34`
- `ephemeral/shutdown/action.yml:37`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: block. `finish/action.yml` interpolates `${{ inputs.preview-url }}` directly inside a shell conditional and echo command: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. A caller-controlled value is injected into the shell string before execution.

Locations:

- `finish/action.yml:65`
- `finish/action.yml:66`

### github-env-injection (severity: high)

`finish/action.yml` writes `inputs.preview-url` (caller-controlled) to `$GITHUB_ENV` without sanitization. The value is interpolated directly via `${{ inputs.preview-url }}` and written with `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection to set arbitrary environment variables.

Locations:

- `finish/action.yml:66`

### github-env-injection (severity: high)

`startup/action.yml` writes the `CONFIGURATION` env var (sourced from `inputs.configuration`) to the shell environment via `eval "${CONFIGURATION} localstack start -d"` and the `CONFIGURATION` variable is built up and used without sanitization. Additionally, `inputs.ci-project` is directly interpolated as `export CI_PROJECT=${{ inputs.ci-project }}` in the run block. The `CONFIGURATION` env var (set from `inputs.configuration`) is also used unsanitized in shell variable expansion throughout the script, and the GH_ACTION_ROOT value is written to `$GITHUB_ENV` via a command substitution that could be influenced by the runner environment.

Locations:

- `startup/action.yml:43`
- `startup/action.yml:65`
- `startup/action.yml:66`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env block (PR_NUMBER), used `"$PR_NUMBER"` in run.

2. startup/action.yml: (a) Replaced _actions/* glob for GH_ACTION_ROOT with `./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")` to fix the _actions glob issue and avoid env injection. (b) Moved `${{ inputs.ci-project }}` to env block as CI_PROJECT_INPUT.

3. ephemeral/startup/action.yml: (a) Create preview environment step: moved localstack-api-key, github.action_path, auto-load-pod, extension-auto-install, and lifetime to env block; updated all shell references. (b) Run preview deployment step: moved preview-cmd to env block, wrote to temp file and executed with `bash -eo pipefail` to preserve errexit. (c) Print logs step: moved localstack-api-key and github.action_path to env block.

4. ephemeral/shutdown/action.yml: Moved localstack-api-key and github.action_path to env block; updated shell references.

5. finish/action.yml: Moved `${{ inputs.preview-url }}` to env block as INPUT_PREVIEW_URL; rewrote conditional using shell variables; sanitized all values written to $GITHUB_ENV with `printf '%s' ... | tr -d '\n\r'` before writing (fixes both script-injection and github-env-injection findings).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 4 findings across 4 files:

1. startup/action.yml (script-injection): Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe loop that parses CONFIGURATION as KEY=VALUE pairs using `xargs -n1` and exports each assignment individually, then runs `localstack start -d` directly without eval.

2. ephemeral/shutdown/action.yml (github-env-injection): Sanitized both the pr-id.txt content written to $GITHUB_OUTPUT and the previewName written to $GITHUB_ENV using `printf '%s' ... | tr -d '\n\r'`.

3. ephemeral/startup/action.yml (github-env-injection): Sanitized both the previewName written to $GITHUB_ENV and $GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'`.

4. finish/action.yml (github-env-injection): Sanitized the pr-id.txt content written to $GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings by properly quoting unquoted shell variable expansions of user-controlled inputs:
1. cloud-pods/action.yml (lines 19, 22): Quoted `$NAME` → `"$NAME"` in `localstack pod save` and `localstack pod load --yes` commands.
2. local/action.yml (lines 34, 37): Quoted `${NAME}.zip` → `"${NAME}.zip"` in `localstack state export` and `localstack state import` commands.
3. startup/action.yml (line 57): Quoted `${IMAGE_NAME}` → `"${IMAGE_NAME}"` in the `docker pull` command.
All variables are already sourced from the step's `env:` block (not inline `${{ }}` expressions), so only the quoting fix was needed.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell expansion in hardened/action/startup/action.yml line 57: changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. The `LS_WAIT_TIMEOUT` variable is inherited from the calling workflow (not set in the step's env: block), making it untrusted. Quoting the expansion prevents shell metacharacters in the value from being interpreted as shell commands.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed action.yml line 91: replaced the _actions directory search (`ls -d ./../../_actions/* | grep -i localstack | tail -n1)/setup-localstack/*`) with `./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH")`. This builds the GH_ACTION_ROOT path from $GITHUB_ACTION_PATH instead of searching by the upstream action name, which works correctly under any fork/owner name. The path is made relative to $GITHUB_WORKSPACE with the required './' prefix for uses: references.

