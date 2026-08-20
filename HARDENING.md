<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.95.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.95.3** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple GitHub Actions expressions are directly interpolated inside a `run:` shell command string in action.yml. Specifically: `${{ github.event_name }}`, `${{ github.event.after }}`, `${{ github.event.before }}`, `${{ github.event.pull_request.base.sha }}`, and `${{ github.event.pull_request.head.sha }}` are all expanded by the YAML template engine before the shell ever sees them, allowing an attacker to inject shell metacharacters via event payloads.

Locations:

- `action.yml:63`
- `action.yml:68`
- `action.yml:69`
- `action.yml:72`
- `action.yml:74`
- `action.yml:76`
- `action.yml:78`
- `action.yml:80`

### script-injection (severity: high)

Sub-rule (a): In release-bot.yml, the expression `${{ steps.auth.outputs.credentials_file_path }}` is directly interpolated inside a `run:` shell command string (used as a volume mount path in a `docker run` command). Any expression inside a `run:` block is a script-injection risk regardless of the context it reads from.

Locations:

- `.github/workflows/release-bot.yml:30`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or branch names instead of full 40-character SHA digests. Failing references include: actions/checkout@v6, actions/setup-go@v6, github/codeql-action/init@v4, github/codeql-action/analyze@v4 (codeql-analysis.yml); actions/checkout@v6, actions/setup-go@v6, jaxxstorm/action-install-gh-release@v3.0.0, rwx-research/setup-captain@v1 (detector-tests.yml); actions/checkout@v6, actions/setup-go@v6, golangci/golangci-lint-action@v9 (lint.yml); actions/checkout@v6, actions/setup-go@v6 (performance.yml); google-github-actions/auth@v3, docker/login-action@v4 (release-bot.yml); actions/checkout@v6, docker/setup-qemu-action@v4, docker/login-action@v4, actions/setup-go@v6, goreleaser/goreleaser-action@v7 (release.yml); actions/checkout@v6 (secrets.yml); actions/checkout@v6, actions/setup-go@v6 (smoke.yml); actions/checkout@v6, actions/setup-go@v6, google-github-actions/auth@v3, buildpulse/buildpulse-action@main, mikepenz/action-junit-report@v6 (test.yml).

Locations:

- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/detector-tests.yml:14`
- `.github/workflows/lint.yml:18`
- `.github/workflows/performance.yml:13`
- `.github/workflows/release-bot.yml:16`
- `.github/workflows/release.yml:20`
- `.github/workflows/secrets.yml:14`
- `.github/workflows/smoke.yml:8`
- `.github/workflows/test.yml:18`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks, meaning they run with the default (potentially broad) token permissions: performance.yml, smoke.yml, and secrets.yml.

Locations:

- `.github/workflows/performance.yml:1`
- `.github/workflows/smoke.yml:1`
- `.github/workflows/secrets.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings:
1. script-injection (action.yml): Moved github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, and github.event.pull_request.head.sha from run: blocks into env: block as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA. Shell script updated to use plain env vars.
2. script-injection (release-bot.yml): Moved steps.auth.outputs.credentials_file_path from run: block into env: block as CREDENTIALS_FILE_PATH.
3. unpinned-uses: Pinned all action references to full SHA digests across codeql-analysis.yml, detector-tests.yml, lint.yml, performance.yml, release-bot.yml, release.yml, secrets.yml, smoke.yml, and test.yml.
4. missing-permissions: Added 'permissions: contents: read' to performance.yml, smoke.yml, and secrets.yml.
5. static-inline-injection: Covered by the same fix as script-injection for action.yml (github.event.pull_request.head.sha moved to env: block).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script injection issues in action.yml:
1. Quoted `$COMMIT_LENGTH` in the if condition: `[ "$COMMIT_LENGTH" == "0" ]`
2. Quoted `${VERSION}` in the docker image reference: `"ghcr.io/trufflesecurity/trufflehog:${VERSION}"`
3. Replaced `${BASE:-''}` with `${BASE:+"$BASE"}` for the optional --since-commit argument
4. Replaced `${HEAD:-''}` with `${HEAD:+"$HEAD"}` for the optional --branch argument
5. Replaced `${ARGS:-''}` with xargs-based tokenization into an `extra_args` array (using the guarded `if [ -n "$ARGS" ]` form + `printf '%s' "$ARGS" | xargs printf '%s\0'` NUL-delimited read loop), then expanded as `"${extra_args[@]}"`

