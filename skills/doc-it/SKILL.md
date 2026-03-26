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

## Step 1: Ask User for Details

Always ask these questions using the `ask_user_input_v0` tool:

```json
{
  "questions": [
    {
      "question": "What should this doc be titled?",
      "type": "single_select",
      "options": ["[Suggested title based on conversation]", "Let me type a custom title"]
    },
    {
      "question": "Which project is this for?",
      "type": "single_select", 
      "options": ["Claude", "Home Server", "RASPI5-n8n", "Home Assistant", "BOT—>Jarvis", "PulsePathAI", "Capstone Paper", "Project 01-Trading Algo", "AI Business", "None"]
    },
    {
      "question": "Work or Personal project?",
      "type": "single_select",
      "options": ["Work Projects", "Personal Projects"]
    }
  ]
}
```

If user selects "Let me type a custom title", ask them to provide the title in the next message.


## Step 2: Extract & Structure Content

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


## Step 3: Save to Obsidian Vault

Use Desktop Commander to write directly to the Obsidian vault.

**Vault base path**: `/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes`

### Folder Structure

```
Project Notes/
├── Work Projects/
│   ├── Claude/
│   │   ├── mcp-integration-guide.md
│   │   └── doc-it-skill-setup.md
│   ├── RASPI5-n8n/
│   │   └── workflow-automation.md
│   └── Project 01-Trading Algo/
│       └── api-documentation.md
├── Personal Projects/
│   ├── Home Server/
│   │   └── docker-setup.md
│   ├── Home Assistant/
│   │   └── automation-config.md
│   └── BOT—>Jarvis/
│       └── telegram-bot-docs.md
```

### Path Construction

Based on user selections, build the path:

```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Project Name]/[filename].md
```

**Examples:**
- Work + Claude + "MCP Setup" → `.../Work Projects/Claude/mcp-setup.md`
- Personal + Home Server + "Docker Config" → `.../Personal Projects/Home Server/docker-config.md`

### Create Folder if Missing

Before writing, ensure the project folder exists:

```javascript
// Use desktop-commander:start_process
mkdir -p "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[project]"
```

### Filename Rules
- Lowercase
- Replace spaces with hyphens
- Remove special characters (except hyphens)
- Keep under 50 chars
- Example: "Home Assistant Setup Guide" → `home-assistant-setup-guide.md`


### Write using Desktop Commander

```javascript
// Use desktop-commander:write_file
{
  "path": "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[folder]/[project]/[filename].md",
  "content": "---\ntitle: ...\nproject: \"[[Project Name]]\"\n---\n\n# Content here...",
  "mode": "rewrite"
}
```

## Step 4: Create Notion Page

Use the Notion MCP to create a page in Project Notes database.

**Database ID**: `collection://28543898-8065-8094-a590-000ba4f44803`

**Required properties**:
- `Title ` (note the trailing space) — the document title
- `date:Date:start` — ISO date string (YYYY-MM-DD)
- `date:Date:is_datetime` — set to 0
- `Project` — JSON array of page URLs if linking to a project

### Project URL Mapping (for Notion relations)

When user selects a project, search for it first using `notion-search`, then use the page URL in the relation.

**Known project pages** (search to get current URLs):
- Claude
- Home Server
- RASPI5-n8n
- Home Assistant
- BOT—>Jarvis
- PulsePathAI
- Capstone Paper
- Project 01-Trading Algo
- AI Business


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

## Step 5: Confirm Both Outputs

After creating both, confirm:

> ✅ **Documentation saved to both locations:**
>
> 📁 **Obsidian**: `[folder]/[project]/[filename].md`
> 📓 **Notion**: [Title] — [notion page link]
>
> Linked to project: [[Project Name]]
>
> The file is now syncing via iCloud and will appear in your Obsidian graph.

## Special Cases

### Diagrams & Images

If the conversation includes:
- **Mermaid diagrams** — embed the code in both outputs
- **Architecture diagrams** — include ASCII or Mermaid representation
- **Screenshots referenced** — note their location/context

### Code-Heavy Documentation

For technical docs with lots of code:
- Preserve all code blocks with syntax highlighting
- Add brief explanations before each block
- Include any error messages and solutions
- Link to related docs with `[[wiki-links]]`


### Multi-Topic Conversations

If conversation covered multiple topics:
- Ask user which parts to document
- Or offer to create multiple docs
- Link them together with `[[wiki-links]]`

> This conversation covered several topics:
> 1. n8n workflow setup
> 2. Notion database structure  
> 3. Obsidian sync options
>
> Document all as one doc, or separate docs for each?

## Error Handling

### Desktop Commander Not Available
If Desktop Commander isn't connected:
> ⚠️ Can't write directly to Obsidian. Creating a downloadable .md file instead — drag it to your vault manually.

Fall back to creating file in `/mnt/user-data/outputs/` and present for download.

### Notion MCP Not Available
If Notion tools aren't connected:
> ⚠️ Notion isn't connected right now. I've saved the .md file to Obsidian — you can manually add it to Notion later.

### Project Not Found
If specified project doesn't exist in Notion:
> I couldn't find a project called "[name]". Creating the doc without a project link. You can add the relation manually in Notion.

### Folder Creation Failed
If can't create project subfolder:
> ⚠️ Couldn't create the project folder. Saving to the parent folder instead.


## Quick Reference

| User Says | Action |
|-----------|--------|
| "doc it" | Ask for title, project, folder → save both |
| "doc it for Claude" | Pre-select Claude project, ask for folder |
| "doc it in Work Projects" | Pre-select folder, ask for title/project |
| "doc this to Personal Projects for Home Server" | All info provided, just confirm and save |

## Example Flow

**User**: "doc it"

**Claude** (using ask_user_input_v0):
1. What should this doc be titled? → [suggested title] / Custom
2. Which project? → Claude / Home Server / ... / None  
3. Work or Personal? → Work Projects / Personal Projects

**User selects**: "Claude Workflow Automation" / "Claude" / "Work Projects"

**Claude**:
1. Creates folder: `.../Work Projects/Claude/` (if doesn't exist)
2. Writes to `.../Work Projects/Claude/claude-workflow-automation.md`
3. Content includes `[[Claude]]` wiki-links for graph
4. Creates Notion page with relation to Claude project
5. Confirms:

> ✅ **Done!**
>
> 📁 **Obsidian**: `Work Projects/Claude/claude-workflow-automation.md`
> 📓 **Notion**: Claude Workflow Automation — [link]
>
> Linked to: [[Claude]]
>
> Your Obsidian graph will now show this doc connected to the Claude project.

## Shortcut Patterns

If user provides info upfront, skip those questions:

- "doc it for Home Server" → Skip project question
- "doc it as 'API Setup Guide'" → Skip title question  
- "doc it to Personal Projects" → Skip folder question
- "doc it for Claude in Work Projects as 'MCP Integration'" → Skip all questions, just confirm
