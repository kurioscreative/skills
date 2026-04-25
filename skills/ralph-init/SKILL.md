---
name: ralph-init
description: Scaffold a Ralph loop for autonomous multi-iteration development. Use when setting up iterative AI coding loops, autonomous development, or when someone mentions "ralph" or wants Claude to work in a loop.
argument-hint: "<project goal>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(chmod *), Bash(ls *), AskUserQuestion, Skill(shape)
---

# /ralph-init — Scaffold an Autonomous Development Loop

Set up the Ralph pattern: a bash loop that spawns fresh Claude sessions per
iteration, persisting state through files and git — not context accumulation.

## Existing State

!`ls ralph.sh PRD.md progress.txt 2>/dev/null || echo "NO_RALPH_FILES"`
!`ls CLAUDE.md 2>/dev/null || echo "NO_CLAUDE_MD"`

## Detected Project Stack

!`ls package.json yarn.lock pnpm-lock.yaml bun.lockb Gemfile Makefile Cargo.toml go.mod pyproject.toml requirements.txt mix.exs Dockerfile 2>/dev/null || echo "NO_STACK_FILES"`

---

## Why This Exists

Anthropic's official Ralph Wiggum plugin is broken by design — it keeps all
iterations in a single session, filling the context window until Claude enters
the "dumb zone" (~40% context). The correct pattern exits after each iteration
so every cycle gets a fresh context window. State persists via files + git.

---

## Step 1: Understand the Goal

**Input:** $ARGUMENTS

If `$ARGUMENTS` is empty or vague, ask the user what they want to build.
You need enough to seed the PRD.

---

## Step 2: Detect Tool Permissions

The `claude -p` sessions run headless — no user to approve permission prompts.
We must pre-authorize tools via `--allowedTools`. Detect the project's stack
so the loop can actually run tests and build commands.

**Always include these base tools:**

```
Bash(git:*) Edit Write Read Glob Grep
```

**Auto-detect from project files:**

| File Found         | Add Permission                              |
| ------------------ | ------------------------------------------- |
| `package.json`     | `Bash(npm:*) Bash(npx:*)`                   |
| `yarn.lock`        | `Bash(yarn:*)`                              |
| `pnpm-lock.yaml`   | `Bash(pnpm:*)`                              |
| `bun.lockb`        | `Bash(bun:*)`                               |
| `Gemfile`          | `Bash(bundle:*) Bash(rake:*)`               |
| `Makefile`         | `Bash(make:*)`                              |
| `Cargo.toml`       | `Bash(cargo:*)`                             |
| `go.mod`           | `Bash(go:*)`                                |
| `pyproject.toml`   | `Bash(python:*) Bash(pytest:*) Bash(pip:*)` |
| `requirements.txt` | `Bash(python:*) Bash(pytest:*) Bash(pip:*)` |
| `mix.exs`          | `Bash(mix:*)`                               |
| `Dockerfile`       | `Bash(docker:*)`                            |

Use `Glob` to check which of these exist. Build the combined tools string.

**Confirm with the user** using `AskUserQuestion`:

> Detected tools for your project: `Bash(git:*) Bash(npm:*) Bash(npx:*) Edit Write Read Glob Grep`
>
> These permissions let the autonomous loop run git, npm, and file operations
> without prompting. Add or remove any?

Incorporate their feedback, then use the final string as `$ALLOWED_TOOLS` in Step 3.

---

## Step 3: Create the Loop Script

Write `ralph.sh` to the project root. This is the core — a bash while loop
that calls `claude -p` with fresh context each iteration.

Substitute `$ALLOWED_TOOLS` with the confirmed tools string from Step 2.

