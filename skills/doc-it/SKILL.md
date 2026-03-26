---
name: doc-it
description: Dual-output documentation workflow. Triggers on "doc it", "document this", "save to notion", "create documentation", or any request to save conversation content as documentation. Generates a downloadable .md file AND creates a Notion page simultaneously. Always use this skill when the user wants to preserve technical docs, project notes, meeting notes, or any knowledge from a conversation.
---

# Doc-It: Dual Documentation Output

When triggered, this skill extracts documentation from the conversation and outputs it in two formats:
1. **Markdown file** — saved directly to Obsidian vault with `[[wiki-links]]` for graph connections
2. **Notion page** — created in the user's Project Notes database

## Trigger Phrases

Activate when user says any of:
- "doc it"
- "document this"
- "save to notion"
- "create documentation"
- "save this as a doc"
- "make a doc from this"
- "add this to my notes"

## Step 1: Scan Obsidian Vault for ACTUAL Folders

**CRITICAL: Only show folders that ACTUALLY EXIST in the vault scan. Do NOT hardcode or assume folder names.**

**Vault base path**: `/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes`

### Scan Process

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

### Parse Results STRICTLY

From the scan results:
- **ONLY** extract lines starting with `[DIR]`
- **IGNORE** lines starting with `[FILE]` — these are loose files, not folders
- **IGNORE** `.obsidian`, `.DS_Store`, and hidden folders (starting with `.`)
- Build a list of **ACTUAL folder names found**

**Example scan result:**
```
[FILE] .DS_Store
[DIR] Test
[FILE] some-doc.md
```

From this, ONLY "Test" is a valid folder option.


## Step 2: Ask User for Details (with REAL folders only)

Use `ask_user_input_v0` with **ONLY the actual folders found** in Step 1.

### Question Flow

**Question 1: Title**
```json
{
  "question": "What should this doc be titled?",
  "type": "single_select",
  "options": ["[Suggested title]", "Custom title"]
}
```

**Question 2: Work or Personal**
```json
{
  "question": "Work or Personal project?",
  "type": "single_select",
  "options": ["Work Projects", "Personal Projects"]
}
```

**Question 3: Project Folder (DYNAMIC - from scan)**

Build options from ACTUAL scan results:

```json
{
  "question": "Which project folder?",
  "type": "single_select",
  "options": ["[ONLY FOLDERS FOUND IN SCAN]", "➕ Create new folder"]
}
```

**Example — if scan found these folders:**
- Work Projects: `Claude`, `RASPI5-n8n`
- Personal Projects: `Test`, `Home-Server`

And user selected "Work Projects", show:
```json
{
  "options": ["Claude", "RASPI5-n8n", "➕ Create new folder"]
}
```

**If NO folders found in selected category:**
```json
{
  "options": ["➕ Create new folder"]
}
```

### Handling "➕ Create new folder"

If user selects this, ask:
> What should the new folder be called?

Then create it before saving the file.

### Handling "Custom title"

If user selects this, ask:
> What title do you want for this doc?


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
  - "[[Related Doc 2]]"
---

# [Document Title]

> **Project**: [[Project Folder Name]]
> **Created**: [Date]

## Overview
[2-3 sentence summary of what this document covers]

## [Main Sections]
[Extracted and organized content from conversation]

## Key Points
- [Bullet summary of important takeaways]

## Related
- [[Project Folder Name]] — Parent project
- [[Other Related Doc]] — If referenced in conversation

## Next Steps
- [ ] [Action items as tasks]

