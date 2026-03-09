# skill-jar

A jar of reusable Agent Skills — use one, put the lid back on, grab it again next time.

Works with **Cursor**, **Codex**, or any editor that supports [Agent Skills](https://agentskills.io).

## Skills

| Skill | Description |
|-------|-------------|
| [feature-slicing](feature-slicing/) | Feature-Sliced Design architecture with strict layer hierarchy and import rules |
| [analyze-complexity](analyze-complexity/) | Frontend project complexity analysis using scc + madge with multi-dimensional scoring |
| [neutralize-prompt](neutralize-prompt/) | Rewrite analysis prompts for deeper, more objective AI output |

## Install

Copy the skill directory you need into your project:

```bash
# Codex
cp -r <skill-name>/ .agents/skills/<skill-name>/

# Cursor
cp -r <skill-name>/ .cursor/skills/<skill-name>/
```

## Recommended Skills

- https://github.com/vercel-labs/agent-skills
