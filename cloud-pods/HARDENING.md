<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b) violation: The shell variable $NAME, which holds the user-controlled input `inputs.name` (mapped via the env: block), is expanded **unquoted** in two `localstack` CLI invocations inside the run: block. An unquoted expansion allows the shell to parse metacharacters (spaces, globs, semicolons, command substitution, etc.) out of the value, enabling command injection. Offending lines:
  Line 21: `localstack pod save $NAME`
  Line 23: `localstack pod load --yes $NAME`
Fix: quote the variable — `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Note: $ACTION is always used inside double-quoted strings or quoted comparisons, so it does not trigger this finding.

Locations:

- `action.yml:21`
- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Quoted the $NAME variable in both localstack CLI invocations in action.yml: changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The values are already correctly mapped through the env: block, so only the unquoted shell expansions needed to be fixed.

