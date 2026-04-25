---
name: enhance
description: |
  Make the single most impactful improvement to the project. Evaluates current
  state through structured lenses, picks ONE change (add, remove, or revise),
  executes it, verifies the build, and commits with a user-facing description.
  Use when iteratively polishing a project, when you want to find and fix the
  most important thing, or as the engine inside a ralph loop.
argument-hint: "[focus area, or blank for auto]"
disable-model-invocation: true
allowed-tools: |
  Read, Write, Edit, Glob, Grep,
  Bash(ls *), Bash(cat *), Bash(head *), Bash(grep *), Bash(echo *),
  Bash(git add *), Bash(git commit *), Bash(git log *), Bash(git diff *), Bash(git status *),
  Bash(pnpm *), Bash(npm *), Bash(npx *),
  Bash(yarn *), Bash(bun *), Bash(bundle *), Bash(rake *),
  Bash(make *), Bash(cargo *), Bash(go *), Bash(python *),
  Bash(pytest *), Bash(mix *)
---

# /enhance — Find and Make the Most Impactful Change

One iteration of an evolutionary improvement loop. Evaluate broadly, act
narrowly. The output is one committed change that makes the project measurably
better for users.

## Current State

Project root:
!`ls -1 2>/dev/null | head -20`

Recent commits (voice and cadence to match):
!`git log --oneline -10 2>/dev/null`

Build/test commands:
!`head -30 package.json 2>/dev/null`
!`ls Makefile Rakefile Cargo.toml go.mod pyproject.toml mix.exs 2>/dev/null || true`

Existing enhancement context:
!`ls PRD*.md progress*.txt AUDIENCE.md 2>/dev/null || echo "NO_PRD_FILES"`

CLAUDE.md (project conventions):
!`cat CLAUDE.md 2>/dev/null | head -40`

---

## Focus

**User-specified focus:** $ARGUMENTS

If a focus area was provided, use it to constrain which lenses matter most.
If blank, evaluate all lenses and let the diagnosis choose.

---

## Step 1: Read the Territory

Before evaluating anything, understand the current state:

1. **Read the project's PRD or CLAUDE.md** to understand goals and constraints.
   If a `PRD.md`, `PRD-product.md`, or similar exists, that defines the
   dimensions to optimize. If not, infer dimensions from the project type.

2. **Read progress files** (`progress.txt`, `progress-product.txt`, or recent
   git log) to understand what has already been done. This prevents:
   - Repeating a change that was already made
   - Adding when the last 5 iterations all added (time to subtract)
   - Missing the pattern of what's working vs. what's accumulating noise

3. **Read the relevant source files** — not everything, but the surfaces a
   user actually sees and interacts with. For a web app: pages, layouts,
   key components. For a library: public API, README, main entry points.

---

## Step 2: Evaluate Through Lenses

Evaluate the project through multiple lenses. The lenses adapt to the project
type, but the structure is always the same: each lens asks "what's the most
important thing wrong here?"

### For Web Apps / UI Projects

**Instant Value** — Can a new user get value in seconds?

- Is the primary action obvious and above the fold?
- Can the user accomplish their goal without learning the UI?
- Are there distractions competing with the primary action?

**Clarity** — Can the user tell what to do next?

