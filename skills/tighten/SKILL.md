---
name: tighten
description: Review code for slop and over-engineering. Use when code feels bloated, reviewing AI-generated code, after writing significant code, or before commits. Applies decision framework scoring and red flag detection.
argument-hint: "[file, pattern, or blank for recent changes]"
allowed-tools: Read, Grep, Glob, Bash(git diff *), Bash(git show *), Bash(git status *), Bash(bundle exec rspec *), Bash(npm test *)
---

# /tighten - Code Tightening & Slop Removal

Ruthlessly simplify code. Every line must earn its place.

> "Every line of code is a liability. The best code is no code."

## Current Context

Git status:
!`git status --short 2>/dev/null | head -20`

Recent changes:
!`git diff --stat HEAD~1 2>/dev/null | tail -10`

---

## Workflow

### Step 1: Identify Target

**If `$ARGUMENTS` provided:** Analyze those files/patterns
**If unstaged changes exist:** Analyze the diff
**If recent commit:** Analyze HEAD~1
**Otherwise:** Ask what to tighten

### Step 2: Score with Decision Framework

For every function, class, parameter, and abstraction:

| Criterion                    | Points |
| ---------------------------- | ------ |
| User explicitly requested it | +3     |
| Code fails without it        | +3     |
| Worked around this 3+ times  | +2     |
| Removes existing code        | +1     |
| "Might need it later"        | **-3** |
| "Best practice"              | **-2** |
| "More flexible"              | **-2** |
| Adds new abstraction         | -1     |

**Only code scoring > 0 belongs.**

### Step 3: Check Red Flags

See `red-flags.md` for complete checklist. Key patterns:

- **Abstraction smells:** Single-implementation interfaces, one-subclass base classes, unused registries
- **Over-engineering:** >3 parameters, Handler/Manager/Registry classes, day-one CircuitBreakers
- **AI slop:** Comment spam, log-and-rethrow, redundant nil checks, async without await
- **Ruby/Rails:** Single-includer Concerns, one-call Services, trivial Decorators

### Step 4: Apply 50% Test

> "Can I delete 50% and still solve the problem?"

If yes → propose the deletion.

### Step 5: Report

```markdown
## Tighten Report: [scope]

### Remove (Score ≤ 0)

[Items with scores]

### Red Flags

[Patterns matched]

### Proposed Changes

[Diffs]

### Keeps Its Place

[Code that passed, brief reason]
```

### Step 6: Apply (If Requested)

On "do it" / "fix it" / "apply":

1. Make changes
2. Run tests
3. Show before/after line count
4. Stage but DO NOT commit

---

## Quick Reference

**The Tighten Questions:**

1. Would a new dev understand this in 5 minutes?
2. Am I solving real problems or imaginary ones?
3. Have I added code for problems I've never seen?
4. Does every line earn its place?

**Progressive Enhancement:**

```
Version 1: Just works
Version 2: Optimize (after proving bottleneck)
Version 3: Resilience (after experiencing failures)
```

Reality forces each enhancement. Not imagination.
