<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.ci-project }}` is directly interpolated inside a `run:` shell command: `export CI_PROJECT=${{ inputs.ci-project }}`. Before the shell ever sees this line, GitHub Actions substitutes the raw value of `inputs.ci-project` into the script text. An attacker can supply a value containing shell metacharacters (e.g. `; malicious-command`) to execute arbitrary code on the runner.

Locations:

- `action.yml:75`

### script-injection (severity: high)

Sub-rule (b): `eval "${CONFIGURATION} localstack start -d"` passes the `CONFIGURATION` env var — which is set directly from `${{ inputs.configuration }}` — to `eval`. Even though the value is routed through an env var, `eval` re-parses and executes the expanded string as shell code. An attacker can supply shell metacharacters or subshell expressions in `inputs.configuration` to achieve arbitrary command execution. The env var must be double-quoted AND `eval` must not be used with attacker-controlled data.

Locations:

- `action.yml:76`

### script-injection (severity: high)

Sub-rule (b): `docker pull ${IMAGE_NAME} &` uses an unquoted shell expansion of `IMAGE_NAME`. `IMAGE_NAME` is derived from `IMAGE_TAG`, which is set from `${{ inputs.image-tag }}` via the `env:` block. An unquoted expansion allows the shell to parse metacharacters (spaces, semicolons, pipes, glob characters, etc.) out of the attacker-controlled value, enabling command injection. The fix is to quote the expansion: `docker pull "${IMAGE_NAME}" &`.

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:
1. Replaced the _actions/* glob search for GH_ACTION_ROOT with a GITHUB_ACTION_PATH-based path: `$(cd "$GITHUB_ACTION_PATH/.." && pwd)` (startup/action.yml is one level deep, so parent is the repo root).
2. Removed inline `${{ inputs.ci-project }}` from the run: block; moved it to the env: block as `CI_PROJECT: ${{ inputs.ci-project }}`.
3. Replaced `eval "${CONFIGURATION} localstack start -d"` with `env ${CONFIGURATION} localstack start -d` — the `env` command treats KEY=VALUE tokens as environment variable assignments, not shell code, preventing command injection via the configuration input.
4. Quoted `${IMAGE_NAME}` in `docker pull "${IMAGE_NAME}" &` to prevent word splitting and glob expansion from an attacker-controlled image tag.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in action.yml's 'Start LocalStack' step:
1. CONFIGURATION variable (line 67): Replaced unquoted `env ${CONFIGURATION} localstack start -d` with an xargs-based tokenization into a bash array (`cfg_args`), then used `env "${cfg_args[@]}" localstack start -d`. This safely handles space-separated KEY=VALUE pairs while preventing shell metacharacter injection.
2. LS_WAIT_TIMEOUT variable (line 70): Added double-quotes around `${LS_WAIT_TIMEOUT:-30}` to prevent command injection via the unquoted variable expansion.

