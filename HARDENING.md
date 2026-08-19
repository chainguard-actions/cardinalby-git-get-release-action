<!-- markdownlint-disable -->

# Hardening Report: cardinalby--git-get-release-action/1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cardinalby--git-get-release-action/1.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three uses: references in the workflow are pinned to mutable tags or branch names rather than immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those refs are moved:
- `actions/checkout@v2` (line 17)
- `actions/setup-node@v1` (line 22)
- `ad-m/github-push-action@master` (line 44) — pinned to a branch, which is especially risky

Locations:

- `.github/workflows/node.js.yml:17`
- `.github/workflows/node.js.yml:22`
- `.github/workflows/node.js.yml:44`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ env.PACKED_JS_PATH }} into shell command strings. The env.* context is workflow-controllable and any ${{ ... }} expression inside a run: block is a script-injection risk (sub-rule a). An attacker who can influence the env context could inject arbitrary shell commands.
- Line 33: `run: echo ::set-output name=changes::$(git status ${{ env.PACKED_JS_PATH }} --porcelain)`
- Line 37: `git add ${{ env.PACKED_JS_PATH }}`
- Line 38: `git commit -m "Pack with dependencies to ${{ env.PACKED_JS_PATH }}"`
Fix: replace the expression with the env var reference `$PACKED_JS_PATH` (already defined in the job-level env: block).

Locations:

- `.github/workflows/node.js.yml:33`
- `.github/workflows/node.js.yml:37`
- `.github/workflows/node.js.yml:38`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and the single job (build) also has no job-level permissions: key. Without explicit permissions, the workflow inherits the default repository permissions (which can be write-all in many org configurations), granting the GITHUB_TOKEN broader access than necessary.

Locations:

- `.github/workflows/node.js.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/node.js.yml: (1) Pinned actions/checkout@v2, actions/setup-node@v1, and ad-m/github-push-action@master to their full 40-character SHA digests with original refs preserved as comments. (2) Replaced all three ${{ env.PACKED_JS_PATH }} interpolations in run: blocks with the plain env var reference $PACKED_JS_PATH (already defined in the job-level env: block). (3) Added a top-level permissions: block with contents: write — the minimum needed for the workflow to commit and push the packed JS file.

