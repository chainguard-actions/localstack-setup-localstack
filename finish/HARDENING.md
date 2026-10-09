<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' step directly interpolates `${{ inputs.preview-url }}` inside a `run:` shell command string. This occurs twice: once in the `if` condition (`${LS_PREVIEW_URL:-${{ inputs.preview-url }}}`) and once in the `echo` command. An attacker controlling the `preview-url` input can inject arbitrary shell metacharacters (`;`, `|`, `$(...)`, etc.) that are evaluated by the shell before any quoting takes effect. The value must be passed via an `env:` block and then double-quoted in the shell script instead.

Locations:

- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes the attacker-controlled value `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` (via `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in `inputs.preview-url` can inject additional key=value pairs into the GitHub environment, allowing an attacker to override arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:66`

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

Fixed the 'Load the Ephemeral Instance URL' step in hardened/action/action.yml. Moved `${{ inputs.preview-url }}` from the run: block into an env: block as INPUT_PREVIEW_URL. Updated the shell script to use $INPUT_PREVIEW_URL instead of the inline expression. Added sanitization (printf '%s' ... | tr -d '\n\r') before writing any value to $GITHUB_ENV to prevent newline-based environment injection. The sanitization is done in two steps (capture raw value, then sanitize) to avoid swallowing errors under errexit.

