---
name: wiki-setup
description: >
  Set up a markdown-based wiki knowledge base in a repository. Creates the directory
  structure, writing guide, AGENTS.md configuration, and initial content templates.
  Use when the user asks to "set up a wiki," "create a knowledge base," "start a wiki,"
  "initialize wiki content," "set up a second brain," or wants to create a structured
  markdown content system for learning and research in their project.
---

# Wiki Setup

Initialize a markdown-based wiki knowledge base in any repository. Creates the content directory structure, writing guide, agent configuration, and starter templates.

## Inputs

- **Domain** (optional): The subject area for the wiki (e.g., "healthcare compliance", "fintech", "real estate"). If not provided, set up a generic structure.
- **Location** (optional): Where in the repo to place wiki content. Default: `wiki/` at the repo root.
- **Scope** (optional): "minimal" (terms + meetings only) or "full" (all content types). Default: minimal.

## Workflow

### Step 1: Discover Existing Structure

Before creating anything, check what already exists:

- Look for existing content directories, AGENTS.md, WRITING_GUIDE.md, or markdown content.
- If a structure exists, adapt to it rather than overwriting.
- Ask the user before making changes to existing files.

### Step 2: Create Directory Structure

Read [references/directory-structure.md](references/directory-structure.md) for the full layout.

**Minimal setup:**

```
{location}/
  content/
    terms/          # Domain concepts and definitions
    meetings/       # Customer/stakeholder interview notes
      transcripts/  # Raw cleaned transcripts
    people/         # Stakeholder profiles
    product/        # User stories, features, specs
  WRITING_GUIDE.md  # Content authoring standards
```

**Full setup** adds:

```
    learning/       # Structured curriculum modules
    insights/       # Synthesized research findings
    competitors/    # Market analysis
```

### Step 3: Create the Writing Guide

Read [references/writing-guide-template.md](references/writing-guide-template.md) and generate a `WRITING_GUIDE.md` adapted to the user's domain.

The writing guide should cover:
- Content structure and directory layout
- File naming conventions
- Frontmatter template for each content type
- Writing style rules
- Alias and auto-linking guidance
- Quality checklist

### Step 4: Configure AGENTS.md

Create or update `AGENTS.md` in the wiki directory to point agents to:
- The writing guide as the primary reference
- Content directories and their purposes
- Key patterns for each content type
- Style rules (e.g., no em-dashes)

### Step 5: Create Starter Templates

For each content type in the setup, create one example file showing the expected format:
- `content/terms/_example.mdx` -- a sample term entry
- `content/meetings/_example.mdx` -- a sample meeting note
- `content/people/_example.mdx` -- a sample person profile
- `content/product/_example.mdx` -- a sample feature/user story

Prefix examples with `_` so they sort to the top and are clearly templates.

### Step 6: Report

Tell the user what was created and suggest next steps:
- Install the wiki-meeting-ingest skill for processing transcripts
- Install the wiki-term-research skill for deep term entries
- Start by adding terms for concepts they don't fully understand
- Add their first meeting transcript

---

## Reference Files

- **[references/directory-structure.md](references/directory-structure.md)** -- Full directory layout with explanations for each content type.
- **[references/writing-guide-template.md](references/writing-guide-template.md)** -- Template for generating a project-specific writing guide.
