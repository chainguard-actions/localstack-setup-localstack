<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' run: block directly interpolates ${{ inputs.preview-url }} inside shell command strings. The GitHub Actions template engine substitutes this value before the shell executes it, allowing an attacker to inject arbitrary shell commands via the `preview-url` input. Offending lines:
  Line 64: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
  Line 65: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `action.yml:64`
- `action.yml:65`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' run: block writes the untrusted input value ${{ inputs.preview-url }} directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before the write). A newline character in the input value could inject arbitrary environment variable assignments into subsequent steps. Offending line:
  Line 65: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `action.yml:65`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml: (1) Moved ${{ inputs.preview-url }} from the run: shell script to the env: block as INPUT_PREVIEW_URL to prevent script injection. (2) Added sanitization using 'printf "%s" "$resolved" | tr -d "\n\r"' before writing to $GITHUB_ENV to prevent newline-based environment variable injection. The shell script now uses $INPUT_PREVIEW_URL (a plain environment variable) instead of the template expression directly.

