# Wiki Directory Structure

## Minimal Setup

```
wiki/
  content/
    terms/              # Domain concepts, definitions, glossary entries
    meetings/           # Interview and meeting notes
      transcripts/      # Cleaned raw transcripts (stored separately)
    people/             # Stakeholder and contact profiles
    product/            # User stories, features, specs, strategy
  WRITING_GUIDE.md      # Content authoring standards and templates
  AGENTS.md             # Agent configuration for wiki content
```

## Full Setup

```
wiki/
  content/
    terms/              # Domain concepts, definitions, glossary entries
    meetings/           # Interview and meeting notes
      transcripts/      # Cleaned raw transcripts
    people/             # Stakeholder and contact profiles
    product/            # User stories, features, specs, strategy
    learning/           # Structured curriculum modules
      00-getting-started/
      01-{first-topic}/
    insights/           # Synthesized research findings and themes
    competitors/        # Market analysis and competitor research
    workflows/          # Process documentation
    glossary/           # Quick-reference term groups
  WRITING_GUIDE.md
  AGENTS.md
```

## Content Type Descriptions

### Terms
Canonical definitions for domain-specific concepts. Each term gets its own MDX file with frontmatter (title, summary, tags, categories, aliases, related). Terms are the backbone of the wiki -- they auto-link when mentioned in other content.

### Meetings
Structured notes from customer interviews, stakeholder conversations, and discovery calls. Each meeting links to participants (people), referenced terms, and derived features. Transcripts are stored separately in `transcripts/` to keep notes focused.

### People
Profiles for external stakeholders, interview participants, and key contacts. Each profile links to meetings they participated in and features they influenced.

### Product
User stories, feature specs, epics, and strategy documents. Features accumulate evidence from meetings over time -- quotes are sourced and attributed. Use a `kind` field in frontmatter to distinguish between epics, features, specs, and strategy.

### Learning
Structured educational content organized into numbered modules. Each module contains ordered lessons. Useful for onboarding team members or systematically learning a new domain.

### Insights
Synthesized findings that emerge from multiple meetings or research efforts. Cross-cutting themes that don't belong to a single meeting.

### Competitors
Market analysis with factual, neutral descriptions of competitor products, strengths, limitations, and market gaps.

## Frontmatter Fields (Common Across Types)

```yaml
title: Human-readable name
summary: One sentence, plain language
tags: [searchable keywords]
categories: [broad groupings]
aliases: [synonyms, abbreviations -- keep precise]
related: [slugs of related entries]
section: {auto-derived from folder, override if needed}
status: {sourced | draft | in-progress | done}
```

## File Naming

- Kebab-case for all files: `progress-claim.mdx`, `acme-interview-sarah-2026-01-15.mdx`
- MDX extension for content files (supports JSX components)
- MD extension for transcripts (plain markdown)
- Template/example files prefixed with `_`: `_example.mdx`
