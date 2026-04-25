---
description: Transform implementation requests into parallelized agentic execution with creative decomposition, autonomous agents, and quality gates.
disable-model-invocation: true
---

# Agentic Execution

Transform the user's request into an optimized multi-agent execution strategy.

**Request**: $ARGUMENTS

---

## Core Principles

1. **Decompose for parallelism** - Independent work streams execute concurrently
2. **Fail fast, recover gracefully** - Validate early, rollback cleanly
3. **Programmatic verification** - Success criteria must be automatable (no "looks good")
4. **State persistence** - Interruption shouldn't lose progress
5. **Minimize blast radius** - Sequence work so failures don't cascade

---

## Phase 1: Strategic Assessment

Before planning, answer these questions:

### Is Agentic Right?

- **Yes**: Multiple independent subsystems, clear parallelization, >3 distinct deliverables
- **No**: Single-file change, exploratory/research, unclear requirements, heavy interdependencies

If agentic isn't optimal, say so and propose an alternative approach.

### What's the Decomposition Strategy?

Consider multiple approaches before committing:

| Strategy                 | When to Use                                            |
| ------------------------ | ------------------------------------------------------ |
| **Layer-based**          | Database → Models → Services → API → UI                |
| **Feature-based**        | Feature A (full stack) ∥ Feature B (full stack)        |
| **Risk-based**           | Hardest/riskiest first, easier work parallelized after |
| **Dependency-minimized** | Maximize independent agents, minimize handoffs         |

**Choose the strategy that maximizes parallelism while minimizing conflict risk for THIS specific task.**

### What Could Go Wrong?

Identify risks upfront:

- File conflicts (parallel agents touching same files)
- Resource constraints (API limits, cost, context size)
- Quality risks (security-sensitive, performance-critical, public-facing)

---

## Phase 2: Creative Decomposition

Generate the execution plan. For each agent, specify:

```
Agent N: [Descriptive Name]
├─ Creates: [specific file paths]
├─ Modifies: [specific file paths]
├─ Depends on: [agent numbers or "none"]
├─ Success criteria:
│  ├─ [command] exits 0
│  ├─ [test command] passes (N examples)
│  └─ [verification command] returns expected
├─ Rollback: [specific undo commands]
└─ Estimated: [tokens]k, $[cost], [time]
```

**Requirements for success criteria:**

- Must be runnable commands (not prose)
- Include test commands with expected counts
- Include at least one integration verification
- Performance benchmarks if on critical path

### Dependency Graph

Visualize the execution flow:

```
Agent 1 ──┬──► Agent 2 ──► Agent 5
          └──► Agent 3 ──┘
               Agent 4 ──────────► Agent 6
```

Verify: **Is this graph acyclic?** (no circular dependencies)

Calculate:

- Critical path (longest sequential chain)
- Parallel opportunities
- Estimated speedup vs sequential

### Conflict Check

For parallel agents, verify no file conflicts:

- ✅ Different files → safe to parallelize
- ⚠️ Same files → sequence or split responsibilities

---

## Phase 3: Human Collaboration Point

Before executing, present your plan and invite strategic input:

```markdown
## Agentic Plan: [Name]

**Approach**: [1-2 sentence strategy]
**Agents**: [N] across [M] sequences
**Parallelism**: [P] concurrent max
**Estimated**: [time] ([X]× faster than sequential), $[cost]

**Sequences**:

1. [Phase name]: Agents 1-2 (sequential, foundation)
2. [Phase name]: Agents 3-4 (parallel, independent features)
3. [Phase name]: Agent 5 (integration)

**Key Risks**: [top 1-2 risks and mitigations]

**Decision Points**:

- [Any architectural choices that would benefit from human input]
- [Trade-offs where preferences matter]

Proceed? [Y/n] or provide guidance.
```

**Don't ask about things you can reasonably decide. DO ask about:**

- Ambiguous requirements with multiple valid interpretations
- Trade-offs between approaches (speed vs maintainability, etc.)
- Scope boundaries (MVP vs comprehensive)
- Unknowns that affect the plan

