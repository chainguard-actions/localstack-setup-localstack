<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The `run:` block uses `${NAME}.zip` unquoted in two shell commands (`localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`). The `NAME` env var is sourced directly from `inputs.name` (an untrusted, caller-controlled input). An unquoted shell expansion allows an attacker to supply a value containing shell metacharacters (spaces, globs, semicolons, `$(...)`, etc.) that the shell will interpret before passing to the command, enabling command injection. The fix is to quote the expansion: `"${NAME}.zip"`.

Locations:

- `action.yml:46`
- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Quoted the two unquoted `${NAME}.zip` shell expansions in action.yml (lines 46 and 49) to `"${NAME}.zip"`. The NAME variable is already safely passed via the step's env block from `inputs.name`, but the unquoted expansion allowed shell metacharacters (spaces, globs, semicolons, etc.) to be interpreted. Adding double quotes prevents word splitting and glob expansion, eliminating the injection risk.

