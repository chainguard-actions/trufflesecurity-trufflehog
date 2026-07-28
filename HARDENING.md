<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.95.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.95.5** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple GitHub context expressions are directly interpolated inside the run: shell script in action.yml without going through an env: variable. Offending lines include: `if [ "${{ github.event_name }}" == "push" ]`, `HEAD=${{ github.event.after }}`, `if [ ${{ github.event.before }} == "000..." ]`, `BASE=${{ github.event.before }}`, `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]`, `elif [ "${{ github.event_name }}" == "pull_request" ]`, `BASE=${{github.event.pull_request.base.sha}}`, and `HEAD=${{github.event.pull_request.head.sha}}`. Rule (b): `${ARGS:-''}` is expanded unquoted in the docker run command while ARGS is sourced from inputs.extra_args, allowing shell metacharacter injection.

Locations:

- `action.yml:73`
- `action.yml:79`
- `action.yml:80`
- `action.yml:83`
- `action.yml:85`
- `action.yml:88`
- `action.yml:89`
- `action.yml:90`

### script-injection (severity: high)

Rule (a): The run: block in the 'Run release bot' step directly interpolates `${{ steps.auth.outputs.credentials_file_path }}` inside a shell command string: `-v ${{ steps.auth.outputs.credentials_file_path }}:/tmp/keys/GCP_SA_TRUFFLE_RELEASE_BOT.json:ro`. The steps.*.outputs.* context flows through YAML template substitution before the shell sees it, enabling script injection.

Locations:

- `.github/workflows/release-bot.yml:34`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and the job 'speed' has no job-level permissions: key either. Without explicit permissions, the job inherits the default repository token permissions which may be overly broad.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and neither job ('smoke', 'zombies') has a job-level permissions: key. Without explicit permissions, jobs inherit the default repository token permissions which may be overly broad.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and the job 'test' has no job-level permissions: key. Without explicit permissions, the job inherits the default repository token permissions which may be overly broad.

Locations:

- `.github/workflows/secrets.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, static-inline-injection

**Notes:**

Fixed all 6 findings:

1. action.yml script-injection: Moved all GitHub context expressions (github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, github.event.pull_request.head.sha) into the step's env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, GH_PR_HEAD_SHA. Shell script now references these as plain env vars. Also replaced the unsafe unquoted ${ARGS:-''} docker run expansion with a safe bash array (IFS read -r -a extra_args) and replaced ${BASE:-''}/--since-commit and ${HEAD:-''}/--branch with conditional array building (docker_args) to avoid passing empty arguments.

2. release-bot.yml script-injection: Moved ${{ steps.auth.outputs.credentials_file_path }} into env: as CREDENTIALS_FILE_PATH and referenced it as "${CREDENTIALS_FILE_PATH}" in the docker -v flag.

3. performance.yml missing-permissions: Added top-level `permissions: contents: read`.

4. smoke.yml missing-permissions: Added top-level `permissions: contents: read`.

5. secrets.yml missing-permissions: Added top-level `permissions: contents: read`.

