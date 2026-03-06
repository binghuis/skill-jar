---
name: skill-gen
description: >-
  Scaffold a new Agent Skill at .agents/skills/ with standardized structure and semantic naming.
  Use when user says "create skill", "new skill", "add skill", "创建 skill", "新建 skill", or scaffold a skill template.
---
# Skill Gen

Scaffold a new Agent Skill at `.agents/skills/<name>/`.

## Instructions

1. Ask the user for the skill's **purpose** — encourage detail and concrete use-case examples.
2. If the purpose is too vague to derive a meaningful name and triggers, ask the user to clarify with specific examples before proceeding.
3. Derive `name`, `triggers`, `description`, and `resources` from the purpose using the Derivation Rules below.
4. Create directory `.agents/skills/<name>/`.
5. Generate `SKILL.md` using the Template below. Fill every placeholder — do not leave empty sections.
6. Create `scripts/`, `references/`, `assets/` subdirectories only when inferred from purpose.
7. Run the Quality Checklist. Fix any failing items before finishing.

## Derivation Rules

| Field | How to derive |
|-------|---------------|
| **name** | kebab-case, 1-64 chars, verb-led (e.g. `review-code`). No leading/trailing/consecutive hyphens. Reject generic names: helper, utils, tools, misc. Directory name = name field. |
| **triggers** | Infer activation keywords in both English and Chinese. Include at least 3 triggers. |
| **description** | Third-person. WHAT (capabilities) + WHEN (trigger scenarios) + trigger keywords. Max 1024 chars. All "when to use" info goes here — not in the body. |
| **resources** | `scripts/` if repeated executable code; `references/` if docs/schemas; `assets/` if templates/images/data. Omit directories not needed. |

### Example Derivation

**User purpose:** "帮我审查 PR 代码质量和安全性"

| Field | Result |
|-------|--------|
| name | `review-code` |
| triggers | "review code", "code review", "审查代码", "代码审查", "PR review" |
| description | Review code for quality, security, and best practices following team standards. Use when user says "review code", "code review", "审查代码", "代码审查", or wants code quality feedback on pull requests. |
| resources | (none — text-instruction skill, no scripts needed) |

## SKILL.md Template

```markdown
---
name: {{name}}
description: >-
  {{description}}
  Use when user says "{{trigger1}}", "{{trigger2}}", "{{trigger3}}", or {{trigger_summary}}.
---

# {{Title}}

## Instructions

[Step-by-step procedural guidance in imperative form. Tell the agent exactly what to do.]

## Workflow

[Execution flow for the core task. Break into numbered steps.]

## Resources

[Reference bundled scripts/, references/, assets/ here. Remove this section if none.]
```

## Quality Checklist

- [ ] `name` is kebab-case, verb-led, 1-64 chars; dir name matches
- [ ] `name` is not generic (helper, utils, tools, misc)
- [ ] `description` is third-person
- [ ] `description` has WHAT + WHEN + trigger keywords, under 1024 chars
- [ ] `description` includes both English and Chinese triggers
- [ ] SKILL.md body uses imperative form
- [ ] SKILL.md body under 500 lines
- [ ] No empty sections — every section has content
- [ ] Resource directories created only when needed
