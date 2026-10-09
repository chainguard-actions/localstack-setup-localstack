<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command string. The step 'Save PR number' uses `echo ${{ github.event.number }} > ./pr-id.txt`, which injects the expression value directly into the shell before execution. An attacker who can control `github.event.number` (e.g. via a crafted event payload) could inject arbitrary shell commands. Fix: move the value into an env var and quote it — e.g. `env: { PR_NUMBER: "${{ github.event.number }}" }` and `run: echo "$PR_NUMBER" > ./pr-id.txt`.

Locations:

- `action.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Save PR number' step of action.yml (line 14). Moved `${{ github.event.number }}` out of the `run:` shell string into an `env:` block as `PR_NUMBER`, and updated the shell command to use the quoted variable `"$PR_NUMBER"` instead of the direct expression interpolation.

