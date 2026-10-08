<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b) violation: The `run:` block maps `inputs.name` and `inputs.action` into env vars `NAME` and `ACTION`, but then expands them unquoted in shell commands. Specifically, `${NAME}.zip` is unquoted in `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`, and `$NAME` is unquoted in `echo "Saving State $NAME"` / `echo "Loading State $NAME"`. An attacker-controlled input containing shell metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace) could break out of the intended command. All expansions of workflow-controllable env vars must be double-quoted (e.g., `"${NAME}.zip"`, `"$NAME"`).

Locations:

- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in the run block. Changed `localstack state export ${NAME}.zip` to `localstack state export "${NAME}.zip"` and `localstack state import ${NAME}.zip` to `localstack state import "${NAME}.zip"`. The $NAME and $ACTION variables in echo statements were already inside double-quoted strings and were properly quoted. The env vars NAME and ACTION are correctly set from inputs.name and inputs.action in the env block, following the safe pattern of moving expressions out of the shell script body.