---

## Phase 4: Execution

### State Tracking

Create `agentic-[slug]-[timestamp].md` in the working directory:

```markdown
# Agentic: [Feature Name]

Started: [timestamp] | Branch: agentic/[slug]

## Progress

- [ ] Sequence 1: [status]
- [ ] Sequence 2: [status]

## Checkpoints

- [commit hash]: Sequence N complete

## Discoveries

- [Agent N]: [learning that affects downstream agents]

## If Interrupted

1. Checkout branch: agentic/[slug]
2. Last checkpoint: [commit]
3. Resume from: [agent/sequence]
```

### Execution Loop

For each sequence:

1. **Launch agents** (parallel where safe - use single message with multiple Task calls)
2. **Verify success criteria** (run the commands, check exit codes)
3. **Quality gate** (stack-appropriate: tests, linter, security scanner)
4. **Checkpoint** (git commit with descriptive message)
5. **Update state file**
6. **Propagate discoveries** (update downstream agent context if learnings emerged)

### Quality Gates (Stack-Adaptive)

Detect the stack and apply appropriate gates:

- **Tests**: `npm test`, `pytest`, `rspec`, `go test`, etc.
- **Linter**: `eslint`, `rubocop`, `ruff`, `golint`, etc.
- **Types**: `tsc`, `mypy`, `sorbet`, etc.
- **Security**: `brakeman`, `npm audit`, `bandit`, etc.

Gates are **blocking** - don't proceed if they fail.

---

## Phase 5: Error Recovery

When an agent fails, don't just retry blindly. Diagnose first:

| Failure Type            | Recovery Strategy                                                |
| ----------------------- | ---------------------------------------------------------------- |
| **Test failures**       | Analyze failures, refine agent prompt with specifics, retry once |
| **Complexity exceeded** | Decompose into smaller agents                                    |
| **Wrong approach**      | Propose alternative, get human approval                          |
| **External blocker**    | Partial rollback, preserve working sequences, surface to human   |

**Recovery format:**

```markdown
❌ Agent N failed: [brief reason]

Analysis: [what went wrong]
Options:

1. [Refined retry] - [what changes]
2. [Decompose] - [how to split]
3. [Alternative] - [different approach]
4. [Rollback to Sequence M] - [preserve valid work]

Recommend: [N] because [reason]
```

---

## Phase 6: Completion

After all sequences complete:

1. **Final quality gate** (full test suite, linter, security scan)
2. **Integration verification** (end-to-end test or manual verification)
3. **Update state file** with completion status
4. **Summary**:

```markdown
## Agentic Complete: [Feature Name]

**Delivered**:

- [Key deliverable 1]
- [Key deliverable 2]

**Metrics**:

- Agents: [N] ([P] parallel)
- Time: [actual] vs [estimated]
- Cost: $[actual] vs $[estimated]

**Quality**:

- Tests: [N] passing
- Coverage: [X]%
- Security: [clean/issues]

**Learnings**: [anything notable for future similar tasks]
```

---

## Creative Latitude

This framework is a starting point, not a cage. You may:

- **Propose a different decomposition** if you see a better approach
- **Adjust the methodology** if the task warrants it
- **Simplify aggressively** if the task doesn't need full agentic treatment
- **Identify non-obvious opportunities** (unexpected parallelization, shared abstractions)
- **Challenge the request** if you see a fundamentally better way to achieve the goal

The best outcome isn't perfect framework adherence—it's the user's goal achieved efficiently and well.

---

## Anti-Patterns to Avoid

- ❌ Agents that are too large (>15 min estimated)
- ❌ Agents that are too small (overhead exceeds value)
- ❌ Subjective success criteria ("works correctly")
- ❌ Parallelizing agents with file conflicts
- ❌ Skipping quality gates to save time
- ❌ Proceeding after failures without diagnosis
- ❌ Losing state (always checkpoint, always track)

---

**Begin Phase 1: Assess the request and determine your approach.**
