<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack--startup/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack--startup/v0.3.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Start LocalStack' run: block contains multiple script injection violations:

(a) Direct expression interpolation: `export CI_PROJECT=${{ inputs.ci-project }}` — the `inputs.ci-project` value is interpolated directly into the shell command string before the shell ever sees it, allowing an attacker to inject arbitrary shell commands via the `ci-project` input.

(b) Unquoted shell variable expansion of untrusted data:
- `docker pull ${IMAGE_NAME} &` — `IMAGE_NAME` is derived from `IMAGE_TAG` which is set from `${{ inputs.image-tag }}`. The variable is unquoted, allowing shell metacharacter injection.
- `eval "${CONFIGURATION} localstack start -d"` — `CONFIGURATION` is set from `${{ inputs.configuration }}` via the env: block. The variable is expanded unquoted inside the eval string, allowing an attacker to inject arbitrary commands through the `configuration` input (e.g., by embedding shell metacharacters or additional commands).

Locations:

- `action.yml:58`
- `action.yml:57`
- `action.yml:59`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ci-project }}" appears directly in run: block of step "Start LocalStack"; move to env: map

Locations:

- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. **static-inline-injection / script-injection (ci-project)**: Removed `export CI_PROJECT=${{ inputs.ci-project }}` direct interpolation. Added `CI_PROJECT: ${{ inputs.ci-project }}` to the step's `env:` block and changed the run script to reference it as `export CI_PROJECT="$CI_PROJECT"`.

2. **script-injection (IMAGE_NAME unquoted)**: Quoted `docker pull ${IMAGE_NAME} &` → `docker pull "${IMAGE_NAME}" &`.

3. **script-injection (eval with CONFIGURATION)**: Replaced `eval "${CONFIGURATION} localstack start -d"` with safe xargs-based tokenization: tokenizes CONFIGURATION into a bash array `cfg_args` using `xargs printf '%s\0'` and then runs `env "${cfg_args[@]}" localstack start -d`, preventing arbitrary command injection while preserving the KEY=VALUE env var passing semantics.

4. **_actions/* glob**: Also fixed the `GH_ACTION_ROOT` step to use `$GITHUB_ACTION_PATH` instead of globbing `_actions/*`, with proper `./` prefix for local uses: references.

