---
name: dependency-upgrade-review
description: Review a Dependabot or other dependency-bump PR — identify the changed packages, research their changelogs, assess breaking-change and codebase impact, and post the analysis as a PR comment. Use this whenever someone asks to review, analyze, or evaluate a dependency-upgrade PR.
argument-hint: <PR number, URL, or owner/repo#number>
---

# Dependency Upgrade Review

Analyze a PR that bumps one or more dependencies and post a single, structured review comment with actionable insights.

## Input

`$ARGUMENTS` — the PR to evaluate. Examples:

- `/dependency-upgrade-review 12345`
- `/dependency-upgrade-review https://github.com/dimagi/commcare-hq/pull/12345`
- `/dependency-upgrade-review dimagi/commcare-hq#12345`

If given a bare number, use the current repo (`gh repo view --json nameWithOwner`) unless that doesn't look right for a dependency-bump PR — in that case ask which repo.

## Analysis Process

CommCare HQ uses `uv` for Python dependencies and Yarn for JavaScript. The codebase lives primarily under `corehq/`. Dependabot there is configured for major-version bumps on Python/npm and weekly bumps on GitHub Actions — but this skill works for any dependency-bump PR, not just CommCare HQ's.

**IMPORTANT**: Do NOT mention security vulnerabilities, CVEs, or security advisories in the comment. Security updates are handled through a separate internal process. If the PR turns out to be a security-related upgrade, focus only on breaking changes, migration notes, and codebase impact — omit any security details.

Follow `${CLAUDE_PLUGIN_ROOT}/references/dependency-analysis.md` for the shared research methodology.

### 1. Identify Changed Dependencies

- Run `gh pr diff <PR>` to see what changed.
- Python: parse changes in `pyproject.toml` and/or `uv.lock` or `requirements.txt`, etc., whatever the repo uses.
- JavaScript: parse changes in `package.json` and/or `yarn.lock`.
- GitHub Actions: parse changes in `.github/workflows/*.yml`.
- List each package with old → new version.

### 2. Changelog Research and Impact Assessment

For each updated dependency, apply the reference doc's **Changelog Research** and **Codebase Impact Analysis** sections. Use WebFetch for changelog/release notes lookups. Categorize each notable change per the reference doc's **Categorizing Risk** section, and note whether the repo uses the affected APIs.

## Output Format

Draft a single comment structured as:

```markdown
### 🔍 Dependency Analysis Summary
- One- or two-sentence narrative of what this PR changes and why it matters
  (don't re-list packages and versions if the PR body already has that)
- Overall risk assessment: **LOW** / **MEDIUM** / **HIGH**

### 📋 Detailed Changelog Review
For each updated dependency, use the package name as a subheading and cover:
- **Changes**: Summary of key changes between versions
- **Breaking Changes**: Any breaking changes (or "None")
- **Migration Notes**: Required upgrade steps (or "None")

### ⚠️ Impact Assessment
- **Breaking Changes Found**: Yes/No with details
- **Affected Files**: Repo files that may need updates
- **Test Impact**: Any tests that may need updating
- **Configuration Changes**: Required config updates (or "None")

### 🛠️ Recommendations
- **Action Required**: What the team should do before merging
- **Testing Focus**: Areas to test thoroughly
- **Follow-up Tasks**: Any additional work needed after merge
- **Merge Recommendation**: APPROVE / REVIEW_NEEDED / HOLD

### 📚 Useful Links
- Links to relevant changelogs, migration guides, documentation
```

Be thorough but concise. Focus on actionable insights.

## Posting the Comment

Do not post automatically. Show the user the drafted comment in full and ask them to confirm before posting. Only after they explicitly approve, post it with:

```bash
gh pr comment <PR> --body "<drafted comment>"
```

If the user asks for changes, revise and re-confirm before posting.
