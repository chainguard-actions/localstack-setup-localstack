<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var $NAME holds the value of `inputs.name` (a workflow-controllable input) and is expanded **unquoted** in two shell commands inside the `run:` block:
  - `localstack pod save $NAME` (line 21)
  - `localstack pod load --yes $NAME` (line 24)

An attacker-controlled value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) in `inputs.name` will be parsed by the shell, enabling command injection. The fix is to double-quote every expansion: `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:21`
- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted $NAME variable expansions in action.yml. Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The variable was already correctly placed in the env: block; only the shell-level quoting was missing.

