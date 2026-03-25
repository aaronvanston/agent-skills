# Transcript Format

## File Location

Transcripts are stored separately from meeting notes in a `transcripts/` subdirectory of the meetings content folder.

**File naming:** Same slug as the meeting file: `{meeting-slug}.md`

## Frontmatter

```md
---
title: "Meeting Title"
date: YYYY-MM-DD
participants:
  - Name (Organization)
  - Name (Organization)
source: https://recording-url or "Provided directly"
---
```

## Body Format

The body is a conversation with speaker labels. Each speaker turn is a paragraph starting with `Name:` followed by their text.

```md
Speaker Name: Substantive content begins here. This is what they said about the topic.

Another Speaker: Their response to the above point.

Speaker Name: Continuing the discussion with more detail.
```

## Timestamps

Timestamps are optional. If the source includes them, preserve them in any of these formats:

- `[00:01:23] Speaker: text`
- `(00:01:23) Speaker: text`
- `00:01:23 Speaker: text`

## Cleaning Rules

**Replace:** Generic labels ("Me", "Them", "Speaker 1", "Speaker 2") with actual participant names.

**Strip (remove entirely):**
- Personal greetings and introductions
- Small talk and weather chat
- Note-taker and tech setup discussion
- Sign-off pleasantries ("Nice to meet you", "Thanks for your time")
- "Can you hear me?" and similar audio check exchanges

**Retain (keep everything):**
- ALL substantive discussion about domain topics, workflows, pain points
- Product feedback and feature requests
- Opinions about tools and software
- Industry context and background information
- Questions and answers about processes

**Do not** add a "Stripped:" line or document what was removed.

## Rendering Notes

If the wiki app renders transcripts, the first unique speaker is typically left-aligned, subsequent speakers right-aligned with distinct colors. If no speaker labels are detected, the transcript renders as plain paragraphs.
