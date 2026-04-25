---
name: case-study-research
description: "Collect raw artifacts from a completed feature's development for case study research. Use when the user wants to document how a feature was built, gather development history for a write-up, create a case study of AI-assisted development, or retrospectively analyze a feature's journey from idea to production. Triggers on phrases like 'case study', 'document how we built', 'collect the history of', 'research folder', or 'development retrospective'."
argument-hint: "<feature branch, PR number, or description>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList
---

# /case-study — Feature Development Research Collection

Systematically collect all raw artifacts from a feature's development lifecycle
(idea → implementation → review → merge) into an organized research folder.

> The goal is completeness first, curation second. Collect everything now so
> the narrative can be found later without re-gathering.

## What This Skill Produces

```
docs/case-studies/<feature-name>/
├── MANIFEST.md              — What's here and how to use it
├── timeline.md              — Chronological interleaving of sessions + commits + decisions
├── raw/
│   ├── git-log-full.txt     — Full git log with stats
│   ├── git-commits-timeline.md — Commit messages chronologically
│   ├── pr-<N>.json          — PR metadata
│   ├── pr-<N>-body.md       — PR description
│   ├── pr-diffstat.txt      — Files changed summary
│   ├── *.txt                — Session transcript exports
│   ├── project-docs/        — Pre-implementation planning docs
│   └── feature-docs/        — Final feature documentation
├── sessions/
│   ├── SESSION-INDEX.md      — Every session with timestamps, message counts, first prompts
│   └── *.jsonl               — Raw Claude Code session logs
```

## Current Context

Git branch:
!`git branch --show-current`

Recent commits:
!`git log --oneline -5`

---

## Phase 1: Identify the Feature

Determine what feature to collect research for. The user might provide:

- A branch name (e.g., `worktree-pairwise-church-recommender`)
- A PR number (e.g., `#1524`)
- A feature description (search git log for it)

From whatever input, resolve these anchors:

1. **Merge commit** or branch tip — the commit(s) that contain the feature
2. **Base branch** — what it branched from (usually `master` or `main`)
3. **PR number** — if one exists
4. **Date range** — first commit to merge date

```bash
# If given a PR number:
gh pr view <N> --json title,body,createdAt,mergedAt,headRefName,baseRefName,additions,deletions,files

# If given a branch:
git log --oneline <branch> --not <base> | tail -1  # first commit
git log --oneline <branch> --not <base> | head -1   # last commit
```

If the feature is ambiguous, use AskUserQuestion to clarify.

## Phase 2: Discover Artifact Sources

There are typically 5 categories of artifacts. Check for each:

### 2a. Git History

Always available. Extract:

- Commit count, date range, lines changed
- Full commit log with stats and messages

### 2b. PR Data

If a PR exists, export via `gh`:

```bash
gh pr view <N> --json title,body,comments,reviews,createdAt,mergedAt,additions,deletions,files,labels
gh api repos/<owner>/<repo>/pulls/<N>/comments  # review comments
```

### 2c. Claude Code Session Logs

Session logs live in `~/.claude/projects/` in directories derived from the working directory path (slashes replaced with dashes). A feature developed in a worktree will have its own project directory.

Discovery steps:

```bash
# List project directories that might contain relevant sessions
ls ~/.claude/projects/ | grep -i <project-name-fragment>
```

Look for:

- **Worktree-specific directory**: `~/.claude/projects/...-worktrees-<feature-name>/`
- **Main project directory**: `~/.claude/projects/...-<project-name>/`

For the main project directory, sessions are numerous — filter by searching JSONL content for feature-related keywords:

```python
# Check if a session mentions the feature
for line in open(jsonl_path):
    msg = json.loads(line)
    content = str(msg.get('message', {}).get('content', ''))
    if any(keyword in content.lower() for keyword in feature_keywords):
        # This session is relevant
```

Session JSONL format:

- `type: "user"` — human messages (not "human")
- `type: "assistant"` — Claude responses
- `type: "progress"` — session metadata (first line, contains `gitBranch`, `sessionId`)
- `timestamp` field on most message types

### 2d. Transcript Exports (.txt files)

Users sometimes export session transcripts as .txt files in the repo root or project docs. Look for:

```bash
ls *.txt  # in repo root
```

