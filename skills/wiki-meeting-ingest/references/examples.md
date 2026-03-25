# Examples

## Quote attribution in meeting notes (inline)

```mdx
> "We've been tracking everything in spreadsheets but it falls apart once you have more than three projects." -- Sarah
```

## Quote attribution in feature/product files

```mdx
> "We've been tracking everything in spreadsheets but it falls apart once you have more than three projects." -- [Acme Corp Interview, Jan 15 '26](/meetings/acme-corp-interview-sarah-2026-01-15)
```

## Combining quote fragments

Do this:

```mdx
> "The biggest pain point is visibility ... we don't know where we stand until the end of the month." -- James
```

NOT this (separate fragments):

```mdx
> "The biggest pain point is visibility." -- James

> "We don't know where we stand until the end of the month." -- James
```

## Feature file quote addition

When adding to an existing story:

```mdx
### DASH-03: Multi-project overview

As a project manager, I want a dashboard showing all active projects so I can quickly identify which ones need attention.

**Acceptance criteria:**
- Overview shows status, budget health, and key dates for each project.
- Users can filter and sort by multiple dimensions.

> "I need to see all my projects at a glance without opening each one." -- [Acme Corp Interview, Jan 15 '26](/meetings/acme-corp-interview-sarah-2026-01-15)

> "We've been tracking everything in spreadsheets but it falls apart once you have more than three projects." -- [Beta Industries Feedback, Feb 10 '26](/meetings/beta-industries-feedback-james-2026-02-10)
```

New quotes are appended after existing ones, maintaining chronological order.

## Person profile template

```mdx
---
title: Sarah Chen
summary: Project manager at Acme Corp with experience managing multi-site portfolios.
tags:
  - stakeholder
  - customer interview
categories:
  - Research
  - People
aliases: []
related:
  - meetings/acme-corp-interview-sarah-2026-01-15
section: people
status: sourced
organization: Acme Corp
role: Senior Project Manager
---

## Role

- Organization: Acme Corp
- Role: Senior Project Manager
- Location: Sydney, Australia

## Background

Sarah manages a portfolio of 12 active residential projects for Acme Corp. She has been in the construction industry for 8 years and previously worked at a tier-1 contractor.

## Key Perspectives

- Frustrated with spreadsheet-based tracking across multiple projects.
- Values visibility and real-time status over detailed reporting.
- Currently using a mix of Excel and basic project management tools.

## Meetings

- [Acme Corp Interview, Jan 15 '26](/meetings/acme-corp-interview-sarah-2026-01-15)
```
