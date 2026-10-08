<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var `NAME` holds the caller-controlled value `${{ inputs.name }}` and is expanded **unquoted** in two shell commands: `localstack pod save $NAME` and `localstack pod load --yes $NAME`. An attacker can supply a pod name containing shell metacharacters (e.g. spaces, semicolons, `$(...)`, backticks) to inject arbitrary commands. Fix: quote every expansion — `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:22`
- `action.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted `$NAME` variable expansions in action.yml. Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The NAME env var holds the caller-controlled `${{ inputs.name }}` value, and quoting it prevents shell metacharacters from being interpreted as shell commands.

