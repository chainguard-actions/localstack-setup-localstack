<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ inputs.ci-project }}` is directly interpolated inside a `run:` shell command string at line 71: `export CI_PROJECT=${{ inputs.ci-project }}`. An attacker can supply a value containing shell metacharacters (e.g. `; malicious-command`) via the `ci-project` input to achieve arbitrary command execution.

Locations:

- `action.yml:71`

### script-injection (severity: high)

Sub-rule (b): The shell variable `${IMAGE_NAME}` (derived from `${IMAGE_TAG}`, which is sourced from `inputs.image-tag` via the env block) is expanded unquoted in `docker pull ${IMAGE_NAME} &` at line 70. An attacker-controlled image tag containing shell metacharacters can break out of the intended command.

Locations:

- `action.yml:70`

### script-injection (severity: high)

Sub-rule (b): The shell variable `${CONFIGURATION}` (sourced from `inputs.configuration` via the env block) is passed unquoted into `eval "${CONFIGURATION} localstack start -d"` at line 72. Even though it is inside double-quotes, `eval` re-parses the resulting string, so any shell metacharacters or commands embedded in `inputs.configuration` are executed. This is a critical command-injection vector.

Locations:

- `action.yml:72`

### github-env-injection (severity: high)

The `CONFIGURATION` env var (set from `${{ inputs.configuration }}`) is passed into `eval "${CONFIGURATION} localstack start -d"` without sanitization. An attacker can embed newline characters in `inputs.configuration` to inject arbitrary `KEY=VALUE` pairs into `$GITHUB_ENV`, `$GITHUB_PATH`, or `$GITHUB_OUTPUT` via the eval'd subcommands, bypassing environment file protections. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before use.

Locations:

- `action.yml:72`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all five findings in action.yml:
1. Moved `inputs.ci-project` from inline `${{ inputs.ci-project }}` in run: block to env: block as `CI_PROJECT_INPUT`, referenced safely as `"$CI_PROJECT_INPUT"` in shell.
2. Quoted `${IMAGE_NAME}` → `"${IMAGE_NAME}"` in `docker pull` command.
3. Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe alternative: tokenize CONFIGURATION using xargs into a bash array, then pass via `env "${env_args[@]}" localstack start -d` — eliminates eval entirely and prevents shell metacharacter injection and newline-based github-env-injection.
4. Also fixed the _actions/* glob (which would fail under the hardened repo name) to use `realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/.."` and added `./` prefix to the dynamic-uses `uses:` path.

