<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--cloud-pods/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--cloud-pods/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The shell variable $NAME, which holds the value of inputs.name (an attacker-controlled input), is expanded unquoted in two run: commands: `localstack pod save $NAME` and `localstack pod load --yes $NAME`. An unquoted expansion allows the shell to parse metacharacters (semicolons, pipes, backticks, spaces, glob characters, etc.) from the value, enabling command injection. The variable must be double-quoted: `localstack pod save "$NAME"` and `localstack pod load --yes "$NAME"`.

Locations:

- `action.yml:20`
- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted $NAME variable expansions in action.yml. Changed `localstack pod save $NAME` to `localstack pod save "$NAME"` and `localstack pod load --yes $NAME` to `localstack pod load --yes "$NAME"`. The inputs.name value was already correctly placed in the env: block, so only the quoting of the shell variable needed to be fixed to prevent command injection via shell metacharacters.

