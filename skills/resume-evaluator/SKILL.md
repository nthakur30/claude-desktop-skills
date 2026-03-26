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
> 4. **Generate improved version** — Create an improved resume file
> 5. **Done for now** — We're finished
>
> Pick a number or tell me what you need.

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
