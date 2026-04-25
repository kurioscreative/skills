---
name: improve-skill
description: Analyze and improve Claude Code skills. Use when creating new skills, reviewing existing skills, or when a skill isn't working as expected. Teaches skill design strategy, not just checklist compliance.
argument-hint: "[skill-name or path]"
allowed-tools: Read, Glob, Bash(ls *), Bash(wc *), Write, Edit
---

# /improve-skill - Skill Design Strategy

Skills are teaching documents that shape how Claude approaches problems. A well-designed skill doesn't just list steps—it transfers understanding.

## The Five Strategic Levers

These are ordered by impact. Fix them in order.

### 1. Description Is the Trigger System

The `description` field isn't metadata—it's how Claude decides whether to auto-invoke your skill.

**What happens without it:** Claude never uses the skill unless you type `/skill-name`.

**What happens with a weak one:** Claude uses it at wrong times or ignores it when relevant.

**A good description answers three questions:**

- What does this skill do? (1 sentence)
- When should Claude use it? (trigger scenarios)
- What words would a user naturally say? (keywords)

```yaml
# WEAK - Claude doesn't know when to use it
description: Analyzes code quality

# STRONG - Clear triggers
description: Review code for slop and over-engineering. Use when code feels bloated, reviewing AI-generated code, or before commits.
```

**Test:** Read your description. Would Claude know to invoke this skill if a user said "this feels bloated"?

---

### 2. Dynamic Context Sets the Starting Point

The dynamic context syntax (exclamation mark + backtick-wrapped command) runs shell commands before Claude sees the skill content.

**Without it:** Claude's first action is gathering context (reading files, checking git status). Wastes turns.

**With it:** Claude begins with situational awareness. Can start analyzing immediately.

```
## Current State

Git status: !‹git status --short 2>/dev/null | head -20›
Recent changes: !‹git diff --stat HEAD~1 2>/dev/null | tail -10›
```

(In actual skills, use exclamation mark + backtick-wrapped command instead of angle brackets.)

**Strategic question:** What context does Claude need to start working, not just start asking?

---

### 3. Structure Follows Cognitive Load

SKILL.md should contain **workflow** (what to do).
Supporting files should contain **reference** (what to know).

**The split isn't about line count.** It's about role:

| File         | Contains                    | Loaded                    |
| ------------ | --------------------------- | ------------------------- |
| SKILL.md     | Steps, decisions, flow      | Always                    |
| reference.md | Checklists, rules, patterns | When Claude needs detail  |
| examples.md  | Before/after, edge cases    | When Claude needs clarity |

**Anti-pattern:** 800-line SKILL.md with everything inline. Claude loads all of it every time, even for simple invocations.

**Pattern:** 100-line SKILL.md that says "See `red-flags.md` for complete checklist" and Claude loads it only when doing detailed analysis.

---

### 4. allowed-tools Is UX, Not Security

Specifying `allowed-tools` removes permission prompts from the workflow.

**Without it:** "Can I run git diff?" "Can I read this file?" "Can I run tests?" — friction on every action.

**With it:** Claude flows through the workflow without interruption.

```yaml
# Analysis skill (read-only)
allowed-tools: Read, Grep, Glob, Bash(git diff *), Bash(git status *)

# Modification skill (needs write + test)
allowed-tools: Read, Write, Edit, Bash(git add *), Bash(npm test *)
```

**Scope narrowly:** `Bash(git *)` not `Bash(*)`. Permissions should match the skill's actual needs.

---

### 5. Skills Are Teaching Documents

A skill that works but doesn't explain its reasoning will degrade over time:

- Claude won't know when to deviate
- You won't remember why decisions were made
- Edge cases will be handled inconsistently

**Include the why:**

```markdown
# BAD - Just steps

## Step 3

Score each function using the decision framework.

# GOOD - Steps with reasoning

## Step 3: Score with Decision Framework

For every function, class, parameter, and abstraction, score it.
Only code scoring > 0 belongs.

**Why this works:** Forces explicit justification. "Best practice" and
"might need it later" are the most common sources of bloat—they score
negative to counteract this bias.
```

---

## Diagnosis Framework

When a skill isn't working, diagnose in order:

| Symptom                           | Likely Cause             | Fix                                |
| --------------------------------- | ------------------------ | ---------------------------------- |
| Claude never auto-invokes         | Missing/weak description | Rewrite description with triggers  |
| Claude invokes at wrong times     | Description too broad    | Add specific trigger scenarios     |
| Claude asks permission constantly | Missing allowed-tools    | Add scoped tool permissions        |
| Skill feels slow to start         | No dynamic context       | Add dynamic context commands       |
| Claude misses edge cases          | No reference material    | Split checklist to supporting file |
| Inconsistent behavior             | No reasoning in skill    | Add "why" to key decisions         |

---

## Workflow

### Step 1: Locate Skill

**If `$ARGUMENTS` provided:** Find that skill
**Otherwise:** List available skills

Use `Glob` to list personal skills at `~/.claude/skills/*/SKILL.md`

### Step 2: Assess Current State

Read SKILL.md and answer:

1. **Description quality** (1-10): Would Claude know when to invoke this?
2. **Context setup** (1-10): Does Claude start with awareness or have to gather it?
3. **Structure clarity** (1-10): Is workflow separate from reference?
4. **UX friction** (1-10): Will Claude flow or ask permission constantly?
5. **Teaching quality** (1-10): Does it explain why, not just what?

### Step 3: Identify Highest-Leverage Fix

Work through the five levers in order. Stop at the first broken one—that's your highest-leverage fix.

### Step 4: Generate Recommendations

```markdown
## Skill Analysis: [name]

### Current State

[Scores for each dimension]

### Highest-Leverage Fix

[The one thing that would most improve this skill]

### Additional Improvements

[Other issues, in priority order]

### Rewritten Frontmatter

[If applicable]
```

### Step 5: Apply (If Requested)

On "fix it" / "apply":

1. Back up: `cp SKILL.md SKILL.md.bak`
2. Apply changes
3. Show before/after
