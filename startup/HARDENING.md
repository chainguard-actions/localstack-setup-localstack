<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ inputs.ci-project }}` is directly interpolated inside a `run:` shell command: `export CI_PROJECT=${{ inputs.ci-project }}`. An attacker controlling the `ci-project` input can inject arbitrary shell commands that execute on the runner.

Locations:

- `action.yml:80`

### script-injection (severity: high)

Sub-rule (b): `eval "${CONFIGURATION} localstack start -d"` executes the `CONFIGURATION` env var as a shell command. `CONFIGURATION` is sourced directly from `inputs.configuration` (an untrusted caller-controlled input). Passing attacker-controlled content to `eval` allows arbitrary shell command injection regardless of the env-var indirection.

Locations:

- `action.yml:81`

### script-injection (severity: high)

Sub-rule (b): `docker pull ${IMAGE_NAME} &` uses an unquoted shell expansion of `IMAGE_NAME`, which is derived from the untrusted `inputs.image-tag` input (via the `IMAGE_TAG` env var). An unquoted expansion allows shell metacharacters (spaces, semicolons, pipes, etc.) in the input value to be interpreted by the shell, enabling command injection.

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in hardened/action/action.yml:
1. Quoted `${IMAGE_NAME}` in `docker pull` to prevent shell metacharacter injection from inputs.image-tag.
2. Moved `${{ inputs.ci-project }}` from inline run: block to env: block as `CI_PROJECT_INPUT`, referenced as `$CI_PROJECT_INPUT` in the shell script.
3. Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe xargs-based tokenization: builds a bash array `cfg_args` from the CONFIGURATION env var using `xargs printf '%s\0'` and a null-delimited read loop, then passes it to `env "${cfg_args[@]}" localstack start -d` — eliminating eval of untrusted input entirely.
4. Also replaced the unsafe `_actions` glob pattern with `realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/.."` for correct path resolution under any owner/repo name.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed action.yml line 43: Added './' prefix to GH_ACTION_ROOT assignment. Changed from 'GH_ACTION_ROOT=$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")' to 'GH_ACTION_ROOT=./$(realpath --relative-to="$GITHUB_WORKSPACE" "$GITHUB_ACTION_PATH/..")'. This ensures the value starts with './' making it a valid workspace-relative path for the 'uses: ${{ env.GH_ACTION_ROOT }}/tools' reference on line 49.

