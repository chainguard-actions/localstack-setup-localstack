<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The run: block uses unquoted shell variable expansions of `${NAME}` (sourced from `inputs.name` via env:) in two localstack commands: `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`. Because `${NAME}` is not double-quoted, an attacker-controlled value in `inputs.name` containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) would be parsed by the shell, enabling command injection. The fix is to quote the expansions: `localstack state export "${NAME}.zip"` and `localstack state import "${NAME}.zip"`.

Locations:

- `action.yml:44`
- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in hardened/action/action.yml at lines 44 and 47. The unquoted `${NAME}.zip` expansions in `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip` were changed to `"${NAME}.zip"` (double-quoted). This prevents attacker-controlled values in `inputs.name` containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) from being interpreted by the shell. The `NAME` variable was already correctly sourced from `inputs.name` via the step's `env:` block — only the quoting was missing.

