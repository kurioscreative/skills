---
name: gem-release
description: Prepare a Ruby gem release. Use when bumping gem versions or preparing changelogs. Handles version files, lockfile, changelog, and git commit. Publishing is done via the Push Gem GitHub Action (workflow_dispatch).
argument-hint: "[version, e.g. 0.2.0 or major/minor/patch]"
allowed-tools: Read, Grep, Glob, Edit, Bash(ls *), Bash(grep *), Bash(bundle *), Bash(git status *), Bash(git diff *), Bash(git log *), Bash(git add *), Bash(git commit *), Bash(head *)
---

# /gem-release - Ruby Gem Release

Prepare a versioned release of a Ruby gem: bump version, update changelog, and commit. Tagging and publishing are handled by the Push Gem GitHub Action (`rubygems/release-gem@v1` via `workflow_dispatch`).

## Current State

Gemspec: !`ls *.gemspec 2>/dev/null`
Current version: !`grep -r 'VERSION' lib/*/version.rb 2>/dev/null || grep -r 'VERSION' lib/version.rb 2>/dev/null`
Latest tag: !`git tag --sort=-v:refname | head -1 2>/dev/null`
Unreleased changes (since latest tag): Fetch the latest tag from above and run `git log <tag>..HEAD --oneline` during Step 1.
Changelog unreleased section: !`sed -n '/## \[Unreleased\]/,/## \[/p' CHANGELOG.md 2>/dev/null | head -20`
Push workflow: !`ls .github/workflows/push.yml 2>/dev/null && echo "found" || echo "not found"`
Git status: !`git status --short 2>/dev/null | head -10`

---

## Workflow

### Step 1: Determine Target Version

**If `$ARGUMENTS` is a version number** (e.g. `0.2.0`): Use it directly.

**If `$ARGUMENTS` is a bump level** (`major`, `minor`, `patch`): Calculate from current version.

**If no argument:** Look at the changes since the last tag and recommend a version based on semver:

- **patch** — bug fixes, documentation, internal refactors
- **minor** — new features, non-breaking API additions
- **major** — breaking changes to public API

Present the recommendation and wait for confirmation.

### Step 2: Preflight Checks

Before changing anything, verify:

1. **Working tree is clean** — no uncommitted changes (staged or unstaged). If dirty, stop and ask the user to commit or stash first.
2. **Tests pass** — run the test suite. If tests fail, stop.

**Why preflight matters:** The release commit should contain _only_ version/changelog changes. Mixed commits make rollbacks painful and git bisect useless.

### Step 3: Audit Artifacts

Before updating version files, check that release artifacts are ready. **Fix** mechanical issues directly. **Flag** content issues and ask the user.

**Fix directly:**

- **CHANGELOG.md missing `[Unreleased]` heading** — add one with empty sections
- **Empty changelog subsections** (e.g. `### Changed` with nothing under it) — remove empty subsections from the unreleased block before converting it to the release heading

**Flag and ask:**

- **Changelog unreleased section is empty** — no entries under `[Unreleased]`. Every release should document what changed. List the commits since last tag and ask the user which entries to add.
- **Commits not reflected in changelog** — compare `git log <tag>..HEAD --oneline` against changelog entries. If significant commits are missing, list them and ask the user whether to add entries.
- **README.md references a specific version** — search for old version string in README (install instructions, badges, etc.). Flag any matches so the user can decide whether to update.

**Why audit matters:** A release with an empty or stale changelog erodes trust with users. Catching this before the commit avoids a "fix changelog" follow-up commit that muddies the release.

### Step 4: Update Version Files

Three files change in a release:

1. **`lib/**/version.rb`** — bump the `VERSION` constant
2. **`CHANGELOG.md`** — replace `Unreleased` with today's date on the version heading
3. **`Gemfile.lock`** — run `bundle install` to regenerate with new version

**Why these three together:** They are a single logical unit. The version constant is what the gemspec reads. The lockfile must match. The changelog date marks when the version shipped.

### Step 5: Commit

```
git add lib/**/version.rb CHANGELOG.md Gemfile.lock
git commit -m "Release v$VERSION"
```

**Why one commit:** A release is an atomic event. Splitting version bump and changelog into separate commits creates intermediate states.

**Do not tag.** The `rubygems/release-gem@v1` GitHub Action creates the `v$VERSION` tag when publishing. Tagging locally would conflict.

### Step 6: Present Summary

Show what was done and what the user needs to do next:

```markdown
## Release v$VERSION ready

### Changed files

- lib/.../version.rb: $OLD_VERSION -> $VERSION
- CHANGELOG.md: dated $TODAY
- Gemfile.lock: regenerated

### To publish

1. `git push` — push the release commit to main
2. Go to **Actions → Push Gem → Run workflow** on GitHub
```

**Never push or publish.** The skill stops at the commit. Pushing to remote and triggering the GitHub Action are the user's responsibility.

---

## Post-Release

After a release, remind the user to add a new Unreleased section to CHANGELOG.md for future work:

```markdown
## [Unreleased]

### Added

### Changed

### Fixed
```

This goes above the just-released version heading.
