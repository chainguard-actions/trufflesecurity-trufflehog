<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog--/v3.95.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **trufflesecurity--trufflehog--/v3.95.8** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple GitHub Actions expressions are interpolated directly inside a run: shell command string in action.yml. The expressions ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{github.event.pull_request.base.sha}}, and ${{github.event.pull_request.head.sha}} are all expanded by the YAML template engine before the shell sees them, enabling script injection. Offending lines include: `if [ "${{ github.event_name }}" == "push" ]`, `HEAD=${{ github.event.after }}`, `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]`, `BASE=${{ github.event.before }}`, `BASE=${{github.event.pull_request.base.sha}}`, `HEAD=${{github.event.pull_request.head.sha}}`.

Locations:

- `action.yml:68`
- `action.yml:73`
- `action.yml:74`
- `action.yml:77`
- `action.yml:79`
- `action.yml:82`
- `action.yml:83`
- `action.yml:84`

### script-injection (severity: high)

Sub-rule (a): The expression ${{ steps.auth.outputs.credentials_file_path }} is interpolated directly inside a run: shell command string in release-bot.yml. The offending line is: `-v ${{ steps.auth.outputs.credentials_file_path }}:/tmp/keys/GCP_SA_TRUFFLE_RELEASE_BOT.json:ro`. This allows the steps output value to be injected into the shell command before quoting.

Locations:

- `.github/workflows/release-bot.yml:33`

### missing-permissions (severity: medium)

The workflow file performance.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. This means the workflow runs with default (potentially broad) GitHub token permissions.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

The workflow file smoke.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. This means the workflow runs with default (potentially broad) GitHub token permissions.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file secrets.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. This means the workflow runs with default (potentially broad) GitHub token permissions.

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

Fixed script injection in action.yml by moving ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{ github.event.pull_request.base.sha }}, and ${{ github.event.pull_request.head.sha }} from the run: block into the env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, and GH_PR_HEAD_SHA respectively. Fixed script injection in release-bot.yml by moving ${{ steps.auth.outputs.credentials_file_path }} into the env: block as CREDENTIALS_FILE_PATH. Added 'permissions: contents: read' to performance.yml, smoke.yml, and secrets.yml to address missing-permissions findings.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the docker run command in action.yml:
1. Quoted the image reference: `"ghcr.io/trufflesecurity/trufflehog:${VERSION}"` to prevent injection via the `version` input.
2. Replaced unquoted `${BASE:-''}` (with `--since-commit` flag) with a bash array pattern: `[ -n "$BASE" ] && docker_args+=(--since-commit "$BASE")` — only adds the flag+value when BASE is non-empty, with the value properly double-quoted.
3. Replaced unquoted `${HEAD:-''}` (with `--branch` flag) with the same bash array pattern: `[ -n "$HEAD" ] && docker_args+=(--branch "$HEAD")`.
4. Replaced unquoted `${ARGS:-''}` with a bash array: `IFS=' ' read -ra extra_args <<< "$ARGS"` expanded as `"${extra_args[@]}"`, keeping each argument as a separate properly-quoted shell word.
All four locations from the finding are now safe from shell metacharacter injection.

