<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The `run:` block expands the env var `NAME` (sourced from `inputs.name`, a workflow-controllable value) without double-quoting in the `localstack` commands: `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`. An unquoted `${NAME}` allows shell metacharacter injection (e.g. semicolons, pipes, glob characters) if a caller supplies a crafted value for `inputs.name`. The fix is to quote the expansion: `"${NAME}.zip"`.

Locations:

- `action.yml:44`

### script-injection (severity: high)

Rule (b) violation: The `run:` block expands the env var `NAME` (sourced from `inputs.name`, a workflow-controllable value) without double-quoting in the `localstack` commands: `localstack state import ${NAME}.zip`. An unquoted `${NAME}` allows shell metacharacter injection if a caller supplies a crafted value for `inputs.name`. The fix is to quote the expansion: `"${NAME}.zip"`.

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/action.yml. Both occurrences of unquoted `${NAME}.zip` in the `localstack state export` and `localstack state import` commands were changed to `"${NAME}.zip"` (double-quoted). This prevents shell metacharacter injection when `inputs.name` contains special characters like semicolons, pipes, or glob patterns.