Match by filename (often contains the first prompt words) or by date.

### 2e. Planning & Feature Docs

Check for:

- `docs/projects/<feature-name>/` — pre-implementation planning
- `docs/features/<feature-name>.md` — final feature documentation
- Task files, playbooks, specs in project directories
- Memory files in `~/.claude/projects/.../memory/` that reference the feature

### 2f. Ask the User

After discovering what you can find automatically, present the inventory and ask:

- "Are there additional artifacts I'm missing?" (other .txt exports, external docs, Figma links, etc.)
- "Any of these NOT relevant?" (filter out noise)

## Phase 3: Collect

Create the output directory structure and copy/export all artifacts.

```bash
mkdir -p docs/case-studies/<feature-name>/{raw/project-docs,raw/feature-docs,sessions}
```

Collection checklist:

- [ ] Copy session JSONL files to `sessions/`
- [ ] Copy transcript .txt files to `raw/`
- [ ] Copy planning docs to `raw/project-docs/`
- [ ] Copy feature docs to `raw/feature-docs/`
- [ ] Copy relevant memory files to `raw/`
- [ ] Export git log: `git log --format="%H %ai %an%n  %s%n  %b" --stat <commits> > raw/git-log-full.txt`
- [ ] Export commit timeline: `git log --format="## %ai — %s%n%nCommit: %H%n%n%b%n---" --reverse <commits> > raw/git-commits-timeline.md`
- [ ] Export PR data as JSON and body as markdown
- [ ] Export diffstat: `git diff <base>..<tip> --stat > raw/pr-diffstat.txt`

For git log ranges, use the merge commit to find the right range:

```bash
# If merged via PR merge commit:
git log <merge-commit>^2 --not $(git merge-base <merge-commit>^ <merge-commit>^2)
```

Parallelize independent copy operations for speed.

## Phase 4: Extract & Index

Build the navigational aids that make the raw data usable.

### 4a. Session Index

Parse each JSONL file to extract:

- Session ID, source (worktree vs main-project)
- First/last timestamps, duration
- User message count, assistant message count
- First user prompt text (skip system tags starting with `<`)

Write to `sessions/SESSION-INDEX.md` sorted chronologically.

### 4b. Timeline

Cross-reference sessions with git commits to build a chronological timeline.

The timeline should identify:

1. **Natural phases** — group by development rhythm (ideation, implementation, iteration, review, merge)
2. **Parallel workstreams** — if multiple concerns were developed simultaneously (e.g., data layer, UI, product design)
3. **Key HITL decision points** — moments where the human redirected, reverted, questioned, or confirmed direction
4. **Session-to-commit mapping** — which sessions produced which commits

Spawn a subagent (model: sonnet) to build the timeline from:

- The git commits timeline file
- The session index data
- Transcript filenames (often reveal their topic)

### 4c. Manifest

Document every artifact in the research folder with:

- File counts and sizes per category
- JSONL format explanation
- How to use the research folder for writing the case study
- Entry points for different research goals

Spawn a subagent (model: sonnet) to build the manifest.

## Phase 5: Verify

Run acceptance checks:

```bash
# Structure exists
test -d docs/case-studies/<name>/raw
test -d docs/case-studies/<name>/sessions
test -f docs/case-studies/<name>/MANIFEST.md
test -f docs/case-studies/<name>/timeline.md
test -f docs/case-studies/<name>/sessions/SESSION-INDEX.md

# Git history exported
test -f docs/case-studies/<name>/raw/git-log-full.txt

# Session logs collected (at least some)
ls docs/case-studies/<name>/sessions/*.jsonl | wc -l

# Timeline and manifest have substance
wc -l < docs/case-studies/<name>/timeline.md | awk '$1 > 20'
wc -l < docs/case-studies/<name>/MANIFEST.md | awk '$1 > 20'
```

## Phase 6: Report

Summarize what was collected in a table:

| Category           | Files | Size |
| ------------------ | ----- | ---- |
| Session logs       | N     | X MB |
| Transcript exports | N     | X KB |
| Git history + PR   | N     | X KB |
| Planning docs      | N     | X KB |
| Feature docs       | N     | X KB |

Point the user to `timeline.md` as the starting point for finding the narrative, and `MANIFEST.md` for the full artifact inventory.
