<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a `run:` shell command string. The expression `${{ github.event.number }}` is substituted into the shell command before the shell parses it, allowing an attacker who can control `github.event.number` to inject arbitrary shell commands. The offending line is: `run: echo ${{ github.event.number }} > ./pr-id.txt`. Fix by moving the value into an env var and quoting it: `env:\n  PR_NUMBER: ${{ github.event.number }}\nrun: echo "$PR_NUMBER" > ./pr-id.txt`

Locations:

- `action.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 14: moved `${{ github.event.number }}` out of the `run:` shell string into the step's `env:` block as `PR_NUMBER`, and updated the shell command to use the quoted variable `"$PR_NUMBER"` instead of the direct expression interpolation.

