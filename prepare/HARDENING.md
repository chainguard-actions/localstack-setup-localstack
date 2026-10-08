<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.event.number }}` is directly interpolated inside a `run:` shell command string on line 16. Before the shell executes the command, the Actions runner substitutes the expression value verbatim into the script text, allowing an attacker who can control the PR number field (e.g. via a crafted event payload) to inject arbitrary shell commands. The offending line is: `run: echo ${{ github.event.number }} > ./pr-id.txt`. Fix by moving the value into an environment variable and referencing it as a quoted shell variable: `env:\n  PR_NUMBER: ${{ github.event.number }}\nrun: echo "$PR_NUMBER" > ./pr-id.txt`

Locations:

- `action.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection on action.yml line 16: moved `${{ github.event.number }}` out of the `run:` shell string and into an `env:` block as `PR_NUMBER`. The shell command now uses `echo "$PR_NUMBER" > ./pr-id.txt` instead of directly interpolating the expression, preventing shell command injection via a crafted event payload.

