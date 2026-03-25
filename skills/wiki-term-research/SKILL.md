---
name: wiki-term-research
description: >
  Create deeply researched wiki term entries with web research, practical examples,
  and cross-references. Use when the user asks to "create a wiki term," "write a term entry,"
  "add a glossary definition," "research a concept," "define this term," or when a meeting
  ingestion identifies terms to create. Produces comprehensive entries with external citations,
  worked examples, and detailed breakdowns. Works with any domain wiki using markdown content.
---

# Wiki Term Research

Create production-quality, deeply researched wiki term entries that serve as both quick reference and comprehensive learning resource. Every term goes through mandatory web research, cross-reference validation, and quality checks.

## Inputs

- **Term name**: The concept to research and document.
- **Domain context** (optional): The industry or subject area for the wiki (e.g., "construction finance", "healthcare compliance", "SaaS metrics").
- **Depth override** (optional): User may request "quick" (abbreviated) or "comprehensive" (maximum depth). Default: comprehensive.

## Before You Start

1. **Find the wiki content directory.** Look for a `WRITING_GUIDE.md`, `AGENTS.md`, or a `src/content/` directory structure.
2. **Read the writing guide** if one exists. It is the canonical source for formatting rules. If anything in this skill contradicts a project-specific writing guide, **the writing guide wins**.
3. **Scan existing terms** to calibrate depth, tone, and conventions:
   - List the terms directory to see existing file names and categories.
   - Read 2-3 related terms to understand the expected quality level.

---

## Workflow

### Phase 1: Orientation and Existing Content Audit

1. **Search for existing entries** on this topic. Check if the term already exists (update vs. create).
2. **Scan existing categories and tags** by reading related terms to understand conventions.
3. **Determine the slug** -- use kebab-case, descriptive, with regional suffixes only when meaning materially differs.
4. **Read 3-5 closely related entries** to understand gaps and cross-reference opportunities.

### Phase 2: Web Research (MANDATORY)

**This phase is NOT optional.** Every term entry must be backed by web research.

Read [references/research-guide.md](references/research-guide.md) for search query patterns and source quality hierarchy.

At minimum:
- Run **3+ WebSearch queries** to find authoritative definitions, industry standards, and practical guidance.
- **Fetch 1-3 authoritative pages** via WebFetch to extract exact definitions, requirements, and examples.

### Phase 3: Compose the Entry

Read [references/article-template.md](references/article-template.md) for the full template and usage notes.

**Mandatory sections:**
- **TLDR** -- 2-4 sentences. What is it? Why does it matter?
- **What it is** -- clear definition expanding on the summary.
- **How it works** -- mechanics, process, or application.
- **Practical example** -- concrete worked example with realistic data.
- **External references** -- minimum 3 links to reputable sources.

**Optional sections (include when relevant):**
- Regional/jurisdictional detail
- Standard provisions or specifications
- Common pitfalls
- Industry role / why it matters for the product

### Phase 4: Cross-Reference Validation

1. **Verify all `related` slugs exist.** If a related slug doesn't exist, include it but note it as a missing term.
2. **Check aliases won't cause false auto-linking.** Avoid single common words. Multi-word phrases and specific acronyms are safe.
3. **Update related terms** -- check whether 2-3 closely related existing terms should add this new term to their `related` list.

### Phase 5: Quality Validation

**Frontmatter checklist:**
- [ ] Title is specific and matches the slug
- [ ] Summary is exactly one sentence, plain language
- [ ] Tags are concrete and searchable (5-8 tags)
- [ ] Categories use an existing category from the wiki
- [ ] Aliases are precise, no generic words
- [ ] Related has 3-6 entries using slug format

**Content checklist:**
- [ ] TLDR present and useful in 15 seconds
- [ ] At least one practical example with realistic data
- [ ] At least 3 external references with URLs
- [ ] Entry reads cleanly with `##`/`###` hierarchy
- [ ] A reader can get the gist in 60 seconds

**Research checklist:**
- [ ] At least 3 WebSearch queries were executed
- [ ] At least 1 authoritative source was fetched and read
- [ ] No unsourced claims about requirements or standards

### Phase 6: Missing Term Identification

After writing the entry, review concepts mentioned in the body that are NOT in the wiki.

Output a "Suggested future terms" list with term name and a one-sentence justification for each.

---

## Reference Files

- **[references/article-template.md](references/article-template.md)** -- Full article template with all sections and usage notes.
- **[references/research-guide.md](references/research-guide.md)** -- Search query patterns, source quality hierarchy, and research methodology.
