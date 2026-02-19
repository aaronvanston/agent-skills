# Agent Skills

Public skills repo. **No private data, API keys, internal URLs, or company-specific content.**

## Structure

```
AGENTS.md                                # This file
CLAUDE.md                                # Points to AGENTS.md
skills/[skill-name]/SKILL.md             # Required
skills/[skill-name]/references/*.md      # Optional - detailed docs
skills/[skill-name]/rules/*.md           # Optional - enforcement patterns
```

## Install

```bash
npx skills add aaronvanston/agent-skills
```

## Skill Authoring

Follows the [Agent Skills Specification](https://agentskills.io/specification) and the [Anthropic skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

When reviewing or accepting contributed skills, validate against the spec and flag anything that strays significantly - report back to the user with specifics.

### Naming

- Kebab-case: `creating-presentations`, `outline.md`
- Folder name must match `name` in frontmatter
- Name: max 64 chars, lowercase letters, numbers, hyphens only

### Frontmatter

Only `name` and `description`. No other fields.

- `name`: kebab-case identifier
- `description`: max 1024 chars. This is the trigger mechanism - include what it does AND when to use it with specific phrases the user might say

### Body

- Under 500 lines. Split to `references/` if longer.
- Only add what the agent doesn't already know.
- No "When to Use" sections in body (loads after triggering, wasted tokens).
- References one level deep, linked explicitly from SKILL.md.
- Reference files over 100 lines get a table of contents.

### Public Content Only

Everything here is distributed broadly. Never include:
- Private/team-specific context
- API keys, internal URLs, credentials
- Customer data or proprietary business logic
- Company-internal processes

## Rules

Rules are granular, single-concern patterns inside a skill's `rules/` directory. Individual enforcement items the agent applies when relevant. Inspired by [Vercel's approach](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices/rules).

### Rule Structure

```
skills/[skill-name]/rules/
├── _sections.md          # Section ordering, impact levels
├── _template.md          # Template for new rules
└── [section]-[name].md   # Individual rules
```

### Rule Frontmatter

```yaml
---
title: Rule Title
impact: CRITICAL | HIGH | MEDIUM | LOW
tags: tag1, tag2
---
```

Each rule includes a brief explanation, an **Incorrect** code example, and a **Correct** code example.

### Impact Levels

| Level | Meaning |
|-------|---------|
| CRITICAL | Significant performance/correctness risk if ignored |
| HIGH | Strong recommendation, measurable improvement |
| MEDIUM | Best practice, noticeable benefit |
| LOW | Nice to have, minor improvement |

Rules are optional - not every skill needs them. Use when a skill has specific do/don't patterns worth enforcing.

## Gotchas

- SKILL.md without frontmatter won't be recognised
- Reference files not linked from SKILL.md won't be loaded
- Folder name must match `name` in frontmatter
- Description over 1024 chars will be truncated

## Maintenance

- When adding or removing a skill, update README.md skill table
- When renaming folders or reference files, grep CLAUDE.md and all SKILL.md files for stale paths
