<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. The step 'Save PR number' contains `echo ${{ github.event.number }} > ./pr-id.txt`. The value `github.event.number` is substituted into the shell command string before the shell executes it, allowing an attacker to craft a pull request number (or other event payload field) that injects arbitrary shell commands. The value should be passed via an `env:` variable and then referenced as a quoted shell variable (e.g., `env: PR_NUMBER: ${{ github.event.number }}` and `run: echo "$PR_NUMBER" > ./pr-id.txt`)

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Save PR number' step of action.yml. Moved `${{ github.event.number }}` out of the `run:` shell string into an `env:` block as `PR_NUMBER: ${{ github.event.number }}`, and updated the shell command to use the quoted variable `echo "$PR_NUMBER" > ./pr-id.txt`.

