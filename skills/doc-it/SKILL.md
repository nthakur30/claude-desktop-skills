---
name: doc-it
description: "Obsidian documentation workflow. Triggers on \"doc it\", \"document this\", \"save to obsidian\", \"create documentation\", or any request to save conversation content as documentation. Creates a new note in the vault OR appends to an existing note. Always use this skill when the user wants to preserve technical docs, project notes, meeting notes, or any knowledge from a conversation."
---

# Doc-It: Obsidian Documentation Workflow

When triggered, extract documentation from the conversation and save it to the Obsidian vault — either as a **new note** or **appended to an existing one**.

**Vault root:**
```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes
```

---

## Step 1: Determine Mode

Ask the user one question using `ask_user_input_v0`:

```
"New note or add to an existing one?"
Options: ["📄 New note", "✏️ Add to existing note"]
```

Branch to **Step 2A** or **Step 2B** based on answer.

---

## Step 2A: New Note — Collect Details

Ask all questions in a single `ask_user_input_v0` call:

```
Q1: "What should this note be titled?"
    → Suggest a title based on the conversation + "Custom title"

Q2: "Where should it go?"
    → Options built from the vault structure below
    → Default option always shown first: "📥 00-inbox (sort later)"

Q3: "Any tags to add?" (multi-select)
    → Suggest relevant tags based on conversation content
    → Always include "untagged" as an option
```

If user picks **"Custom title"** → ask for it before continuing.
If user picks a **parent folder** with subfolders (e.g. "Dev & Architecture") → follow up with subfolder options from the map below.

**Store:** final title, full destination folder path, tags list.

---

## Step 2B: Existing Note — Search, Read, and Place

**Ask:** "What's the name or topic of the note you want to add to?"

### 2B-1: Find the note

Run both searches:

```
desktop-commander:start_process({
  command: "find '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes' -name '*.md' | xargs grep -li '[search term]' 2>/dev/null | head -20",
  timeout_ms: 8000
})
```

```
desktop-commander:start_process({
  command: "find '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes' -iname '*[search term]*' -name '*.md' | head -20",
  timeout_ms: 8000
})
```

Deduplicate. Present matches in `ask_user_input_v0`. If no matches → ask user to clarify or offer to create a new note instead.

**Store:** full path of selected file.

### 2B-2: Read and analyze the note

```
desktop-commander:read_file({ path: "[selected file path]" })
```

Parse the file structure:
- Extract all headings (`#`, `##`, `###`) and their line numbers
- Identify existing sections and what they contain
- Note any `## Next Steps` or checklist sections

### 2B-3: Determine where new content belongs

Use the conversation content + the note's existing structure to decide placement:

| New content type | Where to insert |
|---|---|
| Expands on an existing section topic | Inside that section, after the last line of it |
| A new distinct concept or session | New `##` section, before `## Next Steps` if it exists |
| Action items / tasks | Into existing `## Next Steps` checklist, not a new one |
| Corrections or updates to a specific fact | Inline, replacing or annotating the relevant line |
| Completely unrelated to existing sections | New section at bottom, before the footer line |

**Do not** create duplicate sections. If `## Setup` already exists, add to it — don't make `## Setup (Updated)`.

**Store:** insertion point (line number or anchor text), content to insert, edit strategy (`str_replace` or `append`).

---

## Vault Folder Map

Use this to build destination options in Step 2A. Only show `[DIR]` entries.

```
00-inbox/
01-knowledge/
  AI & Claude Code/
    AI Fundamentals/
    CrewAI/
    AutoGen/
    LangChain/
    Microsoft GenAI/
    Pydantic/
    RAG/
  Business/
  Dev & Architecture/
    Data Engineering/
      Airflow / DBT / PySpark / Snowflake / SQL / Structure Questions
    FastAPI & Docker/
      Docker / FastAPI
    Git & GitHub/
    Python & Data/
      Machine Learning / Python
  n8n & Automation/
02-personal/
  Capstone / Claude / H2AI / Home Assistant / Home Server /
  Jarvis / MedFolio / PulsePathAI / RASPI5-n8n / Trading Algo
03-work/
  MCP / PDF Extraction Agent
04-clients/
05-daily/
06-archive/
```

