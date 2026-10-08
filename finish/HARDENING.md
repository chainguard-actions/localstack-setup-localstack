<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Load the Ephemeral Instance URL' step directly interpolates `${{ inputs.preview-url }}` inside a `run:` shell command string. The expression is substituted by the Actions runner before the shell parses the command, allowing an attacker-controlled value to inject arbitrary shell commands. Offending lines:
  Line 65: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
  Line 66: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`
Fix: move `inputs.preview-url` into an `env:` variable and reference it as a quoted shell variable (e.g. `"$PREVIEW_URL"`).

Locations:

- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` — an untrusted caller-controlled input — directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline character in the input value can inject arbitrary environment variable definitions into subsequent steps.
  Line 66: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`
Fix: sanitize the value before writing, e.g.:
  `safe=$(printf '%s' "$PREVIEW_URL" | tr -d '\n\r')`
  `echo "LS_PREVIEW_URL=$safe" >> "$GITHUB_ENV"`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml: (1) moved `inputs.preview-url` out of the run: block into an `env:` map as `PREVIEW_URL: ${{ inputs.preview-url }}`, eliminating shell injection; (2) added sanitization via `printf '%s' "$raw" | tr -d '\n\r'` before writing to $GITHUB_ENV, preventing newline injection; (3) all shell references now use plain `$PREVIEW_URL` / `$raw` / `$safe` variables instead of inline ${{ }} expressions.

### Iteration 2

**Fixes applied:** github-env-injection, github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in action.yml:
1. 'Load the PR ID' step: Added sanitization for pr-id.txt content before writing to $GITHUB_OUTPUT. Now reads into 'raw', sanitizes with 'printf "%s" "$raw" | tr -d "\n\r"', then writes the safe value.
2. 'Load the Ephemeral Instance URL' step: Added sanitization in the elif branch for ls-preview-url.txt content before writing to $GITHUB_ENV. Now follows the same pattern as the if branch above it — reads into 'raw', sanitizes with printf/tr, then writes the safe value.

