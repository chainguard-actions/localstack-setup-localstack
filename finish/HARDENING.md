<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' run: block directly interpolates `${{ inputs.preview-url }}` inside the shell command string. This occurs twice: once in the bash `if` condition (`${LS_PREVIEW_URL:-${{ inputs.preview-url }}}`) and once in the `echo` statement. Any attacker-controlled value for `inputs.preview-url` containing shell metacharacters (`;`, `|`, `$(...)`, etc.) will be executed by the shell before quoting can protect it, because YAML template substitution happens before the shell ever sees the string.

Locations:

- `action.yml:57`
- `action.yml:58`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' run: block writes `${{ inputs.preview-url }}` — an untrusted caller-controlled input — directly into `$GITHUB_ENV` via `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. No sanitization step (`printf '%s' ... | tr -d '\n\r'`) is applied before the write. An attacker can inject newlines into `inputs.preview-url` to set arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:58`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-url }}" appears directly in run: block of step "Load the Ephemeral Instance URL"; move to env: map

Locations:

- `action.yml:65`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-url }}" appears directly in run: block of step "Load the Ephemeral Instance URL"; move to env: map

Locations:

- `action.yml:66`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed the 'Load the Ephemeral Instance URL' step in action.yml: moved `${{ inputs.preview-url }}` from the shell run: block into the step's env: block as INPUT_PREVIEW_URL. The shell script now uses $INPUT_PREVIEW_URL (a plain env var) instead of the inline expression. Added newline sanitization (printf '%s' | tr -d '\n\r') before all writes to $GITHUB_ENV to prevent newline injection. Used the two-step raw/safe pattern to avoid swallowing errors under bash errexit.

