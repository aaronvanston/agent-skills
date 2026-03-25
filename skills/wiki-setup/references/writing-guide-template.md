# Writing Guide Template

Generate a project-specific `WRITING_GUIDE.md` based on this template. Adapt the domain references, categories, and examples to match the user's project.

## Template

```markdown
# How to Write a Wiki Article

This guide standardizes wiki entries so they read consistently and cross-reference cleanly.

## Quick start

1. Create a new MDX file in the correct content directory using kebab-case.
2. Fill out frontmatter (title, summary, tags, categories, aliases, related).
3. Write the entry using the recommended structure.
4. Keep aliases deliberate so auto-linking stays accurate.

## Content structure

**Terms (primary knowledge base):**
- `content/terms/` -- canonical definitions and domain concepts.

**Research:**
- `content/meetings/` -- interview and meeting notes.
- `content/people/` -- stakeholder profiles.
- `content/insights/` -- synthesized findings and themes.

**Product:**
- `content/product/` -- user stories, features, specs, strategy.

## File naming

- Use kebab-case, descriptive slugs.
- Slugs only need to be unique within a section.

## Frontmatter template

---
title: Example Title
summary: One-sentence summary of the term and why it matters.
tags:
  - primary keyword
  - secondary keyword
categories:
  - {Domain Category}
aliases:
  - Common synonym
  - Acronym
related:
  - related-term-slug
---

### Frontmatter guidance

- **title**: Clear, user-facing name. Include acronym in parentheses if common.
- **summary**: 1 sentence, plain language.
- **tags**: Searchable keywords; keep them concrete.
- **categories**: One or two broad groupings.
- **aliases**: Synonyms and abbreviations. Avoid overly generic words.
- **related**: 2-6 closest concepts using slug format.

## Writing style

- **No em-dashes.** Use -- (double hyphen) instead. Em-dashes read as AI-generated text.
- Keep summaries to a single sentence.
- Use ## for main sections, ### for subsections.
- Use --- horizontal rules between major sections.
- Tables for comparisons and structured data.

## Quality checklist

- [ ] Title is specific and matches the slug.
- [ ] Summary is a single sentence.
- [ ] Tags and categories are consistent with existing entries.
- [ ] Aliases are precise and not too generic.
- [ ] A reader can understand the concept in under 60 seconds.
```

## Customization Notes

When generating the writing guide:

1. **Replace `{Domain Category}` with real categories** from the user's domain.
2. **Add domain-specific sections** if the wiki covers regulated or jurisdictional content.
3. **Include meeting note templates** if the user will be ingesting transcripts.
4. **Include feature/product templates** if the user will track user stories.
5. **Keep it under 200 lines** -- the writing guide should be a quick reference, not a manual.
