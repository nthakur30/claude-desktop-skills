---
name: doc-it
description: Dual-output documentation workflow. Triggers on "doc it", "document this", "save to notion", "create documentation", or any request to save conversation content as documentation. Generates a downloadable .md file AND creates a Notion page simultaneously. Always use this skill when the user wants to preserve technical docs, project notes, meeting notes, or any knowledge from a conversation.
---

# Doc-It: Dual Documentation Output

When triggered, this skill extracts documentation from the conversation and outputs it in two formats:
1. **Markdown file** — downloadable for Obsidian/Git
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

## Step 1: Gather Minimal Info

Ask ONLY what's missing. If context is clear, skip to Step 2.

> Quick — I'll create both a .md file and a Notion page:
>
> 1. **Title**: [suggest based on conversation topic]
> 2. **Project**: Which project? (or "none")

Present project options from their database if known:
- Project 01-Trading Algo
- AI Business
- Home Assistant
- PulsePathAI
- Capstone Paper
- Claude
- Home Server
- BOT—>Jarvis
- RASPI5-n8n

If user says a project name or abbreviation, match it. If "none", skip the relation.


## Step 2: Extract & Structure Content

Pull relevant content from the conversation. Structure as:

```markdown
---
title: [Document Title]
date: [YYYY-MM-DD]
project: [Project Name or empty]
tags: [relevant, tags, here]
---

# [Document Title]

## Overview
[2-3 sentence summary of what this document covers]

## [Main Sections]
[Extracted and organized content from conversation]

## Key Points
- [Bullet summary of important takeaways]

## Next Steps
- [Action items if any]

---
*Documentation generated [Date] via Claude*
```

### Content Extraction Rules

1. **Code blocks** — preserve with language tags
2. **Diagrams** — include Mermaid syntax if generated
3. **Commands** — wrap in code blocks
4. **Architecture decisions** — capture reasoning
5. **Troubleshooting steps** — preserve in order
6. **Links/resources** — include all mentioned URLs


## Step 3: Generate Markdown File

Create the .md file and present for download:

```
I've created your documentation:

📄 **[title].md** — [download link]

Drag this to your Obsidian vault.
```

Use the `create_file` tool to write to `/mnt/user-data/outputs/[slugified-title].md`

### Filename Rules
- Lowercase
- Replace spaces with hyphens
- Remove special characters
- Keep under 50 chars
- Example: "Home Assistant Setup Guide" → `home-assistant-setup-guide.md`

## Step 4: Create Notion Page

Use the Notion MCP to create a page in Project Notes database.

**Database ID**: `collection://28543898-8065-8094-a590-000ba4f44803`

**Required properties**:
- `Title ` (note the trailing space) — the document title
- `date:Date:start` — ISO date string (YYYY-MM-DD)
- `date:Date:is_datetime` — set to 0
- `Project` — JSON array of page URLs if linking to a project


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

### Project Linking

If user specifies a project, search for it first:

```javascript
// Use notion-search to find project page
{
  "query": "Project Name",
  "query_type": "internal",
  "filters": {}
}
```

Then include the page URL in the Project relation array.

## Step 5: Confirm Both Outputs

After creating both, confirm:

> ✅ **Documentation saved to both locations:**
>
> 📄 **Markdown**: [filename].md — [download link]
> 📓 **Notion**: [Title] — [notion page link]
>
> The .md file is ready to drag into your Obsidian vault.


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

### Multi-Topic Conversations

If conversation covered multiple topics:
- Ask user which parts to document
- Or offer to create multiple docs

> This conversation covered several topics:
> 1. n8n workflow setup
> 2. Notion database structure
> 3. Obsidian sync options
>
> Document all as one doc, or separate docs for each?

## Error Handling

### Notion MCP Not Available
If Notion tools aren't connected:
> ⚠️ Notion isn't connected right now. I've created the .md file — you can manually add it to Notion later, or reconnect Notion and say "doc it" again.

### Project Not Found
If specified project doesn't exist:
> I couldn't find a project called "[name]". Creating the doc without a project link. You can add the relation manually in Notion.


## Quick Reference

| User Says | Action |
|-----------|--------|
| "doc it" | Extract from full conversation |
| "doc this section" | Extract from recent context only |
| "doc it for [project]" | Link to specified project |
| "doc it as [title]" | Use specified title |

## Example Flow

**User**: "doc it for the Home Server project"

**Claude**:
1. Extracts conversation content
2. Generates `home-server-[topic].md`
3. Creates Notion page with Project relation to "Home Server"
4. Presents both links

> ✅ **Done!**
>
> 📄 `home-server-docker-setup.md` — [download]
> 📓 **Docker Setup Guide** — [Notion link]
>
> Linked to: Home Server project
