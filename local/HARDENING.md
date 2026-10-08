<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--local/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--local/v0.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansion of untrusted data. In the `run:` block, `NAME` is sourced from `inputs.name` (attacker-controlled, set via `env: NAME: "${{ inputs.name }}"`), but it is expanded unquoted in two shell commands: `localstack state export ${NAME}.zip` and `localstack state import ${NAME}.zip`. An attacker-controlled value containing shell metacharacters (e.g. semicolons, pipes, backticks) could result in arbitrary command execution. The fix is to quote the expansions: `"${NAME}.zip"`.

Locations:

- `action.yml:44`
- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two instances of unquoted shell variable expansion in action.yml. On lines 44 and 47, `${NAME}.zip` was changed to `"${NAME}.zip"` to prevent word splitting and glob expansion of the attacker-controlled `inputs.name` value. The variable was already safely passed via the `env:` block, but the unquoted expansion in the shell commands still posed an injection risk.

