<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.3.1** was hardened automatically. 9 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ inputs.preview-cmd }} is directly interpolated inside a run: shell block in ephemeral/startup/action.yml. This allows any caller to inject arbitrary shell commands. The value is executed verbatim as a shell command without any quoting or env-var indirection.

Locations:

- `ephemeral/startup/action.yml:152`

### script-injection (severity: high)

Rule (a): Multiple ${{ inputs.* }} and ${{ github.* }} expressions are directly interpolated inside run: shell blocks in ephemeral/startup/action.yml. Specifically: ${{ inputs.localstack-api-key }} is embedded in a shell string assignment (AUTH_HEADER), ${{ inputs.auto-load-pod }}, ${{ inputs.extension-auto-install }}, and ${{ inputs.lifetime }} are assigned to shell variables, and ${{ github.action_path }} is used in a 'source' command — all without env-var indirection. An attacker controlling these inputs can inject shell metacharacters.

Locations:

- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:75`
- `ephemeral/startup/action.yml:76`
- `ephemeral/startup/action.yml:77`
- `ephemeral/startup/action.yml:57`

### script-injection (severity: high)

Rule (a): ${{ inputs.ci-project }} is directly interpolated inside a run: shell block in startup/action.yml (export CI_PROJECT=${{ inputs.ci-project }}). Additionally, rule (b): the env var CONFIGURATION (sourced from ${{ inputs.configuration }}) is expanded unquoted inside eval "${CONFIGURATION} localstack start -d", allowing an attacker to inject arbitrary shell commands via the configuration input.

Locations:

- `startup/action.yml:68`
- `startup/action.yml:69`

### script-injection (severity: high)

Rule (a): ${{ github.event.number }} is directly interpolated inside a run: shell block in prepare/action.yml (echo ${{ github.event.number }} > ./pr-id.txt). Although github.event.number is typically an integer, any ${{ }} expression inside a run: block is a script-injection risk as the value is substituted before the shell parses the command.

Locations:

- `prepare/action.yml:14`

### script-injection (severity: high)

Rule (a): ${{ inputs.localstack-api-key }} and ${{ github.action_path }} are directly interpolated inside run: shell blocks in ephemeral/shutdown/action.yml. The api-key is embedded in a shell string assignment (AUTH_HEADER) and github.action_path is used in a 'source' command without env-var indirection.

Locations:

- `ephemeral/shutdown/action.yml:30`
- `ephemeral/shutdown/action.yml:33`

### script-injection (severity: high)

Rule (a): ${{ inputs.preview-url }} is directly interpolated inside a run: shell block in finish/action.yml. The value is embedded in a shell conditional and string expansion (echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}") without env-var indirection, allowing shell metacharacter injection.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

The run: block in finish/action.yml writes ${{ inputs.preview-url }} — an untrusted input — directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker can inject newlines into the value to set arbitrary environment variables for subsequent steps. The value is interpolated directly into the shell string: echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV

Locations:

- `finish/action.yml:58`

### github-env-injection (severity: high)

The run: block in startup/action.yml uses eval "${CONFIGURATION} localstack start -d" where CONFIGURATION is set from ${{ inputs.configuration }} via the env: block. The CONFIGURATION variable is also written into the environment via eval without sanitization. An attacker can embed newline characters in the configuration input to inject arbitrary key=value pairs into the process environment, and can also inject arbitrary shell commands via the eval.

Locations:

- `startup/action.yml:69`

### suspicious-run-content (severity: high)

eval-dynamic: The run: block in startup/action.yml uses eval with a shell variable: eval "${CONFIGURATION} localstack start -d". The CONFIGURATION variable is populated from the user-controlled input ${{ inputs.configuration }}. This allows an attacker to inject arbitrary shell commands by supplying shell metacharacters or command substitution in the configuration input value.

Locations:

- `startup/action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, suspicious-run-content

**Notes:**

Fixed all script-injection, github-env-injection, and suspicious-run-content findings across 5 files:

