<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b) violation: The `run:` block expands the env var `${NAME}` (sourced from `inputs.name`, a workflow-controllable value) **unquoted** in two shell commands:
- `localstack state export ${NAME}.zip`
- `localstack state import ${NAME}.zip`

An unquoted `${NAME}` allows shell metacharacters (`;`, `|`, `&`, whitespace, glob chars, etc.) embedded in the input to be interpreted by the shell, enabling command injection. The fix is to double-quote the expansion: `"${NAME}.zip"`.

Locations:

- `action.yml:44`
- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted `${NAME}` expansions on lines 44 and 47 of action.yml. Changed `localstack state export ${NAME}.zip` to `localstack state export "${NAME}.zip"` and `localstack state import ${NAME}.zip` to `localstack state import "${NAME}.zip"`. Double-quoting prevents shell metacharacters in the `inputs.name` value from being interpreted by the shell, eliminating the command injection vulnerability.

