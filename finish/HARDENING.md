<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' run: block directly interpolates the attacker-controllable expression `${{ inputs.preview-url }}` inside shell command strings. This allows an attacker to inject arbitrary shell commands via the `preview-url` input. Offending lines: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]` and `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`.

Locations:

- `action.yml:59`
- `action.yml:60`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' run: block writes untrusted values to $GITHUB_ENV without sanitization (missing `printf '%s' ... | tr -d '\n\r'`). (1) `${{ inputs.preview-url }}` is directly interpolated and written to $GITHUB_ENV — an attacker-controlled input can inject newlines to set arbitrary environment variables. (2) The inherited process env var `LS_PREVIEW_URL` (set by the calling workflow, therefore untrusted) is forwarded to $GITHUB_ENV unsanitized via `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-...}" >> $GITHUB_ENV`.

Locations:

- `action.yml:60`

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
1. Moved `${{ inputs.preview-url }}` out of the run: block into an env: block as `INPUT_PREVIEW_URL`.
2. Replaced direct interpolation in shell with `$INPUT_PREVIEW_URL` env var reference.
3. Added sanitization via `printf '%s' "$value" | tr -d '\n\r'` before writing any value to $GITHUB_ENV, covering both the env-var/input path and the file-read path.
4. Each command substitution is assigned to a separate variable to preserve errexit behavior.

