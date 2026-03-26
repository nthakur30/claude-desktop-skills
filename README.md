# Claude Desktop Skills

A collection of custom skills for Claude Desktop. Each skill is packaged as a `.skill` file — download and install individually.

## Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [Resume Evaluator](skills/resume-evaluator/) | Score resumes, check ATS, rewrite bullets, build one-page version | [resume-evaluator.skill](releases/resume-evaluator.skill) |

## How to Install a Skill

1. Click the skill link in the Install column above
2. Click **Download raw file** (the download icon on the right)
3. Open **Claude Desktop** → Settings → Skills
4. Click **Add Skill** and select the downloaded `.skill` file
5. Done — ask Claude to use the skill

## How to Add a New Skill (for contributors)

1. Create a folder under `skills/your-skill-name/`
2. Add a `SKILL.md` file inside it (see existing skills for format)
3. Build the `.skill` file:
   ```bash
   cd skills/your-skill-name
   zip -j ../../releases/your-skill-name.skill SKILL.md
   ```
4. Add a row to the table above
5. Commit and push

## Structure

```
claude-desktop-skills/
├── skills/
│   └── resume-evaluator/
│       └── SKILL.md          ← source of truth for each skill
└── releases/
    └── resume-evaluator.skill ← installable zip (SKILL.md at root inside)
```

## License

MIT
