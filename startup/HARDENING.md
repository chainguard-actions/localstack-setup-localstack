<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ inputs.ci-project }}` is directly interpolated inside a `run:` shell block (line 69: `export CI_PROJECT=${{ inputs.ci-project }}`). The Actions runner substitutes this value into the shell script before execution, allowing an attacker to inject arbitrary shell commands via the `ci-project` input. The value must be passed through an `env:` variable and properly quoted instead.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Sub-rule (b): `docker pull ${IMAGE_NAME} &` (line 68) uses the shell variable `${IMAGE_NAME}` without double-quoting. `IMAGE_NAME` is derived from the `IMAGE_TAG` env var which is set from `inputs.image-tag` (an attacker-controlled input). An unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) embedded in the input to be interpreted by the shell. It must be quoted as `"${IMAGE_NAME}"`.

Locations:

- `action.yml:68`

### script-injection (severity: high)

Sub-rule (b) + eval: `eval "${CONFIGURATION} localstack start -d"` (line 70) passes the `CONFIGURATION` env var — sourced directly from `inputs.configuration` (untrusted, line 80: `CONFIGURATION: ${{ inputs.configuration }}`) — to `eval`. Even though the variable is double-quoted in the eval argument, `eval` re-parses the resulting string as a full shell command, so any shell metacharacters or commands embedded in `inputs.configuration` will be executed as arbitrary code. This is a critical remote code execution vulnerability.

Locations:

- `action.yml:70`
- `action.yml:80`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all four security findings in hardened/action/action.yml:
1. Moved CI_PROJECT from inline `${{ inputs.ci-project }}` in run block to the env: block.
2. Double-quoted `${IMAGE_NAME}` in `docker pull` to prevent word-splitting/glob injection.
3. Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe `env` invocation: tokenized CONFIGURATION using xargs+while-read-NUL into an array of NAME=VALUE pairs, then called `env "${env_args[@]}" localstack start -d` — eliminating the eval-based RCE vector.
4. Also fixed GH_ACTION_ROOT to use `$GITHUB_ACTION_PATH` (via realpath --relative-to) instead of globbing `_actions/*`, which would fail under the hardened action's different owner/repo directory name.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell parameter expansion in action.yml line 62: changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. This prevents command injection via the inherited `LS_WAIT_TIMEOUT` environment variable, which could be set by a calling workflow to contain shell metacharacters.

