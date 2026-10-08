<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): The `run:` block expands env vars `$NAME` and `$ACTION` (sourced from `inputs.name` and `inputs.action`) without double-quoting in several shell commands. Specifically, `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip` use unquoted `${NAME}`, and `echo "Invalid action: $ACTION"` is the only quoted use — but `echo "Saving State $NAME"` and `echo "Loading State $NAME"` are inside double-quotes while the `localstack` command arguments are not. An attacker-controlled `inputs.name` containing shell metacharacters (e.g. spaces, semicolons, glob characters) could cause command injection. All expansions of workflow-controllable env vars must be double-quoted: e.g. `localstack state export "${NAME}.zip"`.

Locations:

- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted ${NAME} variable expansions in the run: block of action.yml. Changed `localstack state export ${NAME}.zip` to `localstack state export "${NAME}.zip"` and `localstack state import ${NAME}.zip` to `localstack state import "${NAME}.zip"`. The NAME and ACTION variables were already correctly sourced via the step's env: block rather than inline ${{ }} expressions, so only the missing double-quotes needed to be added to prevent word-splitting and glob expansion on attacker-controlled input.

