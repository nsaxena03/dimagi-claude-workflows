---
name: dependency-upgrade
description: Upgrade a Python or JS dependency safely. Given a library name, finds the latest safe version, reviews the changelog, assesses impact based on how the library is used in the repo, performs the upgrade, and creates a PR with the changelog and assessment. Use this whenever someone asks to upgrade, bump, or update a package/library/dependency.
argument-hint: <library name>
---

# Dependency Upgrade

Upgrade a dependency to the latest safe version with a well-documented PR. The goal is to make dependency upgrades low-risk and easy to review — the PR should give reviewers everything they need to feel confident merging.

## Input

`$ARGUMENTS` — The library/package name. Examples:
- `/dependency-upgrade pillow`
- `/dependency-upgrade django-celery-results`
- `/dependency-upgrade markdown-it`

## Step 1: Detect Ecosystem and Find Current Version

Determine whether this is a Python or JS dependency by searching for it in the repo's dependency files:

**Python** — search `requirements/*.txt`, `requirements.txt`, `setup.py`, `setup.cfg`, `pyproject.toml`
**JS** — search `package.json`, `package-lock.json`, `yarn.lock`

If the library appears in both ecosystems, ask the user which one they mean. If it doesn't appear anywhere, tell the user — the library might not be a direct dependency, or might be spelled differently.

Record the **current pinned version**.

## Step 2: Find the Latest Safe Version

### Python

Fetch version and release date info from the PyPI JSON API:

```bash
curl -s https://pypi.org/pypi/<package>/json
```

From the response:
- `info.version` is the latest release
- `releases` is a dict keyed by version string, each containing a list of file objects with `upload_time_iso_8601`

### JS

```bash
npm view <package> versions --json
npm view <package> time --json
```

The `time` object maps version strings to ISO date strings.

### Version Safety Rules

The point of these rules is to avoid being an early adopter of a release that might have undiscovered issues — we'd rather let the broader community shake out problems first.

1. **Skip X.0.0 major releases.** A brand-new major version (e.g., 3.0.0) often has rough edges, breaking changes that aren't fully documented, and ecosystem incompatibilities. Wait for at least X.0.1 or X.1.0 before considering it. If the only available upgrade is an X.0.0, tell the user and suggest waiting.

2. **Skip releases less than 7 days old** — except for patch releases (X.Y.Z where only Z changed from the current version). Patch releases are typically bug/security fixes and are safe to adopt immediately. For minor and major releases, a week gives the community time to surface issues.

3. **Pick the best candidate.** Walk backwards from the latest version until you find one that passes both rules above. This is the **target version**.

If no version passes the safety rules, explain why and ask the user if they want to proceed anyway.

## Step 3: Look Up the Changelog and Assess Impact

Follow the **Changelog Research**, **Codebase Impact Analysis**, and **Categorizing Risk** sections of `${CLAUDE_PLUGIN_ROOT}/references/dependency-analysis.md` to research what changed between the current and target versions and how it affects this repo. Also follow that doc's **Writing About Upstream References** section when summarizing the changelog in the PR body — the PR will be posted to this repo, so bare `#<number>` references to the upstream repo need the same care.

Then **write a short assessment** (3-5 sentences) for the PR body. Cover:
- How widely the library is used in the repo (a few files vs. everywhere)
- Whether any breaking changes or deprecations affect the repo's usage
- The overall risk level from the reference doc's Categorizing Risk section

If the risk is high, flag it to the user before proceeding. They may want to handle it manually.

If no changelog is found, note this — it's still worth proceeding, but flag it in the PR.

## Step 4: Perform the Upgrade

1. **Create a branch:** `dependency-upgrade/<package>-<target_version>`

2. **Update the version pin** in the appropriate file(s). For Python requirements files, match the existing pin style (e.g., `==`, `>=`, `~=`). For package.json, match the existing prefix (`^`, `~`, exact).

3. **Update lock files** if they exist:
   - Python: run `uv pip compile` to regenerate lock files (this is the team's default). If `uv.lock` or `pyproject.toml` with `[tool.uv]` is present, use `uv lock` instead. Fall back to `pip-compile` only if there's no sign of uv in the project.
   - JS: `npm install` or `yarn install`

4. **Don't run tests** — leave that to CI. Just make sure the install/compile step succeeded without errors.

## Step 5: Commit and Create PR

Commit the changes and open a PR. The PR description is the main deliverable — it should give reviewers everything they need.

### Commit message

```
Upgrade <package> from <current> to <target>
```

### PR title

```
Upgrade <package> from <current> to <target>
```

### PR body

Structure the PR description like this:

```markdown
## Summary

Upgrades `<package>` from `<current_version>` to `<target_version>`.

## Risk Assessment

<the 3-5 sentence assessment from Step 3>

## Changelog

<summary of notable changes — breaking changes, deprecations, new features, security fixes>
<link to full upstream changelog or GitHub Releases page>

## Usage in this repo

<summary of how the library is used — number of files, main patterns>
```

Keep it concise but complete. Reviewers should be able to approve based on the PR description alone without having to go read the upstream changelog themselves.

## Edge Cases

- **Library not found in repo:** Tell the user. Don't guess.
- **Already on latest safe version:** Tell the user, no action needed.
- **Multiple requirements files pin different versions:** Flag the inconsistency and ask the user which to update.
- **No changelog found anywhere:** Proceed but note it prominently in the PR. Link to the GitHub compare view (`/compare/v<current>...v<target>`) as a fallback so reviewers can see the raw diff.
- **Target version has known issues:** If you find GitHub issues or advisories mentioning problems with the target version, mention them in the risk assessment.
