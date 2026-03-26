---
name: resume-evaluating
description: Evaluates and improves resumes for job applications. Use when user asks to review, critique, score, improve, or optimize a resume or CV. Handles PDF, DOCX, and text resumes. Provides ATS checks, scoring, and bullet rewrites. Trigger for "look at my resume" or "help with my resume."
---

# Resume Evaluator & Improver

## Step 1: Gather Context

BEFORE evaluating, always ask the user:

> I'll evaluate and improve your resume. First, a few quick questions:
>
> 1. **Target role**: What job or industry is this resume for?
> 2. **Experience level**: Entry-level, mid-career, senior, or executive?
> 3. **Job posting**: Do you have a specific job description to match? (Paste it or say "no")
> 4. **Focus areas**: Any specific concerns? (ATS compatibility, weak bullets, length, formatting)
>
> Feel free to answer in shorthand: "1: SWE at startup, 2: mid, 3: no, 4: my bullets are boring"

Wait for the user's response before proceeding to evaluation.

## Step 2: Extract Resume

- **PDF**: Use pdftotext or pdf-reading skill
- **DOCX**: Use docx skill
- **Text**: Work directly with pasted content

## Step 3: Score Across 6 Dimensions

| Dimension | Weight | What to Check |
|-----------|--------|---------------|
| Impact & Achievements | 25% | Metrics, action verbs, clear "so what?" |
| ATS Compatibility | 20% | Standard headers, no tables/images, keywords |
| Relevance | 20% | Tailored to target role, key skills prominent |
| Structure | 15% | Appropriate length, visual hierarchy, consistency |
| Presentation | 10% | Grammar, professional email, LinkedIn URL |
| Summary | 10% | Compelling value prop, not generic |

**Scoring guide:**
- 9-10: Excellent, no changes needed
- 7-8: Good, minor improvements
- 5-6: Average, needs work
- 3-4: Weak, significant gaps
- 1-2: Poor, major rewrite needed

## Step 4: Present Report

Use this format:
Provide 3-5 before/after examples from their actual resume.

## Step 5: Offer Next Steps

After presenting the report, ask:

> What would you like to do next?
>
> 1. **Deep dive** — Detailed feedback on a specific section
> 2. **Rewrite bullets** — Transform weak bullets using the CAR+M formula
> 3. **Match to job posting** — Analyze gaps against a specific job description
> 4. **Build my one-page resume** — Generate a polished, complete one-page resume
> 5. **Done for now** — We're finished
>
> Pick a number or tell me what you need.

## Step 6: Build One-Page Resume (if user selects option 4)

### Rules — non-negotiable
- **Exactly one page** — ruthlessly cut to fit; never truncate meaning, always compress wording
- **Zero information loss** — every role, company, date, and achievement from the original must appear
- **Preserve all metrics** — any number, percentage, or dollar figure in the original must survive
- **Do not invent** — never add facts, metrics, or responsibilities that weren't in the original

### Compression Hierarchy (cut in this order until it fits)
1. Remove outdated skills list filler (MS Office, typing, etc.)
2. Remove "References available upon request" and objective statements
3. Tighten bullets to single lines — remove filler words, keep the metric
4. Merge redundant bullets that make the same point
5. Reduce education section to: Degree, School, Year (drop GPA if ≥ 3 years out)
6. Trim summary/profile to 2 sentences max
7. Reduce margins and font size only as a last resort (minimum: 0.5in margins, 10pt font)

### Output Format

Produce the resume as a clean markdown block with this structure:

```
# [Full Name]
[email] • [phone] • [LinkedIn] • [city, state only — no street]

## Summary
[2 sentences max — value prop tailored to target role]

## Experience

**[Job Title]** | [Company] | [Start] – [End]
- [Bullet using CAR+M: action + result + metric]
- [Bullet]
- [Bullet — max 3 per role for mid/senior; 2 for older roles]

[repeat for each role]

## Education
**[Degree]**, [School] — [Year]

## Skills
[Comma-separated, role-relevant only — max 12]
```

### Humanizer Pass — REQUIRED

After generating the resume draft, invoke the **humanizer skill** to rewrite the language:

> @humanizer — Please rewrite this resume to sound like a real person wrote it. Keep all facts, metrics, and dates exactly as written. Make bullets punchy and direct. Remove any AI-sounding phrases like "leveraged", "spearheaded", "fostered cross-functional synergies", or "results-driven professional." The writing should feel confident and natural — like someone who knows what they're good at.

Apply the humanizer output as the final version. If the humanizer skill is not available, manually replace any of these AI-clichés before delivering:

| Replace | With |
|---------|------|
| Leveraged | Used / Built with |
| Spearheaded | Led / Started |
| Synergized / Collaborated cross-functionally | Worked with [teams] |
| Results-driven professional | (just delete it) |
| Passionate about | (just delete it) |
| Utilized | Used |
| Interfaced with | Worked with |

### Final Delivery

After the humanizer pass, present the finished resume and say:

> Here's your one-page resume — all your experience, compressed and cleaned up.
>
> Want me to:
> - Adjust the tone for a specific company culture (startup vs. corporate)?
> - Swap in keywords from a job posting?
> - Export this as a formatted file?

## CAR+M Formula for Bullets

- **C**hallenge: What was the situation or problem?
- **A**ction: What specifically did you do?
- **R**esult: What was the outcome?
- **M**etric: Quantify the impact

### Transformation Examples

| Before | After |
|--------|-------|
| Handled customer complaints | Resolved 50+ weekly escalations with 94% satisfaction, reducing churn 15% |
| Worked on software projects | Built 3 microservices handling 10K daily transactions, improving response time 60% |
| Managed a team | Led 8-person team delivering 12 features on schedule, cutting deployment time 40% |
| Helped with marketing | Launched 12 email campaigns reaching 100K subscribers, generating $250K pipeline |

## Power Verbs by Industry

- **Tech**: Architected, Deployed, Optimized, Automated, Scaled, Migrated
- **Sales**: Closed, Negotiated, Exceeded, Captured, Expanded, Accelerated
- **Marketing**: Launched, Grew, Generated, Drove, Positioned, Amplified
- **Operations**: Streamlined, Reduced, Implemented, Standardized, Transformed
- **Finance**: Analyzed, Forecasted, Reconciled, Maximized, Optimized

## Common Mistakes to Flag

1. ❌ Outdated email (aol, hotmail) → Use Gmail or custom domain
2. ❌ Physical address → Remove (enables bias)
3. ❌ "References available upon request" → Delete (outdated)
4. ❌ Objective statement → Replace with professional summary
5. ❌ Personal pronouns (I, me, my) → Remove
6. ❌ Inconsistent date formats → Standardize
7. ❌ GPA below 3.5 → Remove unless recent grad
8. ❌ Generic skills (Microsoft Office, typing) → Remove

## Job Matching (if user provided job description)

Create a match table:

| Requirement | Your Resume | Status |
|-------------|-------------|--------|
| [Skill from JD] | Where it appears or "Not mentioned" | ✅/⚠️/❌ |

List missing keywords to add.

## Special Cases

- **Career changers**: Emphasize transferable skills, reframe with target industry language
- **New grads**: Focus on projects/internships/coursework, strictly 1 page
- **Executives**: Include P&L responsibility, board roles, publications; 2 pages OK
- **Employment gaps**: Suggest constructive framing, include relevant volunteer work

## Tone

Be encouraging — job searching is stressful. Balance honest critique with recognition of strengths. Always end on an actionable, positive note.
