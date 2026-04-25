---
name: agentic-ralph-init
description: Scaffold a criteria-driven Ralph loop that iterates against acceptance criteria from an end-state spec, not a task list. Use when you have a sub-contract or spec with acceptance criteria and want autonomous execution driven by failing checks.
argument-hint: "<path to sub-contract or spec file>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(chmod *), Bash(ls *), Bash(md5 *), Bash(md5sum *), AskUserQuestion
---

# /agentic-ralph-init — Criteria-Driven Autonomous Loop

Set up a Ralph loop that iterates against **acceptance criteria** from an
end-state spec. The agent decides what to implement based on what's failing —
no pre-built task list.

**Key difference from /ralph-init**: The iteration signal comes from failing
acceptance criteria, not a task checklist. The spec describes the target state.
The agent figures out how to get there.

## Existing State

!`ls ralph.sh criteria.sh progress.txt 2>/dev/null || echo "NO_LOOP_FILES"`
!`ls CLAUDE.md 2>/dev/null || echo "NO_CLAUDE_MD"`

## Detected Project Stack

!`ls package.json yarn.lock pnpm-lock.yaml bun.lockb Gemfile Makefile Cargo.toml go.mod pyproject.toml requirements.txt mix.exs Dockerfile 2>/dev/null || echo "NO_STACK_FILES"`

---

## How This Differs from /ralph-init

| Aspect            | /ralph-init                         | /agentic-ralph-init                               |
| ----------------- | ----------------------------------- | ------------------------------------------------- |
| Input             | Goal → generates PRD with task list | Existing spec file with acceptance criteria       |
| Iteration signal  | "Pick next incomplete task"         | "Find first failing criterion"                    |
| Agent autonomy    | Follows prescribed tasks            | Decides implementation from end-state description |
| Progress tracking | progress.txt hash                   | Acceptance criteria pass count                    |
| Completion        | `RALPH_COMPLETE` signal from Claude | All criteria pass (mechanically verified)         |
| Stall detection   | progress.txt unchanged 3x           | Pass count unchanged 3x                           |

---

## Step 1: Locate the Spec File

**Input:** $ARGUMENTS

If `$ARGUMENTS` is a file path, read it. If it's empty, look for common
spec file names (`PRD-agentic*.md`, `spec.md`, `contract.md`) and ask the
user to confirm.

Read the spec file. It MUST have an acceptance criteria section containing
a fenced bash code block with runnable checks. If it doesn't, tell the user
and stop.

---

## Step 2: Extract Acceptance Criteria

Parse the spec file's acceptance criteria bash block. For each criterion:

1. **Command**: The shell command (strip trailing `&& echo "PASS"` if present)
2. **Description**: From the preceding comment, or infer from the command

Generate `criteria.sh` — a standalone script that runs every criterion and
reports results:

```bash
#!/bin/bash
# Auto-generated from: [spec file name]
# Regenerate with: /agentic-ralph-init [spec file]
set -uo pipefail

PASS=0; FAIL=0; FAILURES=""

run_check() {
  local desc="$1"; local cmd="$2"
  if eval "$cmd" >/dev/null 2>&1; then
    ((PASS++)) || true
    echo "  ✅ $desc"
  else
    ((FAIL++)) || true
    echo "  ❌ $desc"
    FAILURES="${FAILURES}  - ${desc}\n"
  fi
}

# -- Criteria extracted from spec --

run_check "Description" 'command here'
# ... one run_check per criterion ...

# -- Summary --
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  $PASS passed, $FAIL failed (of $((PASS+FAIL)) total)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "CRITERIA_PASS=$PASS"
echo "CRITERIA_FAIL=$FAIL"
echo "CRITERIA_TOTAL=$((PASS+FAIL))"
if [ $FAIL -gt 0 ]; then
  echo ""
  echo "FAILING:"
  echo -e "$FAILURES"
fi
```

**Parsing rules:**

- Lines starting with `#` before a command become the description
- Section headers like `# ── Name ──` become group labels
- Strip `&& echo "PASS"` and `&& echo "PASS: ..."` from commands
- Strip `2>/dev/null` only if it's at the end after a pipe (preserve when part of the check)
- Skip blank lines and comment-only lines with no following command
- Slow commands (build, test suites) should be at the END of criteria.sh
  and marked with a `# SLOW` comment so the loop can optionally skip them

After writing, run `chmod +x criteria.sh`.

**Verify** by running `./criteria.sh` and confirming it produces output
with `CRITERIA_PASS=` and `CRITERIA_FAIL=` lines.

---

## Step 3: Detect Tool Permissions

