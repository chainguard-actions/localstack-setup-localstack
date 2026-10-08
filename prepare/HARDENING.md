<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: The `run:` block on line 14 directly interpolates the GitHub Actions expression `${{ github.event.number }}` inside a shell command (`echo ${{ github.event.number }} > ./pr-id.txt`). GitHub Actions substitutes this value into the shell script before execution, allowing an attacker to inject arbitrary shell commands by crafting a malicious PR number (e.g., via a forged event payload). The value should be passed via an `env:` variable and the shell variable should be double-quoted instead.

Locations:

- `action.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection on line 14 of action.yml: moved `${{ github.event.number }}` out of the inline `run:` shell command and into an `env:` block as `PR_NUMBER`. The shell script now uses `"$PR_NUMBER"` (double-quoted) instead of the direct expression interpolation, preventing arbitrary shell command injection via a crafted PR event payload.

