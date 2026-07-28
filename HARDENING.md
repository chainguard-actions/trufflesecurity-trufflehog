<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.96.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.96.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple GitHub Actions expressions are interpolated directly inside a run: shell command string in action.yml. The expressions ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{github.event.pull_request.base.sha}}, and ${{github.event.pull_request.head.sha}} are all expanded by the template engine before the shell sees them, allowing an attacker to inject arbitrary shell commands via crafted event payloads. For example: `if [ "${{ github.event_name }}" == "push" ]; then` and `HEAD=${{ github.event.after }}`.

Locations:

- `action.yml:78`
- `action.yml:84`
- `action.yml:85`
- `action.yml:88`
- `action.yml:90`
- `action.yml:93`
- `action.yml:94`
- `action.yml:95`

### script-injection (severity: high)

Sub-rule (a): The expression ${{ steps.auth.outputs.credentials_file_path }} (a steps.*.outputs.* context value) is interpolated directly inside a run: shell command string in release-bot.yml. This value flows through YAML template substitution before the shell quotes it, allowing injection if the output value contains shell metacharacters. Offending line: `-v ${{ steps.auth.outputs.credentials_file_path }}:/tmp/keys/GCP_SA_TRUFFLE_RELEASE_BOT.json:ro \`

Locations:

- `.github/workflows/release-bot.yml:34`

### missing-permissions (severity: medium)

The workflow file performance.yml has no top-level permissions: key and none of its jobs define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository default (which may be read/write for all scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

The workflow file smoke.yml has no top-level permissions: key and none of its jobs define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository default (which may be read/write for all scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file secrets.yml has no top-level permissions: key and its single job has no job-level permissions: block. Without explicit permissions, the workflow inherits the repository default (which may be read/write for all scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/secrets.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, static-inline-injection

**Notes:**

Fixed all 6 findings across 5 files:
1. action.yml (script-injection + static-inline-injection): Moved github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, and github.event.pull_request.head.sha from inline ${{ }} expressions in the run: block into the step's env: map as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, and PR_HEAD_SHA. Updated all shell references to use the environment variables with proper quoting.
2. .github/workflows/release-bot.yml (script-injection): Moved steps.auth.outputs.credentials_file_path from the inline docker -v flag into the step's env: map as CREDENTIALS_FILE_PATH, then referenced it as "$CREDENTIALS_FILE_PATH" in the shell command.
3. .github/workflows/performance.yml (missing-permissions): Added top-level `permissions: contents: read` block.
4. .github/workflows/smoke.yml (missing-permissions): Added top-level `permissions: contents: read` block.
5. .github/workflows/secrets.yml (missing-permissions): Added top-level `permissions: contents: read` block.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced the three unquoted variable expansions (${BASE:-''}, ${HEAD:-''}, ${ARGS:-''}) in the docker run command with a bash array approach. A docker_args array is built conditionally: --since-commit "$BASE" is added only when BASE is non-empty, --branch "$HEAD" only when HEAD is non-empty, fixed flags are always added, and "$ARGS" is added only when non-empty. The array is then expanded as "${docker_args[@]}" in the docker run command. This ensures all user-controlled values are properly double-quoted (preventing shell metacharacter injection) while also correctly omitting optional flags and their values when the inputs are empty.

