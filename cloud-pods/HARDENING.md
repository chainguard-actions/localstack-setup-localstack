<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var $NAME holds a value sourced from inputs.name (an attacker-controllable input) and is expanded unquoted in two shell commands inside the run: block. Unquoted shell variable expansion allows shell metacharacters (spaces, semicolons, pipes, glob characters, etc.) embedded in the input to be interpreted by the shell, enabling command injection. The offending lines are:
  `localstack pod save $NAME` (line 20)
  `localstack pod load --yes $NAME` (line 22)
Fix: quote the variable in both places — `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:20`
- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted variable expansions in action.yml: `localstack pod save $NAME` → `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` → `localstack pod load --yes "$NAME"`. The `inputs.name` value was already correctly placed in an env var (`NAME`), but the variable was expanded unquoted in the shell commands, allowing shell metacharacters to be interpreted. Quoting both expansions eliminates the injection risk.