---
*Documentation generated [Date] via Claude*
```

### Wiki-Link Rules for Obsidian Graph

**ALWAYS use `[[double brackets]]` for:**
- Project folder names: `[[Claude]]`, `[[Home-Server]]`, `[[RASPI5-n8n]]`
- Related documents: `[[API Setup Guide]]`, `[[Docker Configuration]]`
- Concepts that might have their own notes: `[[n8n]]`, `[[Notion API]]`, `[[MCP]]`
- Cross-references: `See [[Related Topic]]`

**Format patterns:**
- Link to project: `[[Project Name]]`
- Link with alias: `[[actual-file-name|Display Text]]`
- Link to heading: `[[Document#Heading]]`


### Content Extraction Rules

1. **Code blocks** — preserve with language tags
2. **Diagrams** — include Mermaid syntax if generated
3. **Commands** — wrap in code blocks
4. **Architecture decisions** — capture reasoning
5. **Troubleshooting steps** — preserve in order
6. **Links/resources** — include all mentioned URLs
7. **Related concepts** — wrap in `[[wiki-links]]` for graph connections

## Step 4: Save to Obsidian Vault

### Path Construction

```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Project Folder]/[filename].md
```

### Create Folder if New

If user selected "➕ Create new folder", create it first:

```javascript
desktop-commander:start_process({
  command: "mkdir -p '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[new-project-name]'",
  timeout_ms: 5000
})
```

### Filename Rules
- Lowercase
- Replace spaces with hyphens
- Remove special characters (except hyphens)
- Keep under 50 chars
- Example: "Home Assistant Setup Guide" → `home-assistant-setup-guide.md`

### Write File

```javascript
desktop-commander:write_file({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[project]/[filename].md",
  content: "---\ntitle: ...\nproject: \"[[Project Name]]\"\n---\n\n# Content...",
  mode: "rewrite"
})
```


## Step 5: Create Notion Page

Use the Notion MCP to create a page in Project Notes database.

**Database ID**: `collection://28543898-8065-8094-a590-000ba4f44803`

**Required properties**:
- `Title ` (note the trailing space) — the document title
- `date:Date:start` — ISO date string (YYYY-MM-DD)
- `date:Date:is_datetime` — set to 0
- `Project` — JSON array of page URLs if linking to a project

### Project Linking for Notion

Search for the project in Notion (may have different name than Obsidian folder):

```javascript
Notion:notion-search({
  query: "Project Name",
  query_type: "internal",
  filters: {}
})
```

Then include the page URL in the relation.

### Notion API Call

```javascript
Notion:notion-create-pages({
  parent: { data_source_id: "28543898-8065-8094-a590-000ba4f44803" },
  pages: [{
    properties: {
      "Title ": "Document Title Here",
      "date:Date:start": "2026-03-26",
      "date:Date:is_datetime": 0,
      "Project": "[\"https://notion.so/project-page-id\"]"
    },
    content: "# Document content..."
  }]
})
```

## Step 6: Confirm Both Outputs

> ✅ **Documentation saved to both locations:**
>
> 📁 **Obsidian**: `[folder]/[project]/[filename].md`
> 📓 **Notion**: [Title] — [notion page link]
>
> Linked to project: [[Project Name]]


## Error Handling

### Desktop Commander Not Available
> ⚠️ Can't write directly to Obsidian. Creating a downloadable .md file instead.

Fall back to `/mnt/user-data/outputs/` and present for download.

### Notion MCP Not Available
> ⚠️ Notion isn't connected. Saved to Obsidian only.

### No Folders Found in Category
Show only "➕ Create new folder" as the option.

### Folder Scan Failed
> ⚠️ Couldn't scan your vault. Please tell me the folder name.

## Quick Reference

| User Says | Action |
|-----------|--------|
| "doc it" | Scan vault → Ask title, category, folder → Save both |
| "doc it for [project]" | Match to existing folder if found |
| "doc it in Work Projects" | Pre-select category, scan for folders |

## Example Flow

**User**: "doc it"

**Claude**:
1. Scans Work Projects → finds: (no folders)
2. Scans Personal Projects → finds: `Test`
3. Asks:
   - Title: "Doc-It Skill Setup" / Custom
   - Work or Personal: Work / Personal
   - Folder: `Test` / ➕ Create new folder (if Personal selected)
   
**User selects**: "Doc-It Skill Setup" / "Work Projects" / "➕ Create new folder"

**Claude**: "What should the new folder be called?"

**User**: "Claude"

**Claude**:
1. Creates folder: `.../Work Projects/Claude/`
2. Writes `.../Work Projects/Claude/doc-it-skill-setup.md`
3. Creates Notion page
4. Confirms both locations
