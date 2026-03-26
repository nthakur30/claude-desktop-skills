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

## Step 1: Scan Obsidian Vault for Folders

**BEFORE asking questions**, scan the Obsidian vault to get current project folders.

**Vault base path**: `/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes`

Use Desktop Commander to scan:

```javascript
// First, scan Work Projects
desktop-commander:list_directory({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Work Projects",
  depth: 1
})

// Then, scan Personal Projects
desktop-commander:list_directory({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Personal Projects", 
  depth: 1
})
```

**Extract folder names** from the results:
- Look for lines starting with `[DIR]`
- Ignore `.obsidian`, `.DS_Store`, and hidden folders
- Build a list of project folder names for each category


## Step 2: Ask User for Details (with dynamic options)

Use `ask_user_input_v0` with the **actual folders found** in Step 1.

**Build the questions dynamically:**

```json
{
  "questions": [
    {
      "question": "What should this doc be titled?",
      "type": "single_select",
      "options": ["[Suggested title based on conversation]", "Let me type a custom title"]
    },
    {
      "question": "Work or Personal project?",
      "type": "single_select",
      "options": ["Work Projects", "Personal Projects"]
    },
    {
      "question": "Which project folder? (from Work Projects)",
      "type": "single_select",
      "options": ["[DYNAMICALLY POPULATED FROM SCAN]", "➕ Create new folder"]
    }
  ]
}
```

**Dynamic options example:**

If vault scan found:
- Work Projects: `Claude`, `RASPI5-n8n`, `Trading-Algo`
- Personal Projects: `Home-Server`, `Home-Assistant`, `Test`

Then show:
```json
{
  "question": "Which project folder?",
  "options": ["Claude", "RASPI5-n8n", "Trading-Algo", "➕ Create new folder"]
}
```

**If user selects "➕ Create new folder":**
Ask them to type the new folder name in the next message.

**If user selects "Let me type a custom title":**
Ask them to type the title in the next message.


## Step 3: Extract & Structure Content

Pull relevant content from the conversation. Structure as:

```markdown
---
title: [Document Title]
date: [YYYY-MM-DD]
project: "[[Project Name]]"
tags: [relevant, tags, here]
related:
  - "[[Related Doc 1]]"
  - "[[Related Doc 2]]"
---

# [Document Title]

> **Project**: [[Project Name]]
> **Created**: [Date]

## Overview
[2-3 sentence summary of what this document covers]

## [Main Sections]
[Extracted and organized content from conversation]

## Key Points
- [Bullet summary of important takeaways]

## Related
- [[Project Name]] — Parent project
- [[Other Related Doc]] — If referenced in conversation

## Next Steps
- [ ] [Action items as tasks]

---
*Documentation generated [Date] via Claude*
```

### Wiki-Link Rules for Obsidian Graph

**ALWAYS use `[[double brackets]]` for:**
- Project names: `[[Claude]]`, `[[Home Server]]`, `[[RASPI5-n8n]]`
- Related documents: `[[API Setup Guide]]`, `[[Docker Configuration]]`
- Concepts that might have their own notes: `[[n8n]]`, `[[Notion API]]`, `[[MCP]]`
- Cross-references within the doc: `See [[Related Topic]]`

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

Based on user selections, build the path:

```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Project Folder]/[filename].md
```

### Create Folder if Missing (for new projects)

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


### Write using Desktop Commander

