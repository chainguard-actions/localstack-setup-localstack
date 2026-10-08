<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--finish/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--finish/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: `${{ inputs.preview-url }}` is directly interpolated inside a `run:` shell script in the 'Load the Ephemeral Instance URL' step. The YAML template engine substitutes the expression before the shell processes it, allowing an attacker to inject arbitrary shell commands via the `preview-url` input. Offending lines:
  Line 65: `if [[ -n "${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" ]]; then`
  Line 66: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

Locations:

- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step writes a value derived from the untrusted input `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in `preview-url` could inject additional environment variable definitions, potentially overwriting security-sensitive variables used by subsequent steps. Offending line:
  Line 66: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

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

Fixed the 'Load the Ephemeral Instance URL' step in action.yml: (1) Moved ${{ inputs.preview-url }} out of the run: shell script into an env: block as INPUT_PREVIEW_URL, referencing it as $INPUT_PREVIEW_URL in the shell. (2) Added sanitization using `printf '%s' "..." | tr -d '\n\r'` before writing to $GITHUB_ENV to prevent newline-based environment variable injection. Also sanitized the ls-preview-url.txt file read path for consistency.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Load the PR ID' step in action.yml (line 38) to sanitize the content read from pr-id.txt before writing to $GITHUB_OUTPUT. Changed from `echo "pr_id=$(<pr-id.txt)" >> $GITHUB_OUTPUT` to a multi-line script that reads into a `raw` variable, strips newlines with `printf '%s' "$raw" | tr -d '\n\r'`, then writes the sanitized value. This prevents newline injection attacks where attacker-controlled artifact content could inject additional key-value pairs into the GitHub output environment file.

