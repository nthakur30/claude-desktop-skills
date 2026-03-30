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

**Store the database ID** for the selected Notion option to use when creating the page.

If user selects "Custom title" → ask: *What title do you want for this doc?*
If user selects "➕ Create new folder" → ask: *What should the new folder be called?*


## Step 3: Extract & Structure Content

**This step is MANDATORY and must produce a detailed, thorough document. Do NOT write a short summary. The goal is a complete reference document someone could read months later and fully understand the project.**

### Content Extraction Checklist

Before writing, scan the full conversation and collect answers to ALL of these:

**Project Identity**
- What is this project/tool/system called?
- What problem does it solve? What is its core purpose?
- Who is it for? How does it fit into the user's broader workflow?

**Architecture & Structure**
- What are the major components, modules, or parts?
- How do they connect or interact?
- What is the data flow or process flow?
- What technologies, tools, languages, or platforms are used?

**Implementation Details**
- What specific configuration, settings, or parameters were discussed?
- What code, scripts, or commands were written or referenced?
- What file paths, endpoints, or identifiers were mentioned?

**Decisions & Reasoning**
- What design decisions or tradeoffs were made?
- What alternatives were considered and rejected?
- Why was this approach chosen?

**Problems & Solutions**
- What errors, bugs, or blockers came up?
- How were they resolved?
- What edge cases or gotchas were identified?

**Status & Next Steps**
- What has been completed?
- What is still in progress or not yet started?
- What are the explicit next actions?


### Document Template

Use ALL sections that apply. **Never skip a section because it's hard to fill — if it was discussed, it belongs in the doc.**

```markdown
---
title: [Document Title]
date: [YYYY-MM-DD]
project: "[[Project Folder Name]]"
tags: [relevant, tags, here]
related:
  - "[[Related Doc 1]]"
  - "[[Related Tool or Concept]]"
---

# [Document Title]

> **Project**: [[Project Folder Name]]
> **Created**: [Date]

## Overview
[3-5 sentences. What is this? What problem does it solve? Why does it matter?
Be specific — mention the tools it integrates with using [[wiki-links]].]

## Purpose & Goals
- **Primary goal**: [What this is designed to do]
- **Secondary goals**: [Any supporting objectives]
- **Not in scope**: [What this deliberately does NOT do, if mentioned]

## Architecture & Components

### [Component 1 Name]
[What it does, how it works, why it exists — link tools with [[Tool Name]]]

### [Component 2 Name]
[What it does, how it works, why it exists]

### How They Connect
[Describe the flow: what triggers what, what data passes between components]

## Technical Details

### Technologies Used
- **[[Tool/Language/Platform]]**: [Why it's used / what role it plays]

### Configuration & Settings
[Any specific config values, API keys structure, environment variables, file paths, etc.]

### Key Code / Commands
[Any important scripts, functions, or commands that were written or referenced]

## Decisions & Tradeoffs
| Decision | Why | Alternatives Considered |
|----------|-----|------------------------|
| [Choice made] | [Reasoning] | [What else was considered] |

## Problems & Solutions
### [Problem Title]
- **Issue**: [What went wrong]
- **Root cause**: [Why it happened]
- **Solution**: [How it was fixed]

## Current Status
- ✅ [What's done]
- 🔄 [What's in progress]
- ⏳ [What hasn't started yet]

## Next Steps
- [ ] [Specific action item with enough context to act on later]

## Related
- [[Project Folder Name]] — Parent project
- [[Related Tool]] — [Why it connects]
- [[Concept Name]] — [Why it's relevant]

---
*Documentation generated [Date] via Claude*
```


### Wiki-Link Rules for Obsidian Graph

**CRITICAL: Wiki-links are what make Obsidian useful. Every doc must have meaningful `[[links]]` so the graph shows real connections. Sparse or missing links is a failure.**

#### What ALWAYS gets a wiki-link

| Content Type | Example |
|---|---|
| Project folder / parent project | `[[Home-Server]]`, `[[Claude]]`, `[[n8n]]` |
| Related docs or guides | `[[API Setup Guide]]`, `[[Tailscale Setup]]` |
| Tools, platforms, services | `[[Notion]]`, `[[Docker]]`, `[[GitHub]]` |
| Key concepts or patterns | `[[MCP]]`, `[[Webhook]]`, `[[RAG]]` |
| Technologies used | `[[Python]]`, `[[React]]`, `[[PostgreSQL]]` |
| Other projects this touches | `[[Home-Server]]` if referencing infra |

#### Linking rules

1. **First mention only** — Link a term the first time it appears in each section, not every time
2. **Be precise** — `[[n8n Workflow Setup]]` is better than `[[Setup]]`
3. **Link in frontmatter** — `project: "[[Project Folder Name]]"` and `related: ["[[Doc Name]]"]`
4. **Required: Related section** — every doc must have at least 2-3 `[[links]]` in Related
5. **Don't link generic words** — never link `[[file]]`, `[[code]]`, `[[user]]`

#### Required links in every document

