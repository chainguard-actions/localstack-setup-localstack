<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b) violation: The shell variable `$NAME` (sourced from `inputs.name` via the `env:` block) is expanded unquoted in two `run:` shell commands: `localstack pod save $NAME` and `localstack pod load --yes $NAME`. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) out of the value, enabling command injection. The variable should be double-quoted: `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:18`
- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two instances of unquoted `$NAME` variable expansion in action.yml. Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The variable was already correctly sourced from inputs via the env: block, but needed double-quoting at the point of use to prevent shell word-splitting and glob expansion of attacker-controlled values.