Same as /ralph-init. The `claude -p` sessions run headless — pre-authorize tools.

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

**Also add** `Bash(ruby:*)` if criteria.sh contains ruby commands.
**Also add** `Bash(node:*)` if criteria.sh contains node commands.

**Confirm with the user** using `AskUserQuestion`.

---

## Step 4: Create the Loop Script

Write `ralph.sh` to the project root. This loop runs criteria before and
after each Claude session to mechanically track progress.

Substitute `$ALLOWED_TOOLS` with confirmed tools, `$SPEC_FILE` with the
spec file path.

```bash
#!/bin/bash
set -euo pipefail

MAX_ITERATIONS="${1:-15}"
SPEC_FILE="$SPEC_FILE"
PROGRESS_FILE="progress.txt"
CRITERIA_SCRIPT="criteria.sh"
STALLED=0

ALLOWED_TOOLS="$ALLOWED_TOOLS"

prev_hash() { md5 -q "$PROGRESS_FILE" 2>/dev/null || md5sum "$PROGRESS_FILE" | cut -d' ' -f1; }

if [ ! -f "$SPEC_FILE" ]; then
  echo "Error: $SPEC_FILE not found."
  exit 1
fi

if [ ! -f "$CRITERIA_SCRIPT" ]; then
  echo "Error: $CRITERIA_SCRIPT not found. Run /agentic-ralph-init first."
  exit 1
fi

touch "$PROGRESS_FILE"

# Run criteria and extract pass count
run_criteria() {
  local output
  output=$(./"$CRITERIA_SCRIPT" 2>&1) || true
  echo "$output"
  echo "$output" | grep -oP '(?<=CRITERIA_PASS=)\d+' || echo "0"
}

# Get failing criteria descriptions for the prompt
get_failures() {
  local output
  output=$(./"$CRITERIA_SCRIPT" 2>&1) || true
  echo "$output" | sed -n '/FAILING:/,$ p'
}

for ((i=1; i<=MAX_ITERATIONS; i++)); do
  echo ""
  echo "╔══════════════════════════════════════╗"
  echo "║  Agentic Ralph — Iteration $i / $MAX_ITERATIONS"
  echo "╚══════════════════════════════════════╝"
  echo ""

  # Run criteria BEFORE iteration
  echo "📊 Running acceptance criteria..."
  BEFORE_OUTPUT=$(./"$CRITERIA_SCRIPT" 2>&1) || true
  echo "$BEFORE_OUTPUT"

  BEFORE_PASS=$(echo "$BEFORE_OUTPUT" | grep -oP '(?<=CRITERIA_PASS=)\d+' || echo "0")
  TOTAL=$(echo "$BEFORE_OUTPUT" | grep -oP '(?<=CRITERIA_TOTAL=)\d+' || echo "0")
  BEFORE_FAIL=$(echo "$BEFORE_OUTPUT" | grep -oP '(?<=CRITERIA_FAIL=)\d+' || echo "0")

  # Check if already done
  if [ "$BEFORE_FAIL" = "0" ] && [ "$TOTAL" != "0" ]; then
    echo ""
    echo "✅ All $TOTAL criteria passing. Done!"
    exit 0
  fi

  FAILING=$(echo "$BEFORE_OUTPUT" | sed -n '/FAILING:/,$ p')

  BEFORE_HASH=$(prev_hash)
  ITER_START=$SECONDS
  echo ""
  echo "  ⏳ Claude working on $BEFORE_FAIL failing criteria..."

  result=$(cat <<PROMPT | claude -p \
    --allowedTools "$ALLOWED_TOOLS" 2>&1
You are in iteration $i of $MAX_ITERATIONS of a criteria-driven development loop.

## End-State Specification
$(cat "$SPEC_FILE")

## Currently FAILING acceptance criteria ($BEFORE_FAIL of $TOTAL)
$FAILING

## Progress from previous iterations
$(cat "$PROGRESS_FILE")

## Rules for this iteration
1. Read the spec above — it describes the END STATE, not steps to follow.
2. Pick ONE failing criterion to focus on. Do NOT try to fix everything at once.
   If a criterion requires touching many files (e.g., 280 buttons), do a BOUNDED
   BATCH — convert one directory or ~20 files, then stop. You'll continue next iteration.
3. You decide what to implement. The spec tells you what "done" looks like.
4. After making changes, verify your work by running the relevant checks.
5. Append to $PROGRESS_FILE: iteration number, which criterion you targeted, what you did, how many instances remain.
6. Commit your changes with a descriptive message.
7. SCOPE DISCIPLINE: Better to fully convert 15 files than half-convert 100.
   Each iteration should leave the codebase in a working state.
8. If you encounter a blocker you cannot resolve, note it in $PROGRESS_FILE and output RALPH_BLOCKED.
PROMPT
  ) || true

  echo "  ✅ Iteration $i complete ($((SECONDS - ITER_START))s)"

  # Check for blocker
  if [[ "$result" == *"RALPH_BLOCKED"* ]]; then
    echo ""
    echo "🛑 Agent reported a blocker. Check $PROGRESS_FILE."
    exit 1
  fi

  # Run criteria AFTER iteration
  echo ""
  echo "📊 Re-running acceptance criteria..."
  AFTER_OUTPUT=$(./"$CRITERIA_SCRIPT" 2>&1) || true
  echo "$AFTER_OUTPUT"

  AFTER_PASS=$(echo "$AFTER_OUTPUT" | grep -oP '(?<=CRITERIA_PASS=)\d+' || echo "0")
  AFTER_FAIL=$(echo "$AFTER_OUTPUT" | grep -oP '(?<=CRITERIA_FAIL=)\d+' || echo "0")

  echo ""
  echo "  📈 Progress: $BEFORE_PASS → $AFTER_PASS passing ($((AFTER_PASS - BEFORE_PASS)) flipped)"

  # Check completion
  if [ "$AFTER_FAIL" = "0" ]; then
    echo ""
    echo "✅ All $TOTAL criteria passing after $i iterations!"
    exit 0
  fi

  # Hybrid stall detection: stalled only if BOTH pass count unchanged
  # AND progress.txt unchanged. This handles criteria that take multiple
  # iterations to flip (e.g., converting 280 buttons across many files).
  AFTER_HASH=$(prev_hash)
  PASS_UNCHANGED=$( [ "$AFTER_PASS" -le "$BEFORE_PASS" ] && echo "1" || echo "0" )
  PROGRESS_UNCHANGED=$( [ "$BEFORE_HASH" = "$AFTER_HASH" ] && echo "1" || echo "0" )

  if [ "$PASS_UNCHANGED" = "1" ] && [ "$PROGRESS_UNCHANGED" = "1" ]; then
    ((STALLED++)) || true
    echo "  ⚠️  No progress detected (stall $STALLED/3)"
    if [[ $STALLED -ge 3 ]]; then
      echo "🛑 Stalled for 3 iterations. $AFTER_FAIL criteria still failing. Check $PROGRESS_FILE."
      exit 1
    fi
  elif [ "$PASS_UNCHANGED" = "1" ]; then
    echo "  ℹ️  No criteria flipped yet, but progress logged. Continuing."
    STALLED=0
  else
    STALLED=0
  fi

  sleep 3
done

echo ""
echo "⚠️  Reached max iterations ($MAX_ITERATIONS). $AFTER_FAIL criteria still failing."
echo "    Run ./criteria.sh to see what remains."
```

