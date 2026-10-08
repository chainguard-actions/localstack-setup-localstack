<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The line `export CI_PROJECT=${{ inputs.ci-project }}` injects the raw value of inputs.ci-project directly into the shell script before the shell ever sees it. An attacker controlling this input can inject arbitrary shell commands (e.g. via semicolons, backticks, or $(...) substitutions).

Locations:

- `action.yml:77`

### script-injection (severity: high)

Sub-rule (b): The env var CONFIGURATION (set from ${{ inputs.configuration }}) is expanded unquoted inside `eval "${CONFIGURATION} localstack start -d"`. Even though the value is routed through an env: block, using it unquoted in eval allows an attacker to supply shell metacharacters (;, |, &, $(...), backticks) via inputs.configuration to execute arbitrary commands.

Locations:

- `action.yml:78`

### script-injection (severity: high)

Sub-rule (b): The env var IMAGE_NAME (derived from IMAGE_TAG which is set from ${{ inputs.image-tag }}) is expanded unquoted in `docker pull ${IMAGE_NAME} &`. An attacker controlling inputs.image-tag can inject shell metacharacters into this command.

Locations:

- `action.yml:76`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all four security findings in hardened/action/action.yml:
1. Removed inline `${{ inputs.ci-project }}` expression from run: block; added `CI_PROJECT: ${{ inputs.ci-project }}` to the step's env: block.
2. Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe xargs-based tokenization loop that exports each KEY=VALUE pair individually, then calls `localstack start -d` directly without eval — eliminating the shell metacharacter injection risk.
3. Quoted `${IMAGE_NAME}` as `"${IMAGE_NAME}"` in the `docker pull` command.
4. Also fixed the `_actions` glob search by replacing it with `$(cd "$GITHUB_ACTION_PATH/.." && pwd)` since this is startup/action.yml (one level below the repo root).

