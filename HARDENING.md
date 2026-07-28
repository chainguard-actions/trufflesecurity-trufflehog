<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.95.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.95.6** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple GitHub Actions expressions are directly interpolated inside the run: shell script in action.yml. The expressions ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{ github.event.pull_request.base.sha }}, and ${{ github.event.pull_request.head.sha }} are substituted directly into shell commands before the shell parses them, enabling script injection. For example: `HEAD=${{ github.event.after }}` and `if [ ${{ github.event.before }} == "0000..." ]`. Rule (b): The env vars ${VERSION} (from inputs.version), ${BASE:-''} (from inputs.base or github.event.*), ${HEAD:-''} (from inputs.head or github.event.*), and ${ARGS:-''} (from inputs.extra_args) are used unquoted in the docker run command, allowing shell metacharacter injection.

Locations:

- `action.yml:68`
- `action.yml:74`
- `action.yml:75`
- `action.yml:78`
- `action.yml:80`
- `action.yml:83`
- `action.yml:84`
- `action.yml:85`
- `action.yml:92`
- `action.yml:95`
- `action.yml:97`
- `action.yml:101`

### script-injection (severity: high)

Rule (a): In release-bot.yml, the expression ${{ steps.auth.outputs.credentials_file_path }} is directly interpolated inside a run: shell command: `-v ${{ steps.auth.outputs.credentials_file_path }}:/tmp/keys/GCP_SA_TRUFFLE_RELEASE_BOT.json:ro`. The steps.*.outputs.* context is listed as an untrusted-input source and flows through YAML template substitution before the shell parses it, enabling script injection.

Locations:

- `.github/workflows/release-bot.yml:33`

### missing-permissions (severity: medium)

The workflow file secrets.yml has no top-level permissions: key and the single job 'test' has no job-level permissions: key. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/secrets.yml:1`

### missing-permissions (severity: medium)

The workflow file smoke.yml has no top-level permissions: key and neither the 'smoke' nor 'zombies' jobs have job-level permissions: keys. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file performance.yml has no top-level permissions: key and the 'speed' job has no job-level permissions: key. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/performance.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, static-inline-injection

**Notes:**

Fixed all 6 findings:

1. action.yml script-injection (Rule a): Moved all ${{ github.* }} expressions (github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, github.event.pull_request.head.sha) into the step's env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, GH_PR_HEAD_SHA. Replaced direct interpolation in the shell script with references to these env vars (all properly double-quoted).

2. action.yml script-injection (Rule b): Fixed unquoted ${VERSION}, ${BASE:-''}, ${HEAD:-''}, ${ARGS:-''} in the docker run command by building a bash array (docker_args) that conditionally appends --since-commit, --branch, and extra args only when non-empty, keeping all values properly double-quoted.

3. release-bot.yml script-injection: Moved ${{ steps.auth.outputs.credentials_file_path }} into the env: block as CREDENTIALS_FILE_PATH and referenced it as "${CREDENTIALS_FILE_PATH}" in the shell command.

4. secrets.yml missing-permissions: Added top-level `permissions: contents: read`.

5. smoke.yml missing-permissions: Added top-level `permissions: contents: read`.

6. performance.yml missing-permissions: Added top-level `permissions: contents: read`.

