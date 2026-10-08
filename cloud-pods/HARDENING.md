<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): The env var $NAME, which holds the value of inputs.name (an untrusted caller-controlled input), is expanded unquoted in two shell commands inside the run block: `localstack pod save $NAME` and `localstack pod load --yes $NAME`. An attacker can supply a value containing shell metacharacters (e.g. semicolons, backticks, $(...)) to inject arbitrary commands. The fix is to double-quote the expansion: `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:22`
- `action.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted `$NAME` variable expansions in action.yml. Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The variable is already correctly moved to the env block (using `${{ inputs.name }}`), but the shell expansion was unquoted, allowing shell metacharacter injection. Double-quoting prevents word splitting and command injection.