```javascript
desktop-commander:write_file({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[project]/[filename].md",
  content: "---\ntitle: ...\nproject: \"[[Project Name]]\"\n---\n\n# Content here...",
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

### Project URL Mapping (for Notion relations)

When user selects a project, search for it first using `notion-search`, then use the page URL in the relation.

### Notion API Call Structure

```javascript
// Use notion-create-pages tool
{
  "parent": {
    "data_source_id": "28543898-8065-8094-a590-000ba4f44803"
  },
  "pages": [{
    "properties": {
      "Title ": "Document Title Here",
      "date:Date:start": "2026-03-26",
      "date:Date:is_datetime": 0,
      "Project": "[\"https://notion.so/project-page-id\"]"
    },
    "content": "# Document content in Notion markdown..."
  }]
}
```


## Step 6: Confirm Both Outputs

After creating both, confirm:

> ✅ **Documentation saved to both locations:**
>
> 📁 **Obsidian**: `[folder]/[project]/[filename].md`
> 📓 **Notion**: [Title] — [notion page link]
>
> Linked to project: [[Project Name]]
>
> The file is now syncing via iCloud and will appear in your Obsidian graph.

## Error Handling

### Desktop Commander Not Available
If Desktop Commander isn't connected:
> ⚠️ Can't write directly to Obsidian. Creating a downloadable .md file instead — drag it to your vault manually.

Fall back to creating file in `/mnt/user-data/outputs/` and present for download.

### Notion MCP Not Available
If Notion tools aren't connected:
> ⚠️ Notion isn't connected right now. I've saved the .md file to Obsidian — you can manually add it to Notion later.

### Folder Scan Failed
If can't scan Obsidian vault:
> ⚠️ Couldn't scan your vault. Please tell me the project folder name.

### Project Not Found in Notion
If specified project doesn't exist in Notion search:
> I couldn't find a project called "[name]" in Notion. Creating the doc without a project link. You can add the relation manually.


## Quick Reference

| User Says | Action |
|-----------|--------|
| "doc it" | Scan vault → Ask title, folder, project → Save both |
| "doc it for Claude" | Scan vault → Pre-select Claude if exists → Ask folder |
| "doc it in Work Projects" | Scan Work Projects → Show those folders → Ask title |
| "doc this to Personal Projects/Home-Server" | All info provided → Confirm and save |

## Example Flow

**User**: "doc it"

**Claude**:
1. Scans `/Users/nirvahnthakur/.../Project Notes/Work Projects/` → finds: `Claude`, `RASPI5-n8n`
2. Scans `/Users/nirvahnthakur/.../Project Notes/Personal Projects/` → finds: `Home-Server`, `Test`
3. Asks via `ask_user_input_v0`:
   - Title: "Doc-It Skill Setup" / Custom
   - Work or Personal: Work Projects / Personal Projects
   - Folder: Claude / RASPI5-n8n / ➕ Create new folder

**User selects**: "Doc-It Skill Setup" / "Work Projects" / "Claude"

**Claude**:
1. Writes to `.../Work Projects/Claude/doc-it-skill-setup.md`
2. Content includes `[[Claude]]` wiki-links
3. Creates Notion page with relation to Claude project
4. Confirms:

> ✅ **Done!**
>
> 📁 **Obsidian**: `Work Projects/Claude/doc-it-skill-setup.md`
> 📓 **Notion**: Doc-It Skill Setup — [link]
>
> Linked to: [[Claude]]


## Shortcut Patterns

If user provides info upfront, skip those questions:

- "doc it for Claude" → Skip project question, still show folder scan results
- "doc it as 'API Setup Guide'" → Skip title question
- "doc it to Work Projects" → Skip work/personal question
- "doc it to Work Projects/Claude as 'MCP Integration'" → Skip all, just confirm

## New Folder Creation Flow

If user selects "➕ Create new folder":

1. Ask: "What should the new folder be called?"
2. User types: "PulsePathAI"
3. Create folder: `mkdir -p .../Work Projects/PulsePathAI`
4. Write file into that new folder
5. Confirm creation of both folder and file

## Special Cases

### Multi-Topic Conversations

If conversation covered multiple topics, link them together:

```markdown
## Related Docs
- [[Topic 1 Doc]] — First part of this conversation
- [[Topic 2 Doc]] — Second part
```

### Code-Heavy Documentation

For technical docs with lots of code:
- Preserve all code blocks with syntax highlighting
- Link to related docs with `[[wiki-links]]`
- Include troubleshooting sections
