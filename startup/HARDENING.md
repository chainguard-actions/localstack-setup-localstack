<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation in a run: block. Line 77 contains `export CI_PROJECT=${{ inputs.ci-project }}` — the `inputs.ci-project` expression is interpolated directly into the shell command string before the shell ever sees it. An attacker who controls this input can inject arbitrary shell commands (e.g., by supplying a value like `; malicious-command`). This must be moved to an `env:` block and the variable must be double-quoted in the script.

Locations:

- `action.yml:77`

### script-injection (severity: high)

Rule (b): Unquoted shell variable expansion of untrusted data. Line 76 contains `docker pull ${IMAGE_NAME} &` where `IMAGE_NAME` is derived from `$IMAGE_TAG` (sourced from `inputs.image-tag` via the `env:` block). The variable is used without double-quotes, allowing shell metacharacters in the input to be interpreted by the shell. It should be `docker pull "${IMAGE_NAME}" &`.

Locations:

- `action.yml:76`

### script-injection (severity: high)

Rule (b) + eval: Line 78 contains `eval "${CONFIGURATION} localstack start -d"` where `CONFIGURATION` is sourced from `inputs.configuration` via the `env:` block. Although the variable is double-quoted in the shell, `eval` re-parses the resulting string as a shell command, so any shell metacharacters or command sequences embedded in `inputs.configuration` will be executed. This is a critical arbitrary command execution vulnerability. The `eval` pattern should be replaced with a safe alternative that does not re-parse user-controlled content as shell code.

Locations:

- `action.yml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all four findings in hardened/action/action.yml:
1. Replaced unsafe _actions glob search (ls -d ... | grep -i localstack) for GH_ACTION_ROOT with $GITHUB_ACTION_PATH-based computation: `realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/.."` (one level up since startup/action.yml is one directory deep from the repo root).
2. Quoted `${IMAGE_NAME}` in `docker pull` command (was unquoted, allowing shell metacharacter interpretation).
3. Moved `inputs.ci-project` out of the run block: added `CI_PROJECT_INPUT: ${{ inputs.ci-project }}` to the step's env block and replaced `export CI_PROJECT=${{ inputs.ci-project }}` with `export CI_PROJECT="$CI_PROJECT_INPUT"` in the script.
4. Replaced `eval "${CONFIGURATION} localstack start -d"` with safe xargs-based tokenization: the CONFIGURATION string is tokenized into a bash array using `xargs printf '%s\0'` and a null-delimited read loop, then passed to `env "${cfg_args[@]}" localstack start -d` — this passes KEY=VALUE pairs as environment variables without re-parsing user-controlled content as shell code.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell expansion in action.yml line 80: changed `localstack wait -t ${LS_WAIT_TIMEOUT:-30}` to `localstack wait -t "${LS_WAIT_TIMEOUT:-30}"`. This prevents shell injection via the workflow-controlled `LS_WAIT_TIMEOUT` environment variable.

