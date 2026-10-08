<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The `run:` block expands the env var `${NAME}` (sourced from `inputs.name`, an attacker-controlled value) without double-quoting in two `localstack` commands:
  Line 36: `localstack state export ${NAME}.zip`
  Line 39: `localstack state import ${NAME}.zip`
An unquoted shell variable expansion allows an attacker to inject shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) via the `name` input, potentially achieving command injection. The fix is to quote the expansion: `"${NAME}.zip"`.

Locations:

- `action.yml:36`
- `action.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted variable expansions in action.yml. Changed `localstack state export ${NAME}.zip` to `localstack state export "${NAME}.zip"` and `localstack state import ${NAME}.zip` to `localstack state import "${NAME}.zip"` on lines 36 and 39 respectively. The `NAME` env var was already properly sourced from the step's env block, but the unquoted expansion allowed shell metacharacter injection via whitespace, glob characters, or other shell special characters in the filename.

