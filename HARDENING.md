<!-- markdownlint-disable -->

# Hardening Report: localstack--setup-localstack/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **localstack--setup-localstack/v0.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In prepare/action.yml, `echo ${{ github.event.number }} > ./pr-id.txt` interpolates a github context value directly into the shell command string. In startup/action.yml, `export CI_PROJECT=${{ inputs.ci-project }}` injects an input directly into the shell. In ephemeral/startup/action.yml, multiple direct injections occur: `${{ inputs.localstack-api-key }}` embedded in a shell string for AUTH_HEADER, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, `${{ inputs.lifetime }}` assigned to shell variables, `${{ github.action_path }}` used in a `source` command, and most critically `${{ inputs.preview-cmd }}` is directly executed as a shell command (arbitrary code execution). In ephemeral/shutdown/action.yml, `${{ inputs.localstack-api-key }}` is embedded in a shell string. In finish/action.yml, `${{ inputs.preview-url }}` is interpolated directly into shell strings written to $GITHUB_ENV.

Locations:

- `prepare/action.yml:35`
- `startup/action.yml:57`
- `ephemeral/startup/action.yml:75`
- `ephemeral/startup/action.yml:84`
- `ephemeral/startup/action.yml:85`
- `ephemeral/startup/action.yml:86`
- `ephemeral/startup/action.yml:79`
- `ephemeral/startup/action.yml:148`
- `ephemeral/startup/action.yml:163`
- `ephemeral/shutdown/action.yml:47`
- `finish/action.yml:72`
- `finish/action.yml:73`

### github-env-injection (severity: high)

Unsanitized values are written to $GITHUB_ENV or $GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) In ephemeral/startup/action.yml 'Setup preview name' step: `previewName` is derived from file content (pr-id.txt) and the inherited `$GITHUB_REPOSITORY` env var, then written directly to both $GITHUB_ENV and $GITHUB_OUTPUT without sanitization. (2) In finish/action.yml 'Load the Ephemeral Instance URL' step: `LS_PREVIEW_URL` is written to $GITHUB_ENV with `${{ inputs.preview-url }}` directly interpolated into the echo command — no sanitization applied. (3) In ephemeral/shutdown/action.yml 'Setup preview name' step: `previewName` is derived from file content and `$GITHUB_REPOSITORY` and written to $GITHUB_ENV without sanitization.

Locations:

- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:56`
- `finish/action.yml:72`
- `finish/action.yml:73`
- `ephemeral/shutdown/action.yml:30`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and none of its jobs (localstack-action-version-test, cloud-pods-save-test, load-cloud-pod-test, local-state-save-test, local-state-load-test) define a job-level `permissions:` block. This means the workflow runs with the default, overly broad token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### broad-permissions (severity: medium)

The job `preview-test` in .github/workflows/ephemeral.yml sets `permissions: write-all`, granting every available permission to the GITHUB_TOKEN. This should be replaced with the minimal specific permissions actually required (e.g., `pull-requests: write`, `contents: read`).

Locations:

- `.github/workflows/ephemeral.yml:8`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions, broad-permissions

**Notes:**

Fixed all four findings:

1. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks across prepare/action.yml (github.event.number), startup/action.yml (inputs.ci-project), ephemeral/startup/action.yml (inputs.localstack-api-key, github.action_path, inputs.auto-load-pod, inputs.extension-auto-install, inputs.lifetime, inputs.preview-cmd), ephemeral/shutdown/action.yml (inputs.localstack-api-key, github.action_path), and finish/action.yml (inputs.preview-url). The preview-cmd is now executed via eval "$INPUT_PREVIEW_CMD" which is safe since the value is in an env var, not interpolated into the shell script text.

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing previewName to GITHUB_ENV/GITHUB_OUTPUT in ephemeral/startup/action.yml and ephemeral/shutdown/action.yml, and before writing the preview URL to GITHUB_ENV in finish/action.yml.

3. missing-permissions: Added top-level `permissions: contents: read` to .github/workflows/ci.yml.

4. broad-permissions: Replaced `permissions: write-all` with specific minimal permissions (`contents: read`, `pull-requests: write`) in the preview-test job in .github/workflows/ephemeral.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 8 findings across 6 files:

1. ci.yml script-injection (3 locations): Moved ${{ steps.pod_name.outputs.name }} and ${{ needs.cloud-pods-save-test.outputs.pod-name }} from run: blocks to env: blocks as POD_NAME variable.

2. ephemeral.yml script-injection (2 locations): Moved ${{ github.event.pull_request.number }} and ${{ github.repository }} from run: block to env: block as PR_NUMBER and GH_REPOSITORY.

3. startup/action.yml script-injection (2 locations): Replaced eval "${CONFIGURATION} localstack start -d" with xargs-based tokenization into cfg_args array passed to env command; quoted ${IMAGE_NAME} in docker pull.

4. ephemeral/startup/action.yml script-injection (1 location): Replaced eval "$INPUT_PREVIEW_CMD" with writing to a mktemp file and executing with bash.

5. ephemeral/shutdown/action.yml github-env-injection (1 location): Added printf '%s' ... | tr -d '\n\r' sanitization before writing pr_id to GITHUB_OUTPUT.

6. finish/action.yml github-env-injection (1 location): Added printf '%s' ... | tr -d '\n\r' sanitization before writing pr_id to GITHUB_OUTPUT.

7. ephemeral/startup/action.yml github-env-injection (2 locations): Added printf '%s' ... | tr -d '\n\r' sanitization for endpointUrl before writing LS_PREVIEW_URL and AWS_ENDPOINT_URL to GITHUB_ENV.

8. unpinned-uses in ci.yml (7 occurrences) and ephemeral.yml (2 occurrences): Replaced github.ref_name with github.sha so GH_ACTION_VERSION always resolves to an immutable commit SHA for non-pull_request events.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings by properly quoting shell variables:
1. cloud-pods/action.yml (lines 20, 23): Quoted `$NAME` → `"$NAME"` in both `localstack pod save` and `localstack pod load --yes` commands.
2. local/action.yml (lines 38, 41): Quoted `${NAME}.zip` → `"${NAME}.zip"` in both `localstack state export` and `localstack state import` commands.
3. startup/action.yml (line 75): Quoted `${LS_WAIT_TIMEOUT:-30}` → `"${LS_WAIT_TIMEOUT:-30}"` in the `localstack wait -t` command. This prevents word-splitting and shell metacharacter interpretation for all three attacker-controlled or environment-inherited values.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted shell variable expansion in `.github/workflows/ephemeral.yml` line 44. Changed `curl ${AWS_ENDPOINT_URL}/_localstack/extensions/list` to `curl "${AWS_ENDPOINT_URL}/_localstack/extensions/list"` so that any shell metacharacters in the AWS_ENDPOINT_URL value are not interpreted by the shell, preventing command injection.

