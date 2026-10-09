<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): The `run:` block maps `inputs.name` (attacker-controlled) into the env var `NAME`, then uses it **unquoted** as `${NAME}.zip` in two shell commands: `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`. An unquoted shell variable expansion allows an attacker to inject shell metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace) via the `name` input, enabling command injection. The variable should be double-quoted: `"${NAME}.zip"`.

Locations:

- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in action.yml at line 46. Changed `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip` to use double-quoted expansions: `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"`. This prevents shell metacharacters in the `name` input from being interpreted as shell commands.

