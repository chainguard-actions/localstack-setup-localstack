<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var $NAME holds a value sourced from inputs.name (a workflow-controllable input) and is expanded **unquoted** in two shell commands: `localstack pod save $NAME` and `localstack pod load --yes $NAME`. An attacker-controlled value containing shell metacharacters (e.g. `;`, `|`, `$(...)`) will be parsed by bash before being passed to the command, enabling command injection. Fix: quote the variable in both places — `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:21`
- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in hardened/action/action.yml by quoting the $NAME variable in both shell commands where it was used unquoted: changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The $NAME variable is already correctly moved into the step's env: block (as `NAME: "${{ inputs.name }}"`), so only the quoting fix was needed.

