<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--prepare/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--prepare/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: A GitHub Actions expression is directly interpolated inside a `run:` shell command string. On line 17, `run: echo ${{ github.event.number }} > ./pr-id.txt` embeds `${{ github.event.number }}` directly in the shell command. The value is substituted by the Actions runner before the shell parses it, allowing an attacker who can control the PR number (e.g. via a crafted event payload) to inject arbitrary shell commands. Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g.:

```yaml
env:
  PR_NUMBER: ${{ github.event.number }}
run: echo "$PR_NUMBER" > ./pr-id.txt
```

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in hardened/action/action.yml line 17: moved `${{ github.event.number }}` from the `run:` shell command into an `env:` block as `PR_NUMBER`, and updated the shell command to use the quoted variable `"$PR_NUMBER"` instead of the direct expression interpolation.

