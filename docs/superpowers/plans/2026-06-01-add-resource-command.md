# /add-resource Command Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a `.claude/commands/add-resource.md` slash command that adds a book, article, video, podcast, or person to the correct reading list file with a generated description.

**Architecture:** Single markdown prompt file placed in `.claude/commands/`. Claude Code reads `$ARGUMENTS`, executes the prompt instructions conversationally — fetching URLs, classifying, verifying with the user, generating a description, editing the file, and optionally committing.

**Tech Stack:** Claude Code slash commands (markdown prompt), `eng_leadership.md`, `ai.md`

---

### Task 1: Create `.claude/commands/` directory and command file

**Files:**
- Create: `.claude/commands/add-resource.md`

- [ ] **Step 1: Create the commands directory**

```bash
mkdir -p .claude/commands
```

- [ ] **Step 2: Create `.claude/commands/add-resource.md`**

```markdown
Add a resource to the personal reading list.

**Input:** `$ARGUMENTS` — a book title, person name, or URL (article, video, or podcast)

---

## Your job

Work through these steps in order. Show your reasoning at each classification step so the user can catch errors early.

---

### Step 1 — Parse the input

- If `$ARGUMENTS` is a URL: fetch the page. Extract the title, author/source, and enough content to classify it.
- If `$ARGUMENTS` is a plain title or name: use your training knowledge. Note that you cannot verify the URL.

---

### Step 2 — Classify

Determine:

| Field | Options |
|-------|---------|
| **Type** | book / article / video / podcast / person |
| **Topic** | AI & technology OR engineering leadership |
| **Target file** | `ai.md` (AI/tech topic) OR `eng_leadership.md` (leadership/management) |
| **Section** | See routing table below |

**Section routing:**

`eng_leadership.md`:
- book → `## Books` → `### Read`
- podcast → `## Podcasts`
- article → `## Articles & Links` → pick the best subsection: Performance & Management / Leadership & Behavior / Team Building & Culture / Coaching & Development / Engineering & Product / Motivation & Psychology / Resources & References
- video → `## Videos` → pick subsection: TED Talks / Engineering Leadership / Leadership Development
- person → not applicable (this file has no people section)

`ai.md`:
- book → `## Books` → `### Read`
- podcast → `## Podcasts`
- video → `## Videos` → `### AI Strategy & Leadership`
- person → `## People to Follow`
- article → `## Articles & Links`

---

### Step 3 — Verify placement with the user

Show your classification in one message and ask for confirmation before continuing. Example:

> I'd add this to **eng_leadership.md** under **Articles & Links → Engineering & Product**. Correct?

If the user corrects you, update your classification and confirm the change before moving on.

---

### Step 4 — Ask for extra context (optional)

Ask:

> Anything specific you want included in the description? (Feel free to skip)

If the user provides context, incorporate it into the description. If they skip, proceed.

---

### Step 5 — Generate description

Write a 3-4 sentence paragraph in this style:
1. The core insight or thesis — what does this resource actually argue?
2. Why it's specifically valuable for engineering leaders — what does it unlock or solve?
3. Something concrete and non-obvious — a specific framework, counterintuitive finding, or practice that a reader wouldn't expect.

Avoid generic phrases like "this is a must-read" or "essential for any leader."

---

### Step 6 — Show the formatted entry and get approval

Display the complete markdown block that will be inserted. Use the correct format for the type:

**Book (no URL):**
```
- **Title** - Author Name

  Description paragraph.
```

**Book or article or video or podcast (with URL):**
```
- **[Title](url)** - Author or Source

  Description paragraph.
```

**Person to follow:**
```
- **Name** - Role / Affiliation

  Description paragraph.
```

Ask: "Does this look right? I'll insert it as-is, or tell me what to change."

If the user requests edits, revise and show the block again before proceeding.

---

### Step 7 — Edit the file

Read the target file first to find the exact insertion point.

Insert the entry at the **bottom of the target section** (before the next `##` or `###` heading, or at the end of file if it's the last section). Do not reorder existing entries.

If the section contains only `*Coming soon...*`, replace that line with the new entry (do not keep the placeholder).

---

### Step 8 — Ask about committing

Ask:

> Done. Commit this? (yes / yes + push / no)

- **yes**: `git add <file> && git commit -m "Add <title> to reading list"`
- **yes + push**: commit then `git push`
- **no**: leave the file edited, user commits manually
```

- [ ] **Step 3: Verify the file was created**

```bash
cat .claude/commands/add-resource.md
```

Expected: full prompt content printed to terminal, no errors.

- [ ] **Step 4: Commit**

```bash
git add .claude/commands/add-resource.md
git commit -m "Add /add-resource slash command"
```

---

### Task 2: Smoke-test with a book title

- [ ] **Step 1: Run the command with a plain book title**

In Claude Code terminal, run:

```
/add-resource Thinking in Systems
```

Expected flow:
- Claude classifies as book, `eng_leadership.md`, `## Books → ### Read`
- Shows verification message
- Asks for extra context
- Generates 3-4 sentence description
- Shows formatted entry
- Asks about committing

- [ ] **Step 2: Verify no file edits happened until Step 7**

After approval, confirm the entry appears at the bottom of `### Read` in `eng_leadership.md`.

- [ ] **Step 3: Answer "no" to the commit prompt**

Revert the test edit:

```bash
git checkout eng_leadership.md
```

---

### Task 3: Smoke-test with a URL

- [ ] **Step 1: Run the command with a real article URL**

```
/add-resource https://lethain.com/good-engineering-strategy-is-boring/
```

Expected flow:
- Claude fetches the URL, extracts title ("Good Strategy is Boring" / Will Larson)
- Classifies as article, `eng_leadership.md`, `## Articles & Links → Resources & References` (or similar)
- Runs through full flow

- [ ] **Step 2: Answer "no" to the commit prompt**

Revert test edit:

```bash
git checkout eng_leadership.md
```

- [ ] **Step 3: Confirm error handling — try an unreachable URL**

```
/add-resource https://this-url-does-not-exist-xyz.com/article
```

Expected: Claude notes the URL could not be fetched and asks how to proceed (proceed with title from URL slug, or abort).
