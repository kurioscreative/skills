---
name: shape
description: Shape vague goals into verifiable task contracts before agentic execution. Use when starting any significant task, before /agentic, or when a goal feels unclear. Converts intent into machine-testable acceptance criteria.
argument-hint: "<goal description>"
allowed-tools: Read, Glob, Grep, Bash(git *), Bash(ls *), Bash(bundle *), Bash(npm *), Bash(pnpm *), AskUserQuestion
---

# /shape - Outcome & Constraint Shaping

Turn intent into a **task contract** with verifiable acceptance criteria.

> The difference between "mostly achieved" and "fully achieved" is whether
> "done" is a feeling or a test.

## Current Context

Project root:
!`ls -1 2>/dev/null | head -15`

Git state:
!`git status --short 2>/dev/null | head -10`

Stack signals:
!`ls Gemfile Gemfile.lock package.json tsconfig.json Cargo.toml go.mod pyproject.toml mix.exs 2>/dev/null || true`

---

## Phase 1: Understand Intent

**Input:** $ARGUMENTS

1. Parse the goal into:
   - **What** changes (the deliverable)
   - **Where** in the codebase (files, modules, layers)
   - **Why** it matters (user motivation, if stated)

2. Explore the codebase to ground the goal in reality:
   - What exists already? (don't rebuild what's there)
   - What test infrastructure exists? (rspec, jest, pytest — this determines how we verify)
   - What CI/deployment exists? (what gates already run?)
   - What conventions are established? (CLAUDE.md, linter configs, existing patterns)

   **Every claim you later put in the Context section must come from an
   action you took here** — a file you read, a grep you ran, a command you
   executed. If you catch yourself wanting to write something you didn't
   actually verify, either verify it now or move it to an **Open Questions
   for Executing Agent** subsection in the contract so the agent knows to
   confirm before acting on it. Unverified claims dressed as facts are the
   single most expensive thing a contract can contain — the agent anchors
   on them and stops investigating.

   **When a claim rests on another file's behavior** — a method's return
   type, a rake task's `DRY_RUN` default, a callback chain, a constant's
   value — paste the relevant source lines inline as a quoted code block
   instead of paraphrasing. "Catch yourself before paraphrasing" is
   self-policed and leaks; quoting replaces the self-check with a visible
   artifact. Example, instead of:

   > "`Location#churches` for a city returns `Church.where(location_id: id)`"

   paste:

   ```ruby
   # app/models/location.rb:157-168
   def churches(threshold: 30, boost: false)
     scope = case category
     when "city"
       if search_boundary_type(threshold:) == :near
         Church.near(coordinates, ...).or(Church.where(location_id: id))
       else
         Church.where(location_id: id)
       end
   ```

   Quoting forces the file to have been opened, surfaces branches a
   paraphrase would elide (the `:near` branch above), and lets the executor
   verify the claim without re-reading every cited file.

3. Identify the **category**:

| Category       | Characteristics                 | Verification Style                                    |
| -------------- | ------------------------------- | ----------------------------------------------------- |
| Feature        | New behavior                    | Test passes, UI renders, API responds                 |
| Fix            | Broken behavior                 | Regression test passes, error gone                    |
| Refactor       | Same behavior, better structure | All existing tests still pass + new structural checks |
| Config/Tooling | Developer experience            | Tool runs successfully, output matches expectations   |
| Migration      | Moving from A to B              | B works, A removed, no regressions                    |

---

## Phase 2: Shape Through Questions

Use **AskUserQuestion** to build a complete picture. The number of questions
scales with the complexity discovered in Phase 1 — a config tweak might need
2 questions, a multi-file migration might need 10+.

**AskUserQuestion supports 1-4 questions per call.** Use multiple rounds.
Each round should build on answers from the previous round — don't ask
everything upfront if later questions depend on earlier answers.

### Question Categories

Draw from these categories based on what's genuinely ambiguous. Skip any
category where the answer is obvious from the codebase.

**Scope** — what's in and what's out:

- How far should this go? (minimal / standard / comprehensive)
- Which parts of the codebase? (specific dirs, modules, layers)
- Should this touch tests, docs, CI config, or just source?

**Approach** — how to get there:

- Multiple valid strategies found — which fits your situation?
- Build new vs. extend existing vs. replace existing?
- Which library/tool/pattern among alternatives?

