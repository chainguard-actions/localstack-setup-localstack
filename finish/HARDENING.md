<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Load the Ephemeral Instance URL' run: block directly interpolates `${{ inputs.preview-url }}` inside shell command strings. The expression appears in a bash conditional test (`if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]`) and in an echo command (`echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`). Any `${{ ... }}` expression inside a run: block is substituted by the Actions runner before the shell ever sees it, allowing an attacker-controlled input to inject arbitrary shell metacharacters.

Locations:

- `action.yml:61`
- `action.yml:62`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' run: block writes the untrusted input `${{ inputs.preview-url }}` directly to $GITHUB_ENV without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled value containing newlines could inject arbitrary environment variable definitions into subsequent steps. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:62`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml by: (1) moving `${{ inputs.preview-url }}` from the run: block into an env: block as INPUT_PREVIEW_URL, eliminating script injection; (2) replacing all inline ${{ inputs.preview-url }} references in the shell script with the safe env var $INPUT_PREVIEW_URL; (3) adding `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before every write to $GITHUB_ENV to prevent newline/CRLF injection attacks.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Load the PR ID' step in action.yml (line 37). Changed from `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` to a three-line script that: (1) captures raw content with `raw=$(cat pr-id.txt)`, (2) sanitizes it with `safe=$(printf '%s' "$raw" | tr -d '\n\r')`, and (3) writes the sanitized value with `echo "pr_id=$safe" >> $GITHUB_OUTPUT`. This prevents newline injection attacks from malicious artifact content.

