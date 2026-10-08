<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.event.number }}` is directly interpolated inside a `run:` shell command string. Before the shell executes the command, GitHub Actions substitutes the expression value into the script text, allowing an attacker who can control the event payload (e.g. via a crafted PR number) to inject arbitrary shell commands. The offending line is: `run: echo ${{ github.event.number }} > ./pr-id.txt`. Fix: move the value into an env var and reference it as a quoted shell variable, e.g. `env: PR_NUMBER: ${{ github.event.number }}` then `run: echo "$PR_NUMBER" > ./pr-id.txt`.

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in action.yml line 17: moved `${{ github.event.number }}` out of the `run:` shell string and into an `env:` block as `PR_NUMBER`. The shell command now uses the quoted variable `"$PR_NUMBER"` instead of the direct expression interpolation.