**Quality bar** — what "done" means:

- Working (smoke test) / Solid (tests + lint) / Production (full gates)?
- Coverage expectations for new code?
- Performance requirements if on a critical path?

**Constraints** — hard boundaries:

- Files, directories, or systems that are off-limits?
- Time/cost budget for this task?
- Dependencies that must not change?

**Decision authority** — autonomy boundaries:

- What can the agent decide without asking? (naming, file structure, etc.)
- What requires a human decision during execution? (API design, data model)
- Any decisions that feel reversible vs. ones that feel permanent?

**Dependencies & ordering** — what interacts:

- Does this need to land before/after other work?
- Are there concurrent changes on other branches that might conflict?
- External systems that need coordination? (deploy, migration, API consumers)

**Risk** — what could go wrong:

- What's the blast radius if this goes wrong?
- Is this reversible? How?
- Are there known edge cases or gotchas from past experience?

### Principles for Questioning

- **Only ask about genuine ambiguities** found during Phase 1 exploration.
  If the codebase answers it, state the answer — don't ask.
- **Front-load the most consequential decisions.** Scope and approach shape
  everything downstream, so ask those first.
- **Make options concrete.** "Minimal" means nothing without specifics.
  "Minimal: only lib/converter.rb and its spec" means something.
- **Include tradeoff context in option descriptions.** The user is making
  decisions — give them the information to decide well.
- **Use multiSelect when choices aren't mutually exclusive.** E.g., "Which
  quality gates?" could allow selecting multiple.
- **State what you'll assume if no answer.** This respects the user's time
  on questions they don't care about.

---

## Phase 3: Generate Task Contract

**Output the contract as a fenced code block in the conversation.** Do not
create files (no PLAN.md, no task-contract.md, nothing). The contract is
portable — the user will copy it into their chosen execution method.

Format it as a fenced markdown block the user can copy directly into a new session, `/agentic-v2`, or headless mode.

```markdown
## Task Contract: [title]

### Goal

[1-2 sentence outcome statement — what the world looks like when this is done]

### Acceptance Tests

[Generate per the Acceptance Test Principles below]

### Constraints

- OFF-LIMITS: [files/dirs/systems not to touch]
- BUDGET: [scope boundary — e.g., "only these 3 files" or "this module only"]
- DECIDE FREELY: [what the agent can choose without asking]
- ASK ME: [what requires human decision during execution]

### Context

[Key codebase facts the executing agent needs to know — existing patterns,
relevant files, test commands, conventions discovered in Phase 1]
```

### Acceptance Test Principles

Generate shell commands that return exit 0 when the task is done.

**Rules:**

- Every criterion is a runnable command, never prose
- Order by execution speed — fastest first
- The agent will **loop against the fastest failing criterion**,
  so the primary deliverable check must be fast (< 10 seconds)
- Separate "must pass" from "nice to have" with a clear `DONE =` line
- If a criterion only applies conditionally, state the condition
  as a command the agent can check before running it
- Prefer commands already in the project's toolchain (`bundle exec`,
  `npm test`, `pytest`) over generic checks (`curl`, `grep`)
- Always include a regression check as the final criterion —
  existing tests scoped to the affected area must still pass
- Scale the number of criteria to the task: a 2-file fix might
  need 2 checks; a migration might need 6 — don't pad or skimp
- If the project has auto-formatting hooks (rubocop -a, prettier),
  don't duplicate those as criteria — they run automatically

**Examples of good criteria:**

- `bundle exec rubocop --format json | jq '.summary.offense_count' | grep -q '^0$'`
- `find app/views/admin -name '*.haml' | wc -l | grep -q '^0$'`
- `bundle exec rspec spec/models/user_spec.rb --format progress`

**Examples of bad criteria:**

- "rubocop is configured" (not a command)
- "tests pass" (which tests? be specific)
- "the page looks right" (subjective — find a grep or parse check instead)

---

## Phase 4: Confirm & Handoff

Present the task contract inline (already displayed from Phase 3 — do not
write it to a file) and ask: "Ready to execute this, or want to adjust?"

If the user says go, suggest the execution method:

- **Simple task**: just do it in this session
- **Multi-step task**: hand off to `/agentic-v2` with the contract as input
- **Repeatable task**: headless mode with `claude -p "[contract]"`
- **Exploration needed first**: spawn a research subagent, then shape again with findings
