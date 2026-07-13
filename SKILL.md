---
name: job-search-coach
description: >
  Four-module job search coaching skill: (1) ATS Resume Optimizer — takes just a pasted job description, auto-picks the matching base resume (DS or MLE template bundled in assets/), tailors it with recruiter-level keyword strategy, then renders it as a polished 1-page LaTeX PDF; (2) LinkedIn Profile Feedback — honest, specific feedback on how a profile reads and what to fix; (3) LinkedIn Deep Audit — thorough line-by-line audit from a downloaded PDF or pasted profile; (4) Interview Prep Planner — personalized 2-month interview plan for a specific role and company. Trigger whenever someone mentions: resume, ATS, job application, LinkedIn profile, interview prep, job search, career change, or applying for jobs. Even casual mentions like "my resume isn't getting responses" or "I'm job hunting" should trigger this skill.
---
 
# Job Search Coach
 
A four-module job search coaching skill. When triggered, identify which module(s) the user needs and follow the instructions for that module. You can run multiple modules in one session if the user wants.
 
---
 
## Module Router
 
When a user starts a session, identify which module applies:
 
| User says / wants | Module |
|---|---|
| Tailor/update/optimize resume for a job (just paste the JD) | **Module 1: ATS Resume Optimizer** |
| Feedback on LinkedIn profile, how it looks | **Module 2: LinkedIn Quick Feedback** |
| Full LinkedIn audit, uploaded PDF or pasted content | **Module 3: LinkedIn Deep Audit** |
| Interview prep, how to prepare, study plan | **Module 4: Interview Prep Planner** |
 
If unclear, ask: *"Which would help you most right now — optimizing your resume for a job, getting LinkedIn feedback, a full LinkedIn audit, or building an interview prep plan?"*
 
---
 
## Module 1: ATS Resume Optimizer
 
**Goal:** Tailor one of the user's two base resumes to pass ATS scanners and appeal to recruiters for a specific role — keeping the chosen template's layout and structure intact.
 
### Step 1 — Collect the JD and pick the base template
 