---

## Step 3: Extract and Structure Content

### For a NEW note, write in this format:

```markdown
---
title: [Title]
date: [YYYY-MM-DD]
tags: [tag1, tag2]
related:
  - "[[Related Note]]"
---

# [Title]

> **Created**: [Date]

## Overview
[2–3 sentence summary of the conversation]

## [Main Sections]
[Content extracted from conversation — use headers, bullets, code blocks as needed]

## Next Steps
- [ ] [Action items if any]

---
*Documented [Date] via Claude*
```

### For ADDING TO an existing note:

Do NOT dump a dated block at the bottom. Instead:
- Read the note (Step 2B-2)
- Identify which existing section the new content belongs in
- Write content that matches the voice, formatting, and style of that section
- Use `edit_block` to insert it precisely (Step 4)

New content should feel like it was always part of the note — not a tacked-on addendum.

**Wiki-link rules** — always use `[[double brackets]]` for:
- Project/tool names: `[[n8n]]`, `[[Claude]]`, `[[Home-Server]]`, `[[MCP]]`
- Related notes: `[[API Setup Guide]]`, `[[RASPI5-n8n]]`
- Key concepts: `[[RAG]]`, `[[FastAPI]]`, `[[Docker]]`

**Filename rules (new notes only):** lowercase, hyphens for spaces, no special characters, max 50 chars. Example: `mcp-server-setup-guide.md`

---

## Step 4: Write to Obsidian

### New note:

**If new folder needed first:**
```
desktop-commander:start_process({
  command: "mkdir -p '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder path]'",
  timeout_ms: 5000
})
```

**Write the file:**
```
desktop-commander:write_file({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[filename].md",
  content: "[first chunk — max 30 lines]",
  mode: "rewrite"
})
```

Chunk into 25–30 line blocks for long content. Use `mode: "append"` for all subsequent chunks.

### Editing an existing note:

**Never overwrite the whole file.** Use `desktop-commander:edit_block` to make surgical edits:

**To insert content inside an existing section** — use `str_replace` mode, targeting the last meaningful line of that section as the anchor:

```
desktop-commander:edit_block({
  file_path: "[full path]",
  old_string: "[last line of the target section]",
  new_string: "[last line of the target section]\n\n[new content to add]"
})
```

**To add a new section before `## Next Steps`** (or before the footer `---` line if no Next Steps exists):

```
desktop-commander:edit_block({
  file_path: "[full path]",
  old_string: "## Next Steps",
  new_string: "## [New Section Title]\n\n[content]\n\n## Next Steps"
})
```

**To add tasks into an existing `## Next Steps` checklist:**

```
desktop-commander:edit_block({
  file_path: "[full path]",
  old_string: "## Next Steps\n",
  new_string: "## Next Steps\n- [ ] [new task]\n"
})
```

**Rules:**
- `old_string` must be a unique snippet from the file — include 1–2 lines of surrounding context if needed to ensure uniqueness
- Never include line number prefixes in `old_string`
- Re-read the file before each edit if making multiple changes — prior reads become stale after any write
- If the file has no clear structure to anchor to, append a new dated section at the very bottom (before `*Documented...` footer if present)

---

## Step 5: Confirm

```
✅ Saved to Obsidian

📁 Path: [folder]/[filename].md
🏷️ Tags: [tags]                        ← new notes only
✏️ Updated: [title] — [section edited] ← existing notes only
```

---

## Error Handling

| Situation | Action |
|---|---|
| Desktop Commander unavailable | Generate downloadable `.md` file instead, notify user |
| No search results found | Ask user to clarify or offer to create a new note |
| Destination folder doesn't exist | Create it with `mkdir -p`, then write |
| File write fails | Retry once; if it fails again, offer downloadable `.md` |

---

## Quick Reference

| User Says | Action |
|---|---|
| "doc it" | Ask new vs existing, then full flow |
| "doc it to inbox" | Pre-select `00-inbox`, ask title + tags |
| "doc it to RASPI5" | Pre-select `02-personal/RASPI5-n8n`, ask title + tags |
| "add this to my n8n note" | Search for n8n notes, confirm, append |
| "doc it for clients" | Pre-select `04-clients`, ask title + tags |
