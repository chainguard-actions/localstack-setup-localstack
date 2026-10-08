<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var $NAME, which holds the value of inputs.name (a workflow-controllable input), is used unquoted in two shell commands inside the run: block. Unquoted shell variable expansion allows an attacker to inject shell metacharacters (e.g. semicolons, pipes, backticks) via the `name` input, leading to arbitrary command execution. Offending lines:
  - `localstack pod save $NAME` (line 21)
  - `localstack pod load --yes $NAME` (line 24)
Fix: quote the variable in both places: `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:21`
- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Quoted the $NAME variable in both shell commands in action.yml: `localstack pod save "$NAME"` (line 21) and `localstack pod load --yes "$NAME"` (line 24). The variable was already correctly moved to the env: block; only the missing quotes needed to be added to prevent shell metacharacter injection via the `name` input.

