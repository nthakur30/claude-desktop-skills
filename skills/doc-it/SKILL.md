---
name: doc-it
description: Dual-output documentation workflow. Triggers on "doc it", "document this", "save to notion", "create documentation", or any request to save conversation content as documentation. Generates a downloadable .md file AND creates a Notion page simultaneously. Always use this skill when the user wants to preserve technical docs, project notes, meeting notes, or any knowledge from a conversation.
---

# Doc-It: Dual Documentation Output

When triggered, this skill extracts documentation from the conversation and outputs it in two formats:
1. **Markdown file** — saved directly to Obsidian vault with `[[wiki-links]]` for graph connections
2. **Notion page** — created in user's chosen database

## Trigger Phrases

Activate when user says any of:
- "doc it"
- "document this"
- "save to notion"
- "create documentation"
- "save this as a doc"
- "make a doc from this"
- "add this to my notes"

## Step 1: Scan BOTH Obsidian Vault AND Notion

**Scan Obsidian for folders:**
```javascript
// Scan Work Projects
desktop-commander:list_directory({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Work Projects",
  depth: 1
})

// Scan Personal Projects
desktop-commander:list_directory({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Personal Projects", 
  depth: 1
})
```

**Scan Notion for databases:**
```javascript
// Search for databases the user can save to
Notion:notion-search({
  query: "database",
  query_type: "internal",
  filters: {},
  page_size: 15
})
```

From Notion search results:
- Extract items where `type: "database"`
- Collect their `title` and `id` for the options list
- Ignore pages (type: "page")


### Parse Obsidian Results STRICTLY

From the Obsidian scan:
- **ONLY** extract lines starting with `[DIR]`
- **IGNORE** `[FILE]` entries — these are loose files, not folders
- **IGNORE** `.obsidian`, `.DS_Store`, and hidden folders

### Parse Notion Results

From the Notion search:
- Extract only items with `"type": "database"`
- Save both `title` and `id` for each database
- Common databases found:
  - "Project Notes" — main documentation database
  - "✅ To-Do Database" — tasks
  - "Projects" — project tracker

## Step 2: Ask User for ALL Details

Use `ask_user_input_v0` with **ACTUAL options from both scans**.

```json
{
  "questions": [
    {
      "question": "What should this doc be titled?",
      "type": "single_select",
      "options": ["[Suggested title]", "Custom title"]
    },
    {
      "question": "Where in Notion should I save this?",
      "type": "single_select",
      "options": ["[DATABASES FOUND IN NOTION SCAN]"]
    },
    {
      "question": "Work or Personal project in Obsidian?",
      "type": "single_select",
      "options": ["Work Projects", "Personal Projects"]
    },
    {
      "question": "Which Obsidian folder?",
      "type": "single_select",
      "options": ["[FOLDERS FOUND IN OBSIDIAN SCAN]", "➕ Create new folder"]
    }
  ]
}
```


### Dynamic Notion Database Options Example

If Notion scan found these databases:
- Project Notes (id: 28543898-8065-806e-8a63-f3fcec25888f)
- ✅ To-Do Database (id: 27043898-8065-8107-a846-ef5b91c1637e)
- Projects (id: 1be43898-8065-81ab-b685-ddbde476e5f8)

Show:
```json
{
  "question": "Where in Notion should I save this?",
  "options": ["Project Notes", "✅ To-Do Database", "Projects"]
}
```

**Store the database ID** for the selected option to use when creating the page.

### Handling Custom Title

If user selects "Custom title", ask:
> What title do you want for this doc?

### Handling New Folder

If user selects "➕ Create new folder", ask:
> What should the new folder be called?

## Step 3: Extract & Structure Content

Pull relevant content from the conversation. Structure as:

```markdown
---
title: [Document Title]
date: [YYYY-MM-DD]
project: "[[Project Folder Name]]"
tags: [relevant, tags, here]
related:
  - "[[Related Doc 1]]"
---

# [Document Title]

> **Project**: [[Project Folder Name]]
> **Created**: [Date]

## Overview
[2-3 sentence summary]

## [Main Sections]
[Content from conversation]

## Key Points
- [Bullet summary]

## Related
- [[Project Folder Name]] — Parent project

## Next Steps
- [ ] [Action items]

---
*Documentation generated [Date] via Claude*
```


### Wiki-Link Rules for Obsidian Graph

