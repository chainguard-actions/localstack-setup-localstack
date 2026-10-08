<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var `$NAME` (sourced from `inputs.name`, a workflow-controllable value) is expanded unquoted in two shell commands inside the `run:` block. Unquoted shell variable expansion allows an attacker to inject shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) via the `name` input. Offending lines:
  - `localstack pod save $NAME` (line 21) — should be `localstack pod save "$NAME"`
  - `localstack pod load --yes $NAME` (line 24) — should be `localstack pod load --yes "$NAME"`
Note: `$ACTION` is only used in a quoted `[ "$ACTION" = ... ]` test, so it is safe.

Locations:

- `action.yml:21`
- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two instances of unquoted `$NAME` variable expansion in action.yml. Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` (line 21) and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"` (line 24). The `$NAME` variable is sourced from `inputs.name` and was vulnerable to shell metacharacter injection when left unquoted. The `$ACTION` variable was already safely used only in quoted test expressions and required no changes.

