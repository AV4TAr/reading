# Design: `/add-resource` Slash Command

**Date:** 2026-06-01  
**Status:** Approved

## Overview

A Claude Code slash command that lets the user add a book, article, video, podcast, or person to the reading list by providing a title or URL. Claude handles classification, description generation, file editing, and optionally committing.

## Invocation

```
/add-resource <title or URL>
```

Examples:
- `/add-resource The Phoenix Project`
- `/add-resource https://www.youtube.com/watch?v=abc123`
- `/add-resource https://hbr.org/2023/04/some-article`

## Target Files

| Topic | File |
|-------|------|
| AI, machine learning, AI strategy | `ai.md` |
| Engineering leadership, management, coaching | `eng_leadership.md` |

## Sections Per File

**`eng_leadership.md`**
- Books → `### Read`
- Podcasts → `## Podcasts`
- Articles → `## Articles & Links` (with subsection: Performance & Management, Leadership & Behavior, Team Building & Culture, Coaching & Development, Engineering & Product, Motivation & Psychology, Resources & References)
- Videos → `## Videos` (TED Talks, Engineering Leadership, Leadership Development)

**`ai.md`**
- People to Follow → `## People to Follow`
- Books → `### Read`
- Podcasts → `## Podcasts`
- Videos → `## Videos → AI Strategy & Leadership`

## Flow

### Step 1 — Parse Input
- If URL: fetch the page to get title, type, and enough content to classify topic and generate description.
- If plain text: treat as book title; use training knowledge to classify and describe.

### Step 2 — Classify
Determine:
- **Type**: book / article / video / podcast / person
- **Topic**: AI & tech OR engineering leadership
- **Section**: specific section within the target file

### Step 3 — Verify Placement
Present classification to user in one message:

> "I'd add this to **eng_leadership.md** under **Articles & Links → Engineering & Product**. Correct?"

If user corrects, update classification and confirm.

### Step 4 — Ask for Extra Context (Optional)
Ask: "Anything specific you want included in the description?"  
User can skip. If they provide context, incorporate it into the description.

### Step 5 — Generate Description
Write a 3-4 sentence paragraph matching the existing style:
1. Core insight or thesis
2. Why it's specifically valuable for engineering leaders
3. Something concrete and non-obvious

### Step 6 — Show Entry for Approval
Display the complete formatted markdown block:

```markdown
- **[Title](url)** - Author (if known)

  Description paragraph here.
```

User confirms or requests edits. If edits requested, revise and show again.

### Step 7 — Edit File
Insert entry at the bottom of the target section. Do not reorder existing entries.

### Step 8 — Ask About Commit
Ask: "Commit this? (yes / yes + push / no)"
- **yes**: `git add <file> && git commit -m "Add <title> to reading list"`
- **yes + push**: commit then `git push`
- **no**: leave file edited, user commits manually

## Entry Format

**Book (no URL):**
```markdown
- **Title** - Author Name

  Description paragraph.
```

**Book with URL:**
```markdown
- **[Title](url)** - Author Name

  Description paragraph.
```

**Article / Video / Podcast:**
```markdown
- **[Title](url)** - Source or Author

  Description paragraph.
```

**Person to follow:**
```markdown
- **Name** - Role / Affiliation

  Description paragraph.
```

## Error Handling

- URL unreachable: proceed with training knowledge, note the URL could not be fetched.
- Cannot classify topic confidently: ask the user which file.
- Cannot classify section confidently: ask the user which section.