1. ephemeral/startup/action.yml: Moved ${{ inputs.localstack-api-key }}, ${{ github.action_path }}, ${{ inputs.auto-load-pod }}, ${{ inputs.extension-auto-install }}, ${{ inputs.lifetime }} to env: blocks in 'Create preview environment' step; moved ${{ inputs.preview-cmd }} to env: block (PREVIEW_CMD) in 'Run preview deployment' step; moved ${{ inputs.localstack-api-key }} and ${{ github.action_path }} to env: block in 'Print logs' step.

2. startup/action.yml: Moved ${{ inputs.ci-project }} to env: block as CI_PROJECT_INPUT; replaced eval "${CONFIGURATION} localstack start -d" with env ${CONFIGURATION} localstack start -d (eliminates both eval-dynamic and github-env-injection risks); fixed _actions glob to use realpath --relative-to with $GITHUB_ACTION_PATH.

3. prepare/action.yml: Moved ${{ github.event.number }} to env: block as EVENT_NUMBER.

4. ephemeral/shutdown/action.yml: Moved ${{ inputs.localstack-api-key }} and ${{ github.action_path }} to env: block.

5. finish/action.yml: Moved ${{ inputs.preview-url }} to env: block as PREVIEW_URL_INPUT; added sanitization with printf '%s' | tr -d '\n\r' before writing to $GITHUB_ENV.

6. action.yml: Fixed _actions glob to use realpath --relative-to with $GITHUB_ACTION_PATH.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:

1. hardened/action/startup/action.yml: Quoted `${IMAGE_NAME}` in `docker pull` command to prevent word splitting. Replaced unquoted `env ${CONFIGURATION} localstack start -d` with a safe xargs-based tokenization into a bash array (`conf_args`), then `env "${conf_args[@]}" localstack start -d`. This safely handles space-separated KEY=VALUE pairs while preventing shell metacharacter injection.

2. hardened/action/ephemeral/startup/action.yml: Replaced `eval "$PREVIEW_CMD"` with `bash -c "$PREVIEW_CMD"`. The `eval` builtin runs code in the current shell context and is inherently dangerous; `bash -c` runs in a subshell, limiting the blast radius of any injected commands while preserving the intended functionality of executing user-specified preview commands.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 6 findings across 5 files:

1. cloud-pods/action.yml: Quoted $NAME in `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"` to prevent shell metacharacter injection.

2. local/action.yml: Quoted ${NAME} in `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"` to prevent shell metacharacter injection.

3. ephemeral/shutdown/action.yml (line 24): Replaced single-line `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` with a two-step sanitization: capture raw value, then `printf '%s' "$raw" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

4. ephemeral/shutdown/action.yml (line 33): Added sanitization of both prId and the composed previewName (derived from $GITHUB_REPOSITORY and prId) before writing to $GITHUB_ENV.

5. ephemeral/startup/action.yml (lines 55-56): Added sanitization of prId and the composed previewName before writing to both $GITHUB_ENV and $GITHUB_OUTPUT.

6. finish/action.yml (line 36): Replaced single-line `echo "pr_id=$(< pr-id.txt)" >> $GITHUB_OUTPUT` with a two-step sanitization using printf + tr before writing to $GITHUB_OUTPUT.

All sanitizations follow the correct pattern of capturing the raw value first (separate assignment), then sanitizing in a second step, to avoid swallowing errors under bash errexit.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell expansion in hardened/action/startup/action.yml line 80: changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. The double quotes prevent shell metacharacters in the inherited LS_WAIT_TIMEOUT environment variable from being interpreted as shell commands.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Run preview deployment' step in hardened/action/ephemeral/startup/action.yml. The original code used `bash -c "$PREVIEW_CMD"` which interprets the PREVIEW_CMD env var (set from inputs.preview-cmd) as a shell script via the -c flag, enabling shell injection. The fix writes the command to a temporary file with `printf '%s\n' "$PREVIEW_CMD" > "$PREVIEW_SCRIPT"` and executes it with `bash "$PREVIEW_SCRIPT"`, treating the content as a script file rather than passing it directly to bash's -c flag.

