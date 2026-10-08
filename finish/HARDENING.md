<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: The 'Load the Ephemeral Instance URL' step directly interpolates `${{ inputs.preview-url }}` inside a `run:` shell script. YAML template substitution occurs before the shell processes the command, so an attacker-controlled value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) can achieve arbitrary command execution. Offending lines:
  Line 65: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
  Line 66: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes the untrusted input `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A value containing newlines can inject arbitrary environment variable assignments (e.g. `ACTIONS_RUNTIME_TOKEN=...`) into subsequent steps. The offending write is: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml:
1. Moved `${{ inputs.preview-url }}` from the run: block into the step's env: block as `INPUT_PREVIEW_URL` to prevent shell injection.
2. Updated the shell script to use `$INPUT_PREVIEW_URL` instead of the inline expression.
3. Added sanitization with `printf '%s' "$resolved" | tr -d '\n\r'` before writing to $GITHUB_ENV to prevent newline-based environment variable injection.
4. Also sanitized the ls-preview-url.txt file read path for consistency.
5. Quoted $GITHUB_ENV references as "$GITHUB_ENV" per best practices.