**ALWAYS use `[[double brackets]]` for:**
- Project folder names: `[[Claude]]`, `[[Home-Server]]`
- Related documents: `[[API Setup Guide]]`
- Concepts: `[[n8n]]`, `[[Notion API]]`, `[[MCP]]`

## Step 4: Save to Obsidian Vault

### Path Construction
```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Project Folder]/[filename].md
```

### Create Folder if New
```javascript
desktop-commander:start_process({
  command: "mkdir -p '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[project]'",
  timeout_ms: 5000
})
```

### Filename Rules
- Lowercase, hyphens instead of spaces
- Remove special characters
- Keep under 50 chars

### Write File
```javascript
desktop-commander:write_file({
  path: "/.../[folder]/[project]/[filename].md",
  content: "---\ntitle: ...\n---\n\n# Content...",
  mode: "rewrite"
})
```


## Step 5: Create Notion Page (in USER-SELECTED database)

**CRITICAL: Use the database ID from the user's selection in Step 2.**

### Fetch Database Schema First

Before creating a page, fetch the selected database to get its schema:

```javascript
Notion:notion-fetch({
  id: "[selected-database-id]"
})
```

This returns the database properties (columns). Use these to set the correct property names.

### Common Database Schemas

**Project Notes** (collection://28543898-8065-8094-a590-000ba4f44803):
- `Title ` (note trailing space) — title property
- `date:Date:start` — date property
- `Project` — relation to Projects database

**Projects** (collection://1be43898-8065-81de-ada7-000b3fff7492):
- `Name` — title property
- `Category` — select
- `Status` — status
- `Date Range` — date

### Create Page in Selected Database

```javascript
Notion:notion-create-pages({
  parent: { data_source_id: "[USER-SELECTED-DATABASE-ID]" },
  pages: [{
    properties: {
      "[title-property-name]": "Document Title",
      "date:[date-property]:start": "2026-03-26",
      "date:[date-property]:is_datetime": 0
      // Add other properties based on schema
    },
    content: "# Document content..."
  }]
})
```


### Project Linking (if database has relation property)

If the selected database has a Project relation, search for the project:

```javascript
Notion:notion-search({
  query: "[Project Name]",
  query_type: "internal",
  filters: {}
})
```

Then include the page URL in the relation property.

## Step 6: Confirm Both Outputs

> ✅ **Documentation saved to both locations:**
>
> 📁 **Obsidian**: `[folder]/[project]/[filename].md`
> 📓 **Notion**: [Title] in **[Database Name]** — [link]
>
> Linked to project: [[Project Name]]

## Error Handling

### Desktop Commander Not Available
> ⚠️ Can't write directly to Obsidian. Creating a downloadable .md file instead.

### Notion MCP Not Available
> ⚠️ Notion isn't connected. Saved to Obsidian only.

### No Databases Found in Notion
> ⚠️ Couldn't find any databases. Please provide a Notion database URL or create one first.

### Database Schema Mismatch
If the selected database doesn't have expected properties:
> ⚠️ This database doesn't have a date property. Creating page with title only.


## Quick Reference

| User Says | Action |
|-----------|--------|
| "doc it" | Scan both → Ask all questions → Save both |
| "doc it to Project Notes" | Pre-select Notion DB, ask rest |
| "doc it for Claude in Work" | Pre-select Obsidian folder |

## Example Flow

**User**: "doc it"

**Claude**:
1. Scans Obsidian → finds: `Test` folder in Personal Projects
2. Scans Notion → finds: `Project Notes`, `✅ To-Do Database`, `Projects`
3. Asks via `ask_user_input_v0`:
   - Title: "Doc-It Skill Setup" / Custom
   - Notion database: Project Notes / ✅ To-Do Database / Projects
   - Work or Personal: Work / Personal
   - Obsidian folder: Test / ➕ Create new folder

**User selects**: 
- "Doc-It Skill Setup"
- "Project Notes"
- "Personal Projects"
- "Test"

**Claude**:
1. Writes to Obsidian: `.../Personal Projects/Test/doc-it-skill-setup.md`
2. Fetches Project Notes schema
3. Creates Notion page in Project Notes database
4. Confirms:

> ✅ **Done!**
>
> 📁 **Obsidian**: `Personal Projects/Test/doc-it-skill-setup.md`
> 📓 **Notion**: Doc-It Skill Setup in **Project Notes** — [link]
