---
name: wiki-meeting-ingest
description: >
  Ingest a meeting transcript into a markdown wiki -- creating cleaned transcripts,
  structured meeting notes, person profiles, term stubs, and feature evidence.
  Use when the user asks to "ingest meeting notes," "process a meeting,"
  "add meeting to wiki," "create meeting notes from transcript," "process Granola notes,"
  or provides a transcript to be turned into wiki content. Works with any domain wiki
  that follows the markdown content directory pattern.
---

# Wiki Meeting Ingest

Ingest a meeting transcript and produce a complete set of wiki artifacts: cleaned transcript, structured meeting notes, person profiles, term entries, and feature evidence updates.

## Inputs

The user provides one of:

- **Meeting transcript** -- a pasted or file-based transcript from Granola, Fathom, Otter, or similar.
- **Granola meeting reference** -- a meeting title, date, or URL to fetch via Granola MCP tools (if available).
- **No transcript** -- the user describes the meeting and you ask clarifying questions.

Optional inputs:

- Participant names, roles, and organizations.
- Meeting date, time, timezone, and location.
- Recording URL.
- Specific features or terms to focus on.

## Before You Start

1. **Find the wiki content directory.** Look for a `WRITING_GUIDE.md`, `AGENTS.md`, or a `src/content/` directory structure. Common locations: `apps/wiki/`, `wiki/`, `docs/`, or the repo root.
2. **Read the writing guide** if one exists. It is the canonical source for formatting rules. If anything in this skill contradicts a project-specific writing guide, **the writing guide wins**.
3. **Scan existing meetings** to calibrate depth, tone, and slug conventions:
   - List the meetings directory to see existing file names.
   - Read one or two recent meeting files to match structure.
4. **Identify content directories.** Map out where each content type lives:
   - Meetings (notes and transcripts)
   - People / stakeholder profiles
   - Terms / glossary entries
   - Features / product specs / user stories

---

## Workflow

### Step 1: Obtain the Transcript

**From Granola (if MCP available):** Use Granola MCP tools to find and fetch the meeting transcript.

**From file or paste:** The user provides the text directly.

**If no transcript:** Ask for detailed notes. You can still produce meeting notes without a transcript, but note this in the output.

Extract from the transcript or ask the user for:

- All participant names (full names, organizations, roles)
- Meeting date (YYYY-MM-DD)
- Meeting time and timezone
- Meeting location (e.g., "Remote (video call)", "In-person")
- Recording URL (if available)

### Step 2: Determine the Slug

Build the slug from: `{company-or-topic}-{description}-{person-name}-{YYYY-MM-DD}`

The slug must be kebab-case. Use the primary contact's name unless it's a group session.

### Step 3: Create the Cleaned Transcript

Read [references/transcript-format.md](references/transcript-format.md) for the full format specification.

**Key rules:**
- Replace generic labels ("Me", "Them", "Speaker 1") with real names.
- Strip: personal greetings, small talk, tech setup, sign-off pleasantries.
- Retain: ALL substantive discussion about the domain, workflows, pain points, product feedback.
- Keep speaker turns as `Name:` paragraphs.
- Preserve timestamps if the source includes them.

### Step 4: Create/Update Person Profiles

For each **external** participant (non-team members):

- Check if a person profile already exists.
- If new: create a profile with role, background, and meeting link.
- If existing: add the new meeting to their profile.

### Step 5: Create the Meeting Notes

Read [references/meeting-notes-template.md](references/meeting-notes-template.md) for the full body structure and writing rules.

**Required sections:**
- **Summary** -- 2-4 sentences covering scope and takeaways.
- **Topics Discussed** -- logical topic breakdowns with inline quotes.
- **Key Insights** -- numbered, bold-titled insights.

**Optional sections:**
- **Software Mentioned** -- tools discussed in the meeting.
- **Quotes** -- additional standalone quotes not placed inline.

**Critical rule:** Scatter quotes inline throughout Topics Discussed. Do NOT save all quotes for the end.

### Step 6: Update Feature/Product Files

Scan the transcript for content that supports existing user stories or product specs.

- Add inline quotes to relevant stories with attribution: `> "Quote." -- [Meeting Title, Mon DD 'YY](/meetings/slug)`
- Create new stories if the meeting reveals needs not yet captured.
- Update `related` frontmatter to link the meeting.

### Step 7: Identify and Create New Terms

Review the transcript for domain terms, tools, or concepts not yet in the wiki.

**Criteria for a new term:**
- Specific domain concept (not a generic word).
- Came up substantively in the conversation.
- Would be useful for auto-linking in other wiki content.

For each new term, create a baseline entry with: title, summary, what it is, how it's used, and why it matters. If the wiki-term-research skill is available, delegate to it for a deeper entry.

### Step 8: Cross-Reference Everything

Ensure bidirectional linking:
- Meeting notes `related` references people, features, and terms.
- Person profiles `related` references the new meeting.
- Feature files `related` references the meeting (only if evidence was added).
- Term entries `related` references related terms (only strong connections).

---

## Quality Checklist

- [ ] Transcript file exists with speaker labels and cleaned content.
- [ ] Meeting notes have complete frontmatter.
- [ ] Summary is 2-4 sentences and stands alone.
- [ ] Topics Discussed has 4+ sections with inline quotes.
- [ ] Quotes are scattered inline, not bunched at the bottom.
- [ ] Key Insights are numbered with bold titles.
- [ ] Person profiles created/updated for external participants.
- [ ] Feature files updated where evidence exists.
- [ ] New terms created for substantive concepts not already in the wiki.
- [ ] All `related` links are bidirectional.
- [ ] File names are kebab-case.

---

## Output Summary

After completing the workflow, report what was created and updated:

- Files created (meetings, transcripts, people, terms)
- Files updated (people, features, terms)
- Feature evidence added (which stories, which quotes)
- Notes on any decisions or items needing user review

---

## Reference Files

- **[references/meeting-notes-template.md](references/meeting-notes-template.md)** -- Full body structure and writing rules for meeting notes.
- **[references/transcript-format.md](references/transcript-format.md)** -- Transcript cleaning and formatting specification.
- **[references/examples.md](references/examples.md)** -- Quote formatting examples for meeting notes and feature files.
