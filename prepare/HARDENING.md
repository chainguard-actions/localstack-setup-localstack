<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command string. The step `Save PR number` contains `echo ${{ github.event.number }} > ./pr-id.txt`. The value of `github.event.number` is substituted into the shell command by the Actions runner before the shell ever sees it, allowing an attacker to inject arbitrary shell metacharacters. The fix is to pass the value via an `env:` variable and double-quote it in the script: `env:\n  PR_NUMBER: ${{ github.event.number }}\nrun: echo "$PR_NUMBER" > ./pr-id.txt`

Locations:

- `action.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml: moved `${{ github.event.number }}` from the `run:` shell command into an `env:` block as `PR_NUMBER: ${{ github.event.number }}`, and updated the run command to use `echo "$PR_NUMBER" > ./pr-id.txt`. This prevents the GitHub Actions runner from substituting the value directly into the shell command string before the shell sees it.

