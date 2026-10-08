<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' run: block directly interpolates the untrusted expression `${{ inputs.preview-url }}` inside the shell script string. This allows an attacker-controlled value to be parsed by bash before any quoting takes effect. Offending lines:
  `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
  `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`
The value should be passed via an `env:` block and then double-quoted in the shell script.

Locations:

- `action.yml:62`
- `action.yml:63`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' run: block writes untrusted values to $GITHUB_ENV without sanitization:
(1) `${{ inputs.preview-url }}` is interpolated directly into the echo command that writes to $GITHUB_ENV — an attacker-controlled input can inject newlines to set arbitrary environment variables (sub-rule b/d).
(2) `${LS_PREVIEW_URL}` is an inherited process env var (set by the calling workflow, not by this run: block) and is forwarded to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step (sub-rule e).
Both writes allow environment variable injection via newline characters.

Locations:

- `action.yml:62`
- `action.yml:63`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml: (1) Moved `${{ inputs.preview-url }}` from the run: shell script into an env: block as INPUT_PREVIEW_URL, eliminating direct shell interpolation of the untrusted expression. (2) Replaced all direct ${{ inputs.preview-url }} references in the shell script with the env var $INPUT_PREVIEW_URL. (3) Added newline sanitization for all user-controlled values written to $GITHUB_ENV using `printf '%s' "$value" | tr -d '\n\r'`, following the two-step capture-then-sanitize pattern to correctly handle bash errexit semantics.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Load the PR ID' step in hardened/action/action.yml (line 42). The original code `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` read externally-controlled file content directly into GITHUB_OUTPUT without sanitization. The fix captures the raw content first, then sanitizes it with `printf '%s' "$raw" | tr -d '\n\r'` to strip newline/carriage-return characters before writing to GITHUB_OUTPUT, preventing injection of additional key=value pairs.

