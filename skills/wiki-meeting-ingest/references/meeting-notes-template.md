# Meeting Notes Body Template

Use this template when composing the meeting notes body.

## Frontmatter Template

```mdx
---
title: {Company} {Type} -- {Person Name}
summary: {1-2 sentences covering the scope and main takeaways.}
tags:
  - interview
  - discovery
  - {company name lowercase}
  - {key topic tags}
categories:
  - Research
  - Meetings
jurisdictions: []
aliases: []
related:
  - people/{person-slug}
  - product/{feature-slug}
  - terms/{term-slug}
participants:
  - people/{person-slug}
meetingDate: YYYY-MM-DD
meetingTime: "HH:MM AM/PM"
meetingTimezone: "{timezone}"
meetingLocation: "{location}"
recordingUrl: "{URL or omit}"
transcriptFile: meetings/transcripts/{slug}.md
section: meetings
status: sourced
---
```

**Title format examples:**
- `{Company} Interview -- {Person}` for discovery sessions
- `{Company} {Topic} Feedback -- {Person}` for feedback sessions
- `{Company} {Topic} Review -- {Person}` for reviews
- `{Company} {Workshop Type}` for group workshops
- Do NOT include the date in the title.

## Body Structure

```mdx
## Summary

{2-4 sentences. What was covered, key themes, main takeaways. Write so someone can read this alone and understand the meeting.}

---

## Topics Discussed

### {Topic Name}

{Detailed prose: what was discussed, current state, pain points, desired outcomes. Not bullet points -- write in paragraphs.}

> "{Inline quote -- max ~25 words, combine fragments with ellipses if needed.}" -- {Speaker first name}

{Continue with more context, specifics. What are they doing today? What's broken? What would be better?}

---

### {Another Topic}

{Same pattern. Use `---` separators between major topics.}

---

## Key Insights

1. **{Bold insight title}**: {One-sentence summary of the insight and why it matters.}
2. **{Another insight}**: {Continue numbering.}

---

## Software Mentioned

- **{Tool Name}**: {Brief description -- how they use it, what they think of it, pain points.}

---

## Quotes

{Additional quotes that did not fit inline above. Only include this section if there are meaningful standalone quotes remaining. If all quotes are placed inline, omit this section entirely.}

> "{Quote that captures a key sentiment.}"
```

## Writing Rules

1. **Summary**: 2-4 sentences. Someone should understand the meeting from this alone.
2. **Topics Discussed**: Break the conversation into logical topics (typically 4-10 per meeting). Each topic gets its own `###` heading.
   - Write in prose paragraphs, not bullet points.
   - Include specific details: company names, project names, amounts, counts.
   - Explain the current state before the desired state.
3. **Quotes -- the most important formatting rule:**
   - Scatter quotes inline throughout Topics Discussed, placed right after the context they support.
   - Do NOT save all quotes for the end section.
   - Keep quotes concise: aim for 25 words or fewer.
   - If a quote needs trimming, use ellipses: `"First part ... second part."`
   - Combine two fragments of the same thought into one quote with ellipses.
   - Quotes must stand alone -- add context so they make sense without the transcript.
   - Attribution in Topics Discussed is just `-- {Speaker first name}`.
   - A topic can have 1-3 inline quotes. More than 3 means you should split the topic.
4. **Key Insights**: Numbered list, each with a bold title and one-sentence explanation. Typically 5-10 per meeting.
5. **Software Mentioned**: Only include tools actually discussed.
6. **Quotes section at bottom**: Only leftover quotes that are strong enough to preserve but did not fit a topic.
7. **No timestamps** in the notes.
8. **No "Source" section** -- metadata is in frontmatter.
