<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' run: block directly interpolates `${{ inputs.preview-url }}` inside the shell command string. YAML template substitution occurs before the shell processes the string, so an attacker-controlled value in `inputs.preview-url` can inject arbitrary shell metacharacters. Offending lines:
  `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
  `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`
Fix: move the input into an env: variable and reference it as a quoted shell variable, e.g. `env: PREVIEW_URL: ${{ inputs.preview-url }}` then use `"${LS_PREVIEW_URL:-$PREVIEW_URL}"` in the script.

Locations:

- `action.yml:55`
- `action.yml:56`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' run: block writes the untrusted input `${{ inputs.preview-url }}` directly to $GITHUB_ENV without sanitization. An attacker can supply a value containing newlines to inject arbitrary environment variable assignments (e.g. overwriting PATH or other variables consumed by later steps). The required sanitization step `printf '%s' "$VAR" | tr -d '\n\r'` is absent before every write to $GITHUB_ENV in this step. Offending line:
  `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `action.yml:56`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml: (1) Moved `${{ inputs.preview-url }}` from the run: shell block into an env: block as `INPUT_PREVIEW_URL: ${{ inputs.preview-url }}`, then referenced it as `$INPUT_PREVIEW_URL` in the shell script to prevent script injection. (2) Added sanitization with `printf '%s' "..." | tr -d '\n\r'` before every write to $GITHUB_ENV to prevent newline-based environment variable injection attacks. The 'static-inline-injection' findings at lines 65/66 are the same as the script-injection findings at lines 55/56 — all resolved by the same env: block fix.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Load the PR ID' step in action.yml (line 41) to sanitize content read from pr-id.txt before writing to $GITHUB_OUTPUT. The fix reads the raw content into a variable first, then strips newlines using 'printf "%s" "$raw" | tr -d "\n\r"', then writes the sanitized value. This matches the pattern already correctly used in the 'Load the Ephemeral Instance URL' step. Also fixed the unquoted $GITHUB_OUTPUT to "$GITHUB_OUTPUT".

