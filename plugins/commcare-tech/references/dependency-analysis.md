# Dependency Changelog & Impact Analysis

Methodology for researching a dependency version bump and judging how much it
could affect a codebase. This is a reference document used by other skills. Each
skill supplies its own version resolution and its own output format — this
covers the research steps they share.

## Changelog Research

Find the changelog between the current version and the target version. Check
these sources in order:

1. **PyPI/npm metadata** — look for `project_urls` (PyPI) or
   `repository`/`homepage` (npm) to find the GitHub repo.
2. **GitHub Releases** — `gh api repos/<owner>/<repo>/releases` — look for
   releases tagged between the current and target versions.
3. **CHANGELOG file** — check the repo root for `CHANGELOG.md`, `CHANGES.md`,
   `CHANGES.rst`, `HISTORY.md`, `NEWS.md`, or similar. Also check a `docs/`
   directory.

For GitHub Actions, the same idea applies to the action's own repo — check its
releases and `.github/workflows` for changed inputs/outputs or runner
requirements.

Summarize the notable changes between the current and target versions — breaking
changes, deprecations, new features, and security fixes. Don't paste the full
changelog verbatim; link to the upstream changelog or GitHub Releases page so
readers can dig deeper if they want.

If no changelog is found anywhere, note this rather than silently skipping it —
flag it prominently wherever the analysis is published.

## Codebase Impact Analysis

Search the codebase to understand how the dependency is actually used. This is
the most important step for reader confidence — the goal is to answer "could
this break anything *here*?", not just to restate the upstream changelog.

1. **Find all imports and usages.**
   - Python: grep for `import <package>` and `from <package>`.
   - JS: grep for `require('<package>')` and `import ... from '<package>'`.
   - GitHub Actions: check which workflow files reference the updated action.

   Count the number of files and note the main usage patterns.

2. **Cross-reference with the changelog.** Look for breaking changes,
   deprecations, or API changes that touch functionality the repo actually uses.
   This is where the value is — not just listing what changed, but whether those
   changes affect *this codebase*.

## Categorizing Risk

Two levels of categorization, used together:

- **Per-change**: label each notable changelog entry BREAKING, MAJOR, MINOR,
  or PATCH.
- **Overall**: roll it up into one risk level —
  - **LOW** — patch/minor with no breaking changes in APIs the repo uses
  - **MEDIUM** — minor version with some changes relevant to the repo's usage
  - **HIGH** — major version, or breaking changes in referenced APIs

## Writing About Upstream References

**IMPORTANT**: When summarizing changelogs from upstream repos, never copy bare
`#<number>` PR/issue references into a comment or PR body that will be posted to
a different repo — GitHub auto-links them against *that* repo and surfaces
unrelated PRs/issues. Either drop the reference entirely or rewrite it as a
fully-qualified `owner/repo#<number>` (e.g. `ljharb/qs#555`) so GitHub resolves
it to the upstream repo. The same applies to bare `GH-<number>` shorthands and
to autolinked commit SHAs — qualify them or omit them.
