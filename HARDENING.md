<!-- markdownlint-disable -->

# Hardening Report: cardinalby--git-get-release-action/1.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cardinalby--git-get-release-action/1.2.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks. Unpinned references: `actions/checkout@v2`, `actions/setup-node@v1`, `ad-m/github-push-action@master` in build-pack.yml; `actions/checkout@v2` in functional-test.yml; `actions/checkout@v2` and `actions/setup-node@v1` in integration-test.yml.

Locations:

- `.github/workflows/build-pack.yml:12`
- `.github/workflows/build-pack.yml:18`
- `.github/workflows/build-pack.yml:35`
- `.github/workflows/functional-test.yml:14`
- `.github/workflows/integration-test.yml:10`
- `.github/workflows/integration-test.yml:12`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files — build-pack.yml, functional-test.yml, and integration-test.yml — are affected.

Locations:

- `.github/workflows/build-pack.yml:1`
- `.github/workflows/functional-test.yml:1`
- `.github/workflows/integration-test.yml:1`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing template-substituted values to be interpreted by the shell before any quoting takes effect.

1. In build-pack.yml, `${{ env.PACKED_JS_PATH }}` is interpolated directly in three `run:` lines: `echo ::set-output name=changes::$(git status ${{ env.PACKED_JS_PATH }} --porcelain)`, `git add ${{ env.PACKED_JS_PATH }}`, and `git commit -m "Pack with dependencies to ${{ env.PACKED_JS_PATH }}"`.

2. In functional-test.yml, `${{ steps.getByCommitSha.outputs.tag_name }}` and `${{ toJSON(steps.getByCommitSha.outputs.body) }}` are interpolated directly in a `run:` block: `echo "${{ steps.getByCommitSha.outputs.tag_name }}" &&` and `echo "${{ toJSON(steps.getByCommitSha.outputs.body) }}" &&`. Step outputs are workflow-controllable and can contain shell metacharacters.

Locations:

- `.github/workflows/build-pack.yml:27`
- `.github/workflows/build-pack.yml:31`
- `.github/workflows/build-pack.yml:32`
- `.github/workflows/functional-test.yml:72`
- `.github/workflows/functional-test.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across build-pack.yml, functional-test.yml, and integration-test.yml:

1. unpinned-uses: Pinned all 6 action references to full commit SHAs with tag comments preserved:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e (all 3 files)
   - actions/setup-node@v1 → @f1f314fca9dfce2769ece7d933488f076716723e (build-pack.yml, integration-test.yml)
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c (build-pack.yml)

2. permissions: Added top-level permissions blocks:
   - build-pack.yml: contents: write (required for git push)
   - functional-test.yml: contents: read
   - integration-test.yml: contents: read

3. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks:
   - build-pack.yml: env.PACKED_JS_PATH moved to env: in two steps, referenced as $PACKED_JS_PATH
   - functional-test.yml: steps.getByCommitSha.outputs.tag_name and toJSON(steps.getByCommitSha.outputs.body) moved to env: as TAG_NAME and BODY_JSON