```markdown
---
project: "[[Project Folder Name]]"     ← REQUIRED: links to parent project
related:
  - "[[At least one related doc]]"     ← REQUIRED: minimum one cross-link
---

## Overview
This project extends [[Parent Project]] and integrates with [[Related Tool]]...

## Related
- [[Project Folder Name]] — Parent project
- [[Related Tool]] — [why it connects]
- [[Concept Name]] — [why it's relevant]
```

#### How to find what to link

Scan the conversation for:
- Any tool or technology mentioned → `[[tool name]]`
- Any other project referenced → `[[project name]]`
- Any concept that might have its own Obsidian note → `[[concept]]`
- The Obsidian folder the doc lives in → always link the parent folder

#### Good vs bad linking examples

❌ **Too sparse** (do not do this):
```markdown
## Overview
This is a skill that saves documentation to Notion and Obsidian.

## Related
- Parent project
```

✅ **Correct linking**:
```markdown
## Overview
Doc-It is a dual-output documentation skill for [[Claude Desktop]] that saves structured
notes to [[Obsidian]] and [[Notion]] simultaneously. It was built as part of the [[Claude]]
project to eliminate manual copy-paste from Claude sessions into the knowledge base.

## Related
- [[Claude]] — Parent project
- [[Obsidian]] — Primary vault target
- [[Notion]] — Secondary documentation target
- [[MCP]] — Protocol used for Notion API integration
- [[Desktop Commander]] — Used to write files to the vault
```

### Content Depth Standard

❌ **Too shallow** (do not do this):
> ## Overview
> This is a skill that saves documentation.

✅ **Correct depth**:
> ## Overview
> Doc-It is a dual-output documentation workflow that simultaneously saves a structured
> Markdown file to the user's [[Obsidian]] vault and creates a linked [[Notion]] page.
> It was built to eliminate the manual copy-paste step between [[Claude]] conversations
> and the user's personal knowledge base, ensuring no project context is lost after a session.


## Step 4: Save to Obsidian Vault

**THIS STEP IS NOT OPTIONAL. You must write the .md file to Obsidian every time. Do not skip, summarize, or defer this step. If Desktop Commander fails, fall back to a downloadable file — but always produce the file.**

### Path Construction

Build the full path using the user's answers from Step 2:

```
BASE:      /Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/
SEGMENT 1: [Work Projects] or [Personal Projects]   ← from user's answer
SEGMENT 2: [Project Folder Name]                    ← from user's answer
SEGMENT 3: [filename].md                            ← derived from title

FULL PATH = BASE + SEGMENT 1 + "/" + SEGMENT 2 + "/" + SEGMENT 3
```

**Before writing, verify:**
- SEGMENT 1 is exactly `Work Projects` or `Personal Projects` — nothing else
- SEGMENT 2 matches a folder from the Obsidian scan (or the new folder name confirmed by user)
- SEGMENT 3 follows filename rules below

### Filename Rules
- Lowercase, hyphens instead of spaces
- Remove special characters (`?`, `!`, `:`, `'`, `"`, `/`)
- Keep under 50 chars
- Example: `Doc-It Skill Setup` → `doc-it-skill-setup.md`

### Concrete Path Example

User selected **Personal Projects** → folder **Test** → title **Doc-It Skill Setup**:
```
/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/Personal Projects/Test/doc-it-skill-setup.md
```

### Create Folder if New
```javascript
desktop-commander:start_process({
  command: "mkdir -p '/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Project Folder]'",
  timeout_ms: 5000
})
```

### Write File — REQUIRED

**You must call this. Do not skip it.**

```javascript
desktop-commander:write_file({
  path: "/Users/nirvahnthakur/Library/Mobile Documents/iCloud~md~obsidian/Documents/Project Notes/[Work or Personal Projects]/[Project Folder]/[filename].md",
  content: "---\ntitle: ...\n---\n\n# Full document content here...",
  mode: "rewrite"
})
```

**After writing, verify it succeeded.** If `desktop-commander` returns an error, immediately fall back:
```
⚠️ Couldn't write to Obsidian. Creating a downloadable .md file instead.
```
Then use `present_files` to deliver the file to the user.


## Step 5: Create Notion Page (in USER-SELECTED database)

**CRITICAL: Use the database ID from the user's selection in Step 2.**

### Fetch Database Schema First

```javascript
Notion:notion-fetch({
  id: "[selected-database-id]"
})
```

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
      "date:[date-property]:start": "2026-03-30",
      "date:[date-property]:is_datetime": 0
    },
    content: "# Document content..."
  }]
})
```

### Project Linking (if database has relation property)

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
3. Asks via `ask_user_input_v0` for title, Notion DB, Work/Personal, folder
4. User selects: "Doc-It Skill Setup" / "Project Notes" / "Personal Projects" / "Test"
5. Writes `.../Personal Projects/Test/doc-it-skill-setup.md` with full content and `[[wiki-links]]`
6. Fetches Project Notes schema → creates Notion page
7. Confirms:

> ✅ **Done!**
>
> 📁 **Obsidian**: `Personal Projects/Test/doc-it-skill-setup.md`
> 📓 **Notion**: Doc-It Skill Setup in **Project Notes** — [link]
