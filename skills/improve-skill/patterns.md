# Skill Patterns Reference

Load this file when you need templates or examples for specific skill types.

## Pattern Library

Each pattern shows the frontmatter, explains why it works, and notes what to watch out for.

---

### Analysis Skill

**Use for:** Code review, linting, auditing—read-only inspection.

```yaml
---
name: analyze-x
description: Analyze [target] for [issues]. Use when [natural triggers]. Checks [what].
argument-hint: "[file or pattern]"
allowed-tools: Read, Grep, Glob, Bash(git diff *), Bash(git status *)
---
```

**Why this works:**

- Read-only tools prevent accidental modification
- Git commands scoped to diff/status (safe operations)
- Description includes trigger phrases Claude can pattern-match

**Dynamic context to add:**

```
!‹git status --short 2>/dev/null | head -20›
!‹git diff --stat HEAD~1 2>/dev/null | tail -10›
```

(In actual skills, use exclamation mark + backtick-wrapped command instead of angle brackets.)

---

### Modification Skill

**Use for:** Refactoring, fixing, applying changes.

```yaml
---
name: fix-x
description: Fix [problem] in [target]. Use when [triggers].
argument-hint: "[file or pattern]"
allowed-tools: Read, Write, Edit, Bash(git add *), Bash(npm test *), Bash(bundle exec rspec *)
---
```

**Why this works:**

- Includes test runners—modifications should be verified
- Git add (not commit)—lets user review before committing
- Write + Edit for different modification styles

**Watch out for:**

- Always include a "verify" step in the workflow
- Never auto-commit; let user control that

---

### Manual Workflow Skill

**Use for:** Deploys, releases, anything with real-world consequences.

```yaml
---
name: deploy
description: Deploy to [environment] with safety checks.
argument-hint: "[environment]"
disable-model-invocation: true
allowed-tools: Bash(*)
---
```

**Why this works:**

- `disable-model-invocation: true` means Claude NEVER auto-triggers this
- User must explicitly type `/deploy`
- Broad Bash permissions acceptable because it's gated by manual invocation

**Watch out for:**

- Include explicit confirmation steps in workflow
- Include rollback instructions
- Log what's happening at each step

---

### Background Knowledge Skill

**Use for:** Conventions, patterns, domain knowledge that Claude should absorb.

```yaml
---
name: ruby-conventions
description: Ruby and Rails coding conventions for this project. Applied when writing Ruby code.
user-invocable: false
---
```

**Why this works:**

- `user-invocable: false` hides from `/` menu
- Claude loads it automatically when description matches context
- No workflow needed—just reference content

**Watch out for:**

- Keep description specific so Claude loads it at right times
- If too broad, Claude loads it for everything (wastes context)

---

### Forked Execution Skill

**Use for:** Long-running analysis, deep exploration, anything that would pollute main conversation.

```yaml
---
name: deep-analysis
description: Thorough codebase analysis. Use for architecture review or planning major refactors.
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob
---
```

**Why this works:**

- `context: fork` runs in isolated subcontext
- Main conversation stays clean
- `agent: Explore` uses exploration subagent (better for codebase navigation)

**Watch out for:**

- Forked context can't modify files in main session
- Results come back as summary, not full context
- Use for read-only deep dives

---

## Anti-Patterns

### The Empty Description

```yaml
---
name: my-skill
---
```

**What breaks:** Claude never auto-invokes. You must always type `/my-skill`.

**Fix:** Add description with triggers.

---

### The Vague Description

```yaml
description: Helps with code
```

**What breaks:** Claude invokes it for everything or nothing—no clear signal.

**Fix:** Be specific about WHEN: "Use when reviewing PRs" not "helps with code."

---

### The Permission Spam

```yaml
# No allowed-tools specified
```

**What breaks:** "Can I read this file?" "Can I run git diff?" — constant interruption.

**Fix:** Add `allowed-tools` scoped to actual needs.

---

### The Monolith

```
SKILL.md: 800 lines with checklists, examples, templates, workflows all inline
```

**What breaks:** Claude loads 800 lines every invocation. Slow. Expensive. Most content unused.

**Fix:** Split by role:

- SKILL.md: 100-150 lines (workflow)
- reference.md: Checklists, rules
- examples.md: Before/after patterns

---

### The Silent Argument

```markdown
## Step 1

Analyze the code for issues.
```

**What breaks:** Ignores what user passed. `/skill foo.rb` analyzes random code.

**Fix:** Handle `$ARGUMENTS`:

```markdown
**If `$ARGUMENTS` provided:** Analyze those files
**Otherwise:** Analyze recent changes
```

---

## Description Templates

These capture the three-part structure: what, when, keywords.

**Analysis:**

> Analyze [target] for [issues]. Use when [triggers]. Checks [specifics].

**Modification:**

> [Action] [target] to [outcome]. Use when [triggers].

**Workflow:**

> [Process] with [safeguards]. Use when [triggers].

**Knowledge:**

> [Domain] conventions and patterns. Applied when [context].