After writing, run `chmod +x ralph.sh`.

**Platform note:** The `grep -oP` (Perl regex) works on Linux. On macOS,
replace with `grep -o 'CRITERIA_PASS=[0-9]*' | cut -d= -f2` pattern.
Detect the platform and use the appropriate syntax.

---

## Step 5: Create progress.txt

Write an empty `progress.txt`:

```
# Agentic Ralph Progress
# Spec: [spec file name]
# Criteria: [N total checks]
```

---

## Step 6: Verify and Report

1. Run `./criteria.sh` to show the user the starting state
2. Report:

> **Scaffold ready.** Three files created:
>
> - `criteria.sh` — [N] acceptance checks extracted from [spec file]. Run anytime to check progress.
> - `ralph.sh` — Criteria-driven loop. Runs criteria before/after each iteration. Stalls if pass count doesn't increase for 3 cycles.
> - `progress.txt` — Iteration log.
>
> **Current state:** [X] of [N] criteria already passing.
>
> Start the loop:
>
> ```
> ./ralph.sh       # default 15 iterations
> ./ralph.sh 5     # or set a limit
> ```
>
> Check progress anytime: `./criteria.sh`

---

## Rules

- **Never create a PRD or task list.** The spec IS the requirements.
- **Keep ralph.sh simple.** The intelligence is in Claude, not the loop.
- **criteria.sh must be independently runnable.** The user can check progress without running the loop.
- **Three files only:** `ralph.sh`, `criteria.sh`, `progress.txt`.
- **Don't modify the spec file.** It's the contract — read-only to the loop.
- If files already exist, ask before overwriting.
- Detect macOS vs Linux for grep syntax (macOS lacks `grep -P`).