```bash
#!/bin/bash
set -euo pipefail

MAX_ITERATIONS="${1:-10}"
PROMPT_FILE="PRD.md"
PROGRESS_FILE="progress.txt"
STALLED=0

# Tools the autonomous Claude session is allowed to use without prompting.
# Detected during /ralph-init from project files. Edit to add/remove permissions.
ALLOWED_TOOLS="$ALLOWED_TOOLS"

if [ ! -f "$PROMPT_FILE" ]; then
  echo "Error: $PROMPT_FILE not found. Run /ralph-init first."
  exit 1
fi

touch "$PROGRESS_FILE"

prev_hash() { md5 -q "$PROGRESS_FILE" 2>/dev/null || md5sum "$PROGRESS_FILE" | cut -d' ' -f1; }

for ((i=1; i<=MAX_ITERATIONS; i++)); do
  echo ""
  echo "╔══════════════════════════════════════╗"
  echo "║  Ralph Loop — Iteration $i / $MAX_ITERATIONS"
  echo "╚══════════════════════════════════════╝"
  echo ""

  BEFORE=$(prev_hash)
  ITER_START=$SECONDS
  echo "  ⏳ Claude working..."

  result=$(cat <<PROMPT | claude -p \
    --allowedTools "$ALLOWED_TOOLS" 2>&1
You are in iteration $i of $MAX_ITERATIONS of an autonomous development loop.

## Your instructions
$(cat "$PROMPT_FILE")

## Progress so far
$(cat "$PROGRESS_FILE")

## Rules for this iteration
1. Read the PRD and progress file. Find the highest-priority incomplete task.
2. Implement ONE task only. Do it well.
3. Run any tests or checks relevant to your change.
4. Append a brief summary of what you did to $PROGRESS_FILE.
5. Commit your changes with a descriptive message.
6. If ALL tasks in the PRD are complete, output RALPH_COMPLETE as the last line.
PROMPT
  ) || true

  echo "  ✅ Iteration $i complete ($((SECONDS - ITER_START))s)"
  echo "$result"

  if [[ "$result" == *"RALPH_COMPLETE"* ]]; then
    echo ""
    echo "✅ All tasks complete after $i iterations."
    exit 0
  fi

  AFTER=$(prev_hash)
  if [[ "$BEFORE" == "$AFTER" ]]; then
    ((STALLED++)) || true
    echo "⚠️  No progress detected (stall $STALLED/3)"
    if [[ $STALLED -ge 3 ]]; then
      echo "🛑 Stalled for 3 iterations. Aborting. Check progress.txt."
      exit 1
    fi
  else
    STALLED=0
  fi

  sleep 3
done

echo ""
echo "⚠️  Reached max iterations ($MAX_ITERATIONS). Check progress.txt for status."
```

After writing, run `chmod +x ralph.sh`.

---

## Step 4: Create the PRD

Write `PRD.md` as a starting template seeded from `$ARGUMENTS`:

```markdown
# Project: [title from $ARGUMENTS]

## Goal

[1-2 sentence description of what "done" looks like]

## Tasks

- [ ] [First concrete task]
- [ ] [Second concrete task]
- [ ] [Third concrete task]

## Constraints

- [Any boundaries, off-limits areas, or quality requirements]

## Context

- [Key codebase facts, patterns, or conventions the loop should know]
- [Test commands, build commands, relevant file paths]
```

Populate it with what you can infer from `$ARGUMENTS` and the codebase.
Leave `[ ]` placeholders only where you genuinely don't know.

---

## Step 5: Create progress.txt

Write an empty `progress.txt` with just a header:

```
# Ralph Loop Progress
```

---

## Step 6: Offer to Shape the PRD

After creating the files, tell the user:

> The scaffold is ready. Your PRD is a rough starting point.
>
> **Recommended next step:** Run `/shape` to turn the PRD into a
> proper task contract with verifiable acceptance criteria. This makes
> the loop dramatically more effective — Claude knows exactly when
> each task is "done" instead of guessing.
>
> Then run the loop from your terminal:
>
> ```
> ./ralph.sh      # default 10 iterations
> ./ralph.sh 5    # or set a limit
> ```

---

## Rules

- **Never create a plugin or hook.** The loop runs outside Claude Code.
- **Keep ralph.sh simple.** The intelligence is in Claude, not the loop.
- **Don't over-scaffold.** Three files: `ralph.sh`, `PRD.md`, `progress.txt`.
- If files already exist, ask before overwriting.
