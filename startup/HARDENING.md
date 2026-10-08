<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Start LocalStack' run block in action.yml contains multiple script injection vulnerabilities:

(a) Sub-rule (a) — Direct expression interpolation: Line 80 interpolates `${{ inputs.ci-project }}` directly inside a shell `run:` command: `export CI_PROJECT=${{ inputs.ci-project }}`. An attacker controlling this input can inject arbitrary shell commands.

(b) Sub-rule (b) — Unquoted shell variable expansion: Line 79 uses `docker pull ${IMAGE_NAME} &` where `IMAGE_NAME` is derived from the `IMAGE_TAG` env var (set from `${{ inputs.image-tag }}`). The variable is unquoted, allowing shell metacharacter injection.

(b) Sub-rule (b) — Eval with env var from untrusted input: Line 81 uses `eval "${CONFIGURATION} localstack start -d"` where `CONFIGURATION` is set from `${{ inputs.configuration }}` via the env block. Even though the variable is double-quoted in the eval string, `eval` re-parses the expanded string, so any shell metacharacters in the input are executed as commands.

Locations:

- `action.yml:79`
- `action.yml:80`
- `action.yml:81`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection vulnerabilities in action.yml:
1. Removed inline `${{ inputs.ci-project }}` expression from the run block (line 80) and moved it to the step's env: block as `CI_PROJECT: ${{ inputs.ci-project }}`.
2. Quoted `${IMAGE_NAME}` in the docker pull command to prevent metacharacter injection.
3. Replaced `eval "${CONFIGURATION} localstack start -d"` with `env ${CONFIGURATION} localstack start -d` to eliminate shell re-parsing of the CONFIGURATION variable.
4. Also fixed the _actions glob search by replacing it with `$(cd "$GITHUB_ACTION_PATH/.." && pwd)` to correctly resolve the repo root without depending on the action's name in the runner's _actions directory.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script-injection vulnerability in the 'Start LocalStack' step of action.yml. The unquoted `env ${CONFIGURATION} localstack start -d` command was replaced with a safe xargs-based tokenization approach that parses KEY=VALUE pairs from the CONFIGURATION input without evaluating shell metacharacters. DNS_ADDRESS=127.0.0.1 and IMAGE_NAME are now exported directly as literal values rather than being prepended to the CONFIGURATION string. The localstack start -d command now runs without the env prefix, relying on the exported environment variables instead.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in action.yml at line 76 by quoting the `${LS_WAIT_TIMEOUT:-30}` shell variable expansion in the `localstack wait -t` command. Changed from `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. This prevents an attacker-controlled calling workflow from injecting shell metacharacters through the LS_WAIT_TIMEOUT environment variable.