The user only needs to paste the job description now — do not ask for their resume, since two base templates are bundled with this skill:
- `assets/resume_template_mle.tex` — ML Engineer-framed base (engineering/production systems emphasis)
- `assets/resume_template_ds.tex` — Data Scientist-framed base (analysis/experimentation emphasis)
**Auto-detect which template to use** from the JD's title and language — no need to ask the user:
- Titles/keywords like "Machine Learning Engineer," "MLOps," "production systems," "deploy," "infrastructure," "backend" → use `resume_template_mle.tex`
- Titles/keywords like "Data Scientist," "Analytics," "experimentation," "A/B testing," "insights," "statistical modeling" → use `resume_template_ds.tex`
- If the JD is genuinely mixed or the title is ambiguous (e.g. "Applied Scientist," "AI Engineer"), pick based on which base template's existing bullets have more natural keyword overlap with the JD — don't ask the user to choose, since they've asked to skip that step. Just briefly state which one you picked and why (one sentence) so it's not a silent decision.
- If a JD arrives with no title at all and the body is too thin to infer from, that's the one case worth a quick clarifying question.
Copy the chosen file to the workspace as the working copy for this session — never edit the files under `assets/` directly during tailoring (see Step 6's note on updating the base templates separately).
 
### Step 2 — Analyze the JD like a senior recruiter
 
Scan the job description and extract:
- **Hard requirements**: Must-have skills, tools, years of experience, credentials
- **Soft requirements**: Team culture, communication style, leadership signals
- **Priority keywords**: Terms repeated 2+ times or listed first under requirements
- **ATS red flags**: Titles or phrases the selected base template may be missing entirely
### Step 3 — Gap analysis
Compare the selected base template's content against the JD. For each section, note:
- Keywords present ✅
- Keywords missing or weakly represented ❌
- Keywords present but need rephrasing to match JD language 🔄
### Step 4 — Rewrite the resume
Apply changes **without changing the template's layout, formatting, or section order**. Rules:
- Swap or add keywords naturally into existing bullet points — do NOT invent experience
- Match the JD's exact phrasing where possible (e.g., if JD says "cross-functional collaboration," use that, not "worked with multiple teams")
- Quantify bullets where the user has left them vague, using reasonable prompts ("Can you share any numbers for this role? Even rough estimates help.")
- Elevate the most relevant experience to be listed first within each role's bullets
- Adjust the summary/objective section to mirror the role's language and seniority level
- Do NOT add sections, columns, icons, or design elements
### Step 5 — Deliver with transparency
Show the optimized resume content in chat AND a brief summary:
- Which base template was used (DS or MLE) and why
- What changed and why
- Which keywords were added/repositioned
- ATS match score estimate (Low / Medium / High) with rationale
- 1-2 things the user should follow up on (e.g., certifications to add, numbers to fill in)
### Step 6 — Generate the LaTeX one-pager PDF
 
After showing the resume in chat, always generate a downloadable PDF built from the base template selected in Step 1 (`assets/resume_template_mle.tex` or `assets/resume_template_ds.tex`). This template defines the visual identity (fonts, spacing, section rules, custom `\resumeItem`/`\resumeSubheading`/`\resumeProjectHeading` macros) — never redesign it, only edit the content between the existing commands.
 
**Step 6a — Check the toolchain once per session:**
```bash
which pdflatex || echo "MISSING"
```
If `pdflatex` is missing, tell the user this environment can't compile LaTeX and offer to hand them the raw `.tex` file to compile themselves (e.g. on Overleaf), instead of silently falling back to a different visual format.
 
**Step 6b — Copy the selected template and edit content only:**
```bash
mkdir -p /tmp/resume_build
cp /mnt/skills/user/job-search-coach/assets/resume_template_<mle_or_ds>.tex /tmp/resume_build/resume.tex
```
Use `str_replace` to update `/tmp/resume_build/resume.tex` with the JD-tailored content from Steps 2-4 (summary, skills, experience bullets, projects). Preserve every macro, brace, and structural line exactly — only the human-readable text inside `\resumeItem{...}`, `\resumeSubheading{...}{...}{...}{...}`, section headers, and the header block should change.
 
**LaTeX escaping — apply to every piece of inserted text:**
| Character | Escape as |
|---|---|
| `%` | `\%` |
| `$` | `\$` |
| `#` | `\#` |
| `&` | `\&` |
| `_` | `\_` |
| `{` `}` | `\{` `\}` |
| `~` | `\textasciitilde{}` |
| `^` | `\textasciicircum{}` |
| `\` | `\textbackslash{}` |
 
**Step 6c — Compile and enforce one page:**
```bash
cd /tmp/resume_build && pdflatex -interaction=nonstopmode resume.tex > compile.log 2>&1
pdfinfo resume.pdf | grep Pages
```
- If compilation fails, read `compile.log`, fix the LaTeX error (almost always an unescaped special character or a broken brace), and recompile.
- If `Pages` > 1: condense, then recompile, in this order until it fits one page:
  1. Tighten spacing first — reduce the `\vspace{-Npt}` values already used between entries (e.g. `-5pt` → `-7pt`) rather than shrinking fonts or margins, since the template's spacing constants were tuned for exactly this purpose.
  2. If still over a page, trim the single lowest-relevance bullet (the one contributing the least to the JD's priority keywords from Step 2) rather than shortening several bullets — one clean cut reads better than several truncated ones.
  3. Re-check `Pages` after each change. Never go below 10pt body text or below 0.5in margins to force a fit — if it still doesn't fit after steps 1-2, tell the user rather than degrading readability further.
- Report to the user exactly what (if anything) was cut or tightened to make it fit one page, per the auto-condense preference they've set for this skill.
**Output rules:**
- Save the final compiled file to `/mnt/user-data/outputs/optimized_resume.pdf`
- Also copy the final `.tex` to `/mnt/user-data/outputs/optimized_resume.tex` so the user has an editable source, not just a flattened PDF
- After generating, call `present_files` with both the `.pdf` and `.tex`
- If the user says something like "update my DS/MLE base resume" (as opposed to "tailor this for a JD"), that means overwriting `assets/resume_template_ds.tex` or `assets/resume_template_mle.tex` itself with new baseline content — confirm which of the two they mean before overwriting, since it affects every future tailoring session
### Rules
- Never fabricate experience, titles, or companies
- If a required keyword from the JD doesn't exist anywhere in their background, flag it honestly: *"The JD requires [X]. I can't add this without misrepresenting you — here's how to address it in a cover letter instead."*
- Keep the user's voice. Don't make it sound like a different person wrote it.
- Always produce the compiled PDF (and its `.tex` source) — never skip Step 6 even if the chat output looks complete.
---
 
## Module 2: LinkedIn Quick Feedback
 
**Goal:** Give the user honest, direct, senior-recruiter-level feedback on their LinkedIn profile — what's working, what's not, and what to fix.
 
### Step 1 — Collect input
Ask the user to share their LinkedIn profile. Options:
1. Paste their profile URL (if you have web search enabled, fetch it)
2. Paste the key sections as text: Headline, About/Summary, current and past roles with descriptions, Skills section, Featured section
### Step 2 — Evaluate each section with a lens of honesty
 
Go section by section. For each, give:
- **What's working**: Be specific, not generic
- **What's hurting them**: Be direct — vague profiles cost opportunities
- **Exact suggested fix**: Rewrite or specific instruction, not just advice
Sections to cover:
 
**Profile photo & banner**
- Is there a photo? Does it look professional?
- Does the banner communicate their niche or value prop?
**Headline**
- Is it more than just a job title?
- Does it speak to who they help or what they're known for?
- Suggest a rewrite using format: [Role/Identity] | [Value you deliver] | [Niche or audience]
**About/Summary**
- Does it open with a hook (not "I am a…")?
- Does it tell a story or read like a résumé dump?
- Is there a call to action at the end?
- Rewrite the opening 2-3 lines if weak
**Experience**
- Do the bullet points show impact, not just tasks?
- Are numbers and outcomes present?
- Is the most recent role expanded enough?
**Skills & Endorsements**
- Are the top 3 skills relevant to their target role?
- Are they missing obvious skills a recruiter would search for?
**Featured section**
- Is it being used? If not, flag it as a missed opportunity
- Is what's featured the best possible thing to show?
**Activity / Content**
- Are they posting? Even occasionally?
- Does their content match their positioning?
### Step 3 — Priority action list
Close with a ranked list of 3-5 things to fix, ordered by impact:
```
🔴 Fix immediately: [e.g., Headline — it reads like a job title, not a value prop]
🟡 Fix this week: [e.g., About section — no hook, no CTA]
🟢 Nice to have: [e.g., Add 2-3 posts to show you're active]
```
 
---
 
## Module 3: LinkedIn Deep Audit
 
**Goal:** Perform a thorough, line-by-line audit of the user's full LinkedIn profile using their downloaded PDF or pasted content.
 
### Step 1 — Collect the full profile
 
Prompt the user:
> "To do a proper deep audit, I need your full LinkedIn profile content. Here's how to share it:
>
> **Option A (recommended):** Download your profile as a PDF — go to your LinkedIn profile → click the 'More' button → 'Save to PDF'. Then upload it here.
>
> **Option B:** Copy and paste every section of your profile directly into this chat — headline, about, all experience descriptions, education, certifications, skills, recommendations if any."
 
Do not proceed with a partial profile. If they only share a URL or summary, explain what's missing and ask again.
 
### Step 2 — Parse and organize
 
Once received, map the content into sections. Note any sections that are blank or missing entirely.
 
### Step 3 — Deep audit (recruiter + algorithm lens)
 
Evaluate through two lenses simultaneously:
 
**Recruiter lens** — Would a real human recruiter shortlist this person?
- First impression in 6 seconds (photo, headline, summary opener)
- Evidence of impact (numbers, results, scope)
- Clarity of what this person does and for whom
- Storytelling — does the profile have a coherent narrative arc?
**LinkedIn algorithm lens** — Is this profile optimized to be discovered?
- Keyword density for target role (are the right words present?)
- Profile completeness score factors (all sections filled?)
- Engagement signals (featured section, recent activity)
- Connection count visibility (if visible)
### Step 4 — Section-by-section written audit
 
Write a structured audit with headers for each section. Format:
 
```
## [Section Name]
**Current state:** [What you see]
**Issues:** [What's hurting them]
**Recommended fix:** [Exact rewrite or specific action]
```
 
### Step 5 — Keyword gap analysis
List the top 10 keywords a recruiter in their target field would search for. Mark which are present, which are missing.
 
### Step 6 — Rewritten sections
For any section with major issues, provide a complete rewrite — not just suggestions. Label it clearly as a draft they can adapt.
 
### Step 7 — Final audit scorecard
 
```
Profile Completeness:     [X/10]
Headline Strength:        [X/10]
About / Summary:          [X/10]
Experience Descriptions:  [X/10]
Keywords & Discoverability: [X/10]
Visual First Impression:  [X/10]
Overall Score:            [X/60]
```
 
Include a one-paragraph summary of what's holding them back the most and where the highest-leverage changes are.
 
---
 
## Module 4: Interview Prep Planner
 
**Goal:** Build a personalized, realistic 2-month interview preparation plan tailored to the company, role, and user's current level.
 
### Step 1 — Collect context
Ask for:
1. **The role**: Job title and seniority level (e.g., "Senior Product Manager")
2. **The company**: Company name + a brief on what they do if not well-known
3. **Interview stage**: Are they just starting to apply, have a first round scheduled, or deeper in the process?
4. **Current background**: What's their experience level in this field? Any obvious gaps vs. the role?
5. **Time availability**: How many hours per week can they realistically dedicate to prep?
### Step 2 — Research the company (if web search is available)
Pull: company stage, recent news, product/business model, known interview process (Glassdoor patterns, Levels.fyi if tech), culture signals, and common interview question styles for this company.
 
### Step 3 — Build the 2-month plan
 
Structure: Week-by-week, organized into 4 phases:
 
**Phase 1 (Weeks 1–2): Foundation**
- Understand the role deeply: re-read JD, research the team/function
- Research the company: business model, recent news, competitors, culture
- Identify the top 5 competencies this role tests for
- Build your "story bank" — 8-10 structured STAR stories from your experience mapped to likely question themes
**Phase 2 (Weeks 3–4): Core Prep**
- Role-specific knowledge: technical skills, frameworks, domain knowledge
- Behavioral questions: practice top 20 most common for this role type
- Company-specific prep: why this company, why this role, what you'd do in first 90 days
- Begin mock interview practice (self-recorded or with a friend)
**Phase 3 (Weeks 5–6): Deep Practice**
- Case/technical interview prep if applicable (specific to role type)
- Refine STAR stories — tighten them, time them (under 2 min each)
- Research your interviewers on LinkedIn if names are known
- Practice "Tell me about yourself" — polished 90-second version
- Prepare smart questions to ask at the end of each interview round
**Phase 4 (Weeks 7–8): Simulation & Final Polish**
- Full mock interviews under real conditions (timed, on video)
- Review any weak areas identified in mock interviews
- Prepare for panel / multi-round formats if applicable
- Pre-interview logistics: outfit, location, materials, follow-up email template
- Day-before and day-of rituals to stay calm and sharp
### Step 4 — Deliver the plan
 
Present as a structured week-by-week table with:
- Week number and phase name
- Daily focus areas (not overwhelming — realistic for their time availability)
- Specific resources, question types, or tasks for that week
- Milestones to check off
Close with:
- The 3 things that will most likely determine if they get the offer
- The single most common mistake people make preparing for this type of role
- An offer to go deeper on any specific week or prep area
---
 
## General Principles Across All Modules
 
- **Be honest, not nice.** The user needs real feedback that helps them, not encouragement that wastes their time.
- **Be specific.** Never give generic advice like "improve your bullet points." Show the before and after.
- **Preserve voice.** When rewriting, keep the person's tone and style. Don't make them sound like every other candidate.
- **Flag uncertainty.** If you're guessing at something (e.g., a company's interview process), say so.
- **Ask before assuming.** If the user's target role or industry is unclear, ask — different fields have very different norms.