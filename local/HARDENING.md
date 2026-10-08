<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b) violation: The `run:` block expands `${NAME}` without double-quoting in two shell commands (`localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`). The env var `NAME` is sourced directly from `${{ inputs.name }}` (an untrusted caller-controlled input). An unquoted expansion allows the shell to parse metacharacters (spaces, globs, semicolons, etc.) out of the value, enabling command injection. Fix: quote the expansion as `"${NAME}.zip"`.

Locations:

- `action.yml:40`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in action.yml at lines 40 and 43. Changed `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip` to use double-quoted expansions: `"${NAME}.zip"`. This prevents word splitting and glob expansion on the caller-controlled `NAME` environment variable, which is sourced from `${{ inputs.name }}`.

