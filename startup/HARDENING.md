<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of `${{ inputs.ci-project }}` inside a `run:` shell command on line 76: `export CI_PROJECT=${{ inputs.ci-project }}`. The Actions runner substitutes this value before the shell sees it, allowing an attacker to inject arbitrary shell metacharacters via the `ci-project` input.

Locations:

- `action.yml:76`

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansion of `${IMAGE_NAME}` on line 75: `docker pull ${IMAGE_NAME} &`. `IMAGE_NAME` is derived from the `IMAGE_TAG` env var which is set to `${{ inputs.image-tag }}` (attacker-controlled). The unquoted expansion allows word splitting and glob expansion of the attacker-supplied value.

Locations:

- `action.yml:75`

### script-injection (severity: high)

Sub-rule (b): `eval "${CONFIGURATION} localstack start -d"` on line 77. `CONFIGURATION` is set from `${{ inputs.configuration }}` (attacker-controlled). Even though the variable is double-quoted in the eval argument, `eval` re-parses the resulting string as shell code, so any shell metacharacters or commands embedded in `inputs.configuration` will be executed — this is command injection via eval.

Locations:

- `action.yml:77`

### suspicious-run-content (severity: high)

eval-dynamic: Line 77 uses `eval "${CONFIGURATION} localstack start -d"` where `${CONFIGURATION}` is a `$`-prefixed shell variable holding user-controlled data from `inputs.configuration`. This matches the eval-dynamic pattern (`eval\s+[\x60$]`) and allows dynamic execution of attacker-supplied shell commands.

Locations:

- `action.yml:77`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, suspicious-run-content, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. Removed direct `${{ inputs.ci-project }}` interpolation from run block; moved to env: block as `CI_PROJECT: ${{ inputs.ci-project }}`.
2. Quoted `${IMAGE_NAME}` in `docker pull` to prevent word splitting/glob expansion.
3. Replaced `eval "${CONFIGURATION} localstack start -d"` with safe xargs tokenization into a bash array followed by `env "${cfg_args[@]}" localstack start -d`, eliminating the eval-based command injection.
4. Also fixed the `_actions` glob path search to use `$GITHUB_ACTION_PATH` with `realpath --relative-to` for correctness under the hardened action name.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in hardened/action/action.yml line 62. Changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"` to prevent shell metacharacter injection via the LS_WAIT_TIMEOUT environment variable.

