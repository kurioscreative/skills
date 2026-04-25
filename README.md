# kurioscreative/skills

A Claude Code plugin marketplace for skills extracted from real sessions.

## Install via Claude Code

```
/plugin marketplace add kurioscreative/skills
/plugin install flow@kurioscreative-skills
```

After install, skills are namespaced as `/flow:agentic`, `/flow:shape`, `/flow:enhance`, `/flow:improve-skill`, `/flow:tighten`.

## Or copy-paste a single skill

Each skill lives in [`skills/<name>/SKILL.md`](./skills) — drop the folder into your `~/.claude/skills/` directory.

## Plugins

| Plugin | Skills                                                    | What it does                                                                  |
| ------ | --------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `flow` | `agentic`, `shape`, `enhance`, `improve-skill`, `tighten` | Opinionated chain: shape intent → execute via agentic decomposition → refine. |

## All available skills

The repo contains every skill (not just those in published plugins). Browse [`skills/`](./skills) to discover what's available; future plugins will bundle them into themed chains.

## Promoting a local skill into the repo

Use `bin/promote-skill` to copy a skill out of your local Claude config into this repo:

```
bin/promote-skill <name>        # from ~/.claude/skills/<name>
bin/promote-skill -p <name>     # from $PWD/.claude/skills/<name>
bin/promote-skill -f <name>     # overwrite if destination already exists
```

The script copies the skill directory into `skills/<name>/`. It does not move the source, touch git, or edit `.claude-plugin/marketplace.json` — register the skill with a plugin manually if you want it bundled.

## License

MIT
