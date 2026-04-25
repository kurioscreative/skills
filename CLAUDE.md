# CLAUDE.md

Context for Claude when working in the kurioscreative/skills repo.

## What this is

A Claude Code plugin marketplace. See README.md for install instructions.

## Layout: B-bundled (Anthropic-style)

- All skills live flat at `skills/<name>/SKILL.md`
- Plugins in `.claude-plugin/marketplace.json` reference skills via `skills: [...]` arrays
- One marketplace, multiple plugins, plugins reference shared skills

This layout supports three install paths: full marketplace, individual plugin install, and direct copy-paste of a single SKILL.md folder.

## Conventions for skills

- Frontmatter must include `description`. Heavyweight operations should add `disable-model-invocation: true`.
- No hardcoded user-specific paths (`~/Developer/proj/foo`, private CLI tools). Skills must work for anyone who installs them.
- Each skill is standalone — no cross-skill imports or dependencies.

## When a skill joins a plugin bundle

Only after it's proven itself in repeated real use. Speculative bundling creates churn. Most skills should ship as copy-paste-only first; promote into a plugin bundle later.

## Helper scripts

`bin/promote-skill` copies a skill from the local Claude config into this repo. See README for usage.
