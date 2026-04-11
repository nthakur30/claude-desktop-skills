---
name: doc-it
description: Dual-output documentation workflow. Triggers on "doc it", "document this", "save to notion", "create documentation", or any request to save conversation content as documentation. Generates a downloadable .md file AND creates a Notion page simultaneously. Always use this skill when the user wants to preserve technical docs, project notes, meeting notes, or any knowledge from a conversation.
---

# Doc-It: Dual Documentation Output

When triggered, extract documentation from the conversation and save it in two places:
1. **Markdown file** — saved to Obsidian vault with `[[wiki-links]]`
2. **Notion page** — created in user's chosen database, linked to a project

---

## Step 1: Scan Obsidian and Notion

Run all three scans before asking anything.

**Obsidian — Work Projects:**
```
desktop-commander:list_directory({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Work Projects",
  depth: 1
})
```

**Obsidian — Personal Projects:**
```
desktop-commander:list_directory({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Personal Projects",
  depth: 1
})
```

**Notion — Databases:**
```
Notion:notion-search({ query: "database", query_type: "internal", filters: {}, page_size: 15 })
```
Extract only items where `type: "database"`. Save title and id.

**Notion — Projects list:**
```
Notion:notion-fetch({ id: "collection://1be43898-8065-81de-ada7-000b3fff7492" })
```
Extract all project page titles and URLs from the result.

**Obsidian parsing rules:**
- Only use lines starting with `[DIR]`
- Ignore `[FILE]` entries, `.obsidian`, `.DS_Store`, and hidden folders

---

## Step 2: Ask User for All Details

Ask all questions in one `ask_user_input_v0` call using real data from Step 1.

```
Questions:
1. "What should this doc be titled?" — suggest a title based on conversation + "Custom title"
2. "Where in Notion should I save this?" — options from Notion database scan
3. "Which project should this be linked to?" — options from Projects DB fetch + "None / Skip"
4. "Work or Personal project in Obsidian?" — ["Work Projects", "Personal Projects"]
5. "Which Obsidian folder?" — options from matching Obsidian scan + "➕ Create new folder"
```

If user picks "Custom title" — ask for the title before continuing.
If user picks "➕ Create new folder" — ask for the folder name before continuing.

Store:
- Selected database ID
- Selected project URL (or null if skipped)
- Final title, Obsidian folder path

---

## Step 3: Extract and Structure Content

Write the Obsidian file in this format:

```markdown
---
title: [Title]
date: [YYYY-MM-DD]
project: "[[Project Folder Name]]"
tags: [relevant, tags]
related:
  - "[[Related Doc]]"
---

# [Title]

> **Project**: [[Project Folder Name]]
> **Created**: [Date]

## Overview
[2-3 sentence summary]

## [Main Sections]
[Content from conversation]

## Next Steps
- [ ] [Action items]

---
*Documentation generated [Date] via Claude*
```

Wiki-link rules — always use `[[double brackets]]` for:
- Project names: `[[Claude]]`, `[[Home-Server]]`
- Related docs: `[[API Setup Guide]]`
- Key concepts: `[[MCP]]`, `[[n8n]]`

---

## Step 4: Save to Obsidian

**Path:**
```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Folder]/[filename].md
```

**Filename rules:** lowercase, hyphens for spaces, no special characters, under 50 chars.

**If new folder needed:**
```
desktop-commander:start_process({
  command: "mkdir -p '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[scope]/[folder]'",
  timeout_ms: 5000
})
```

**Write file:**
```
desktop-commander:write_file({ path: "[full path]", content: "[markdown content]", mode: "rewrite" })
```

Chunk into 25-30 line blocks if content is long. Use `mode: "append"` for subsequent chunks.

---

## Step 5: Create Notion Page

**Always fetch the database schema first:**
```
Notion:notion-fetch({ id: "[selected-database-id]" })
```

**Create the page:**
```
Notion:notion-create-pages({
  parent: { type: "data_source_id", data_source_id: "[selected-data-source-id]" },
  pages: [{
    properties: {
      "Title ": "[Document Title]",
      "date:Date:start": "[YYYY-MM-DD]",
      "date:Date:is_datetime": 0,
      "Project": "[selected-project-url]"   // omit entirely if user chose "None / Skip"
    },
    content: "[full markdown content]"
  }]
})
```

**Known schemas:**

Project Notes (collection://28543898-8065-8094-a590-000ba4f44803):
- `Title ` (trailing space) — title
- `date:Date:start` — date
- `Project` — relation to Projects DB

Projects (collection://1be43898-8065-81de-ada7-000b3fff7492):
- `Name` — title
- `Category` — select
- `Status` — status

---

## Step 6: Confirm

```
✅ Documentation saved to both locations:

📁 Obsidian: [Work or Personal Projects]/[Folder]/[filename].md
📓 Notion: [Title] in [Database Name] — [url]
🔗 Linked to project: [Project Name]  (omit this line if None / Skip)
```

---

## Error Handling

- **Desktop Commander unavailable** — create a downloadable .md file instead, skip Obsidian write
- **Notion MCP unavailable** — save to Obsidian only, notify user
- **No databases found** — ask user to provide a Notion database URL
- **No projects found** — skip project linking, notify user
- **Schema mismatch** — create page with title only, notify user

---

## Quick Reference

| User Says | Action |
|-----------|--------|
| "doc it" | Full flow — scan, ask all questions, save both |
| "doc it to Project Notes" | Pre-select that Notion DB, ask rest |
| "doc it for Claude in Personal" | Pre-select Obsidian folder, ask rest |