- Is information hierarchy clear (what's primary, secondary, tertiary)?
- Are labels, placeholders, and empty states guiding the user?
- Does the layout make the user's next action obvious?

**Content** — Are words doing their job?

- Are descriptions accurate and specific, or generic filler?
- Are CTAs action-oriented and clear?
- Is there unnecessary jargon that could be plain language?

**Design Quality** — Does this feel intentionally designed?

- Typography: is weight/size creating hierarchy or fighting it?
- Spacing: is there breathing room between surfaces?
- Consistency: do similar elements look and behave the same way?

**Accessibility** — Can everyone use this?

- Text contrast meeting WCAG minimums
- Focus states visible on all interactive elements
- Touch targets large enough on mobile

### For Libraries / CLIs / APIs

**API Surface** — Is the public interface clean and obvious?

- Can a user accomplish the common case with minimal configuration?
- Are defaults sensible?
- Is the naming consistent and self-documenting?

**Documentation** — Can someone use this without reading source?

- Does the README show the most common use case first?
- Are examples runnable, not pseudocode?
- Are edge cases documented where users will look for them?

**Error Experience** — What happens when things go wrong?

- Are error messages actionable (say what to do, not just what failed)?
- Are common mistakes caught with helpful messages?

### Universal Lenses (All Projects)

**Subtraction** — What should be removed?

- Is there accumulated noise from iterative development?
- Effects, features, or code that don't earn their place?
- Complexity that serves hypothetical users, not real ones?

**Revision** — What past addition isn't working?

- Read the progress log. Which changes feel wrong in context?
- Has iterative addition created inconsistency?
- Would a user be confused by something that made sense in isolation?

---

## Step 3: Diagnose and Commit to ONE Change

After evaluating all relevant lenses, pick **one** change. Not two. Not "and
also while I'm here." One.

The change can be any scope — from a single label to a layout restructure —
but it must be:

1. **The most impactful** thing you found across all lenses
2. **User-visible** — someone using the project would notice
3. **Completable** in a single pass — no half-finished states

State your diagnosis before acting:

> **Lens:** [which lens surfaced this]
> **Diagnosis:** [what's wrong, in one sentence]
> **Action:** [add / remove / revise]
> **Change:** [what you'll do, in one sentence]

**Bias toward subtraction and revision.** If the progress log shows many
additive iterations, the most impactful thing is almost certainly to remove
or revise, not add more.

---

## Step 4: Make the Change

Execute the change. Be thorough — if you're changing a label, change it
everywhere it appears. If you're removing an effect, remove all traces.
If you're revising a layout, make sure it works at all breakpoints.

Do NOT:

- Refactor code without user-visible change
- Add comments, documentation, or type annotations to code you didn't change
- Make "while I'm here" improvements to adjacent code
- Add new dependencies

---

## Step 5: Verify

Run the project's build and/or test commands. The change must not break
anything.

Detect and run the appropriate commands:

- `pnpm build` / `npm run build` / `yarn build` (if package.json exists)
- `bundle exec rspec` / `rake` (if Gemfile exists)
- `cargo build` / `cargo test` (if Cargo.toml exists)
- `go build ./...` / `go test ./...` (if go.mod exists)
- `pytest` / `python -m pytest` (if pyproject.toml/requirements.txt exists)
- `mix compile` / `mix test` (if mix.exs exists)
- `make` (if Makefile exists)

If the build fails, fix the issue. Do not commit broken code.

---

## Step 6: Commit

Commit with a message that describes **what changed for the user**, not what
code changed. Follow the project's commit voice (check recent git log).

Good: `ux: pipeline bar shows tool descriptions on hover`
Good: `remove: strip inner shadows from content wells — cleaner read`
Good: `revise: suggested pipelines use action verbs instead of tool names`

Bad: `update ToolWidget.jsx`
Bad: `refactor CSS variables`
Bad: `improve UX`

---

## Step 7: Log Progress

If a progress file exists (`progress.txt`, `progress-product.txt`, etc.),
append 2-3 lines:

- Whether this was an **addition**, **removal**, or **revision**
- What changed for the user (not what code changed)
- Why this improves the experience

If no progress file exists, the commit message is sufficient.

---

## The Core Tension

This skill works because of the tension between **evaluate broadly** and
**act narrowly**. The lenses force you to see the whole picture. The one-change
constraint forces you to prioritize ruthlessly. The commit discipline forces
you to articulate impact. The build gate forces you to leave things working.

Resist the urge to do more. One good change, well-executed and well-described,
compounds across iterations.
