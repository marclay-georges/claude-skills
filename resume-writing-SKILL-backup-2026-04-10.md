---
name: resume-writing
description: >
  Write, tailor, and refine resumes for job applications. Use this skill whenever the user asks to
  create a resume, update a resume, tailor a resume to a job description, write or polish resume
  bullets, compare a resume against a job posting, optimize for ATS, or do any resume-related work.
  Triggers include: "resume", "CV", "job application", "tailor my resume", "write bullets for",
  "how does my resume compare to this JD", "ATS optimization", "keyword match", or any request
  involving resume content, formatting, or strategy. Also use when the user shares a job description
  and wants to position their experience against it.
---

# Resume Writing

Write accurate, well-structured resumes tailored to specific job descriptions. The core philosophy:
every bullet should be truthful, quantified where possible, and positioned to match what the
target role actually asks for.

## Core Principles

### Accuracy Over Impact

Never overstate ownership, scope, or technical depth. The user's real experience, precisely
described, is stronger than inflated language that crumbles in an interview.

Common overclaims to watch for:
- "Architected" when the person designed requirements but didn't build the system
- "Led" when the person contributed to or participated in
- "Built" when the person specified and evaluated
- "Owned" when the person was responsible but not accountable

Common undersells to watch for:
- "Collaborated" when the person actually drove cross-functional work
- "Assisted" when the person was the primary contributor
- "Supported" when the person designed the solution

When in doubt, ask. Present the proposed bullet and let the user confirm whether it matches their
actual level of involvement. Propose changes before applying them — the user should always review
framing adjustments.

### The What → How → Impact Structure

Every bullet should follow this general flow:

**What** you did → **How** you did it (tools, methods, approach) → **Impact** (result, scope, or
business value — ideally with a number)

This isn't a rigid template. The three elements can be reordered for better sentence flow (the user
may prefer leading with context, e.g. "Serving as the analytical backbone of an AI-powered BI
platform, designed requirements and prompt strategies..."). But all three elements should be present
in some form.

**Example:**
> Developed Python ELT pipelines [what] to migrate 3B+ records from Oracle/Teradata into
> Snowflake [how], reducing manual data reconciliation by 10 hours per week [impact]

### Quantification Target: 75%+

At least 75% of bullet points should include a number. Numbers ground abstract descriptions in
real scale and make bullets memorable to reviewers.

Good sources of numbers to ask the user about:
- Records processed, users served, models maintained
- Time saved, error reduction percentages
- Number of dashboards, pipelines, data sources, clients, stakeholders
- Dollar values (infrastructure costs, program budgets, revenue impacted)
- Frequency (weekly campaigns, daily pipeline runs)

Even rough estimates are better than no number at all. "~30 campaigns per week" beats a
generic description of "monitored marketing campaigns." When asking for metrics, frame it as
"what's a reasonable ballpark?" — precision isn't the point, scale is.

### Voice and Flow Preservation

When the user provides their own phrasing or preferred sentence structure, preserve it. Polish
grammar, fix awkward constructions, and tighten language — but don't rewrite the sentence into
a "stronger" version that changes the rhythm. If the user prefers a participial opening over a
standard action-verb lead, that's their voice. Respect it.

The goal of editing is to bring the user's version up to professional polish, not to replace it
with yours.

### Iterative Refinement Over Wholesale Rewrites

Work incrementally. When editing an existing resume:
- Propose specific changes and explain the reasoning
- Wait for approval before applying
- Make one type of change at a time (e.g., verb swaps first, then quantification, then keyword
  alignment) rather than rewriting everything at once

## Resume Tailoring Workflow

When the user has a target job description, follow this sequence:

### 1. Requirement-by-Requirement Gap Analysis

Go through each requirement in the JD and assess where the resume speaks to it, where it's
partially covered, and where there are genuine gaps. Be specific about what's missing and what's
strong. Structure this as:

- **Strong matches** — requirements clearly covered by existing bullets
- **Partial matches** — experience exists but framing doesn't highlight it
- **Gaps** — requirements not reflected in the resume at all

For gaps, ask the user whether they have relevant experience that isn't on the resume before
concluding it's a real gap. People often have experience they haven't thought to include.

### 2. Bullet-to-Bullet Matching

The strongest resumes don't just cover the JD's requirements in aggregate — they address specific
JD bullet points directly and visibly. A hiring manager or recruiter scanning the resume should be
able to draw a mental line from each JD responsibility to a resume bullet that clearly speaks to it.

The ideal outcome is a near one-to-one correspondence: for each key bullet in the JD, there's a
resume bullet that a reader would recognize as directly relevant. This doesn't mean copying JD
language verbatim or forcing artificial matches — it means ensuring the resume's strongest, most
relevant bullets are framed in a way that maps clearly to what the JD asks for.

How to do this:
- List the JD's key responsibilities and qualifications as discrete items
- For each one, identify which resume bullet is the best match (or could become one with reframing)
- Where a single resume bullet covers multiple JD items, that's fine — but flag JD bullets that have
  no clear resume counterpart
- Reframe existing bullets to make the connection more explicit when the experience is there but the
  framing doesn't surface it
- Don't force matches that aren't real. Robust coverage across the JD matters more than a strained
  one-to-one mapping. If 80% of JD bullets have a clear resume counterpart and the remaining 20% are
  partially covered, that's a strong resume

This is the highest-leverage tailoring step. ATS keyword matching gets the resume past automated
filters, but bullet-to-bullet alignment is what makes a human reader think "this person has done
exactly what we need."

### 3. Keyword Alignment

Compare the JD's terminology against the resume's language. Flag mismatches where the user has
the experience but uses different words:
- "forecasting" vs "forecast"
- "optimization" vs "optimize"
- "workforce management" vs "staffing"

Use the JD's exact phrasing where it accurately describes the user's experience. Don't force
keywords into places they don't fit — forced keyword stuffing reads poorly to humans even if it
passes ATS.

### 4. Bullet Refinement

With the gap analysis and keyword alignment done, propose specific bullet edits:
- Reframe partial-match bullets to better highlight JD-relevant experience
- Add new bullets only if the user confirms they have the experience to support them
- Reorder bullets within a role to lead with the strongest JD matches

### 5. ATS Optimization Pass

After content is finalized:
- Check for verb repetition across the full resume (same leading verb used 3+ times is a flag)
- Verify quantification coverage (75%+ target)
- Confirm keyword density against the JD's critical requirements
- Ensure consistent formatting (punctuation, tense, bullet length)

## Working with Existing Materials

The user may have a bullet bank — a collection of pre-written bullet variations organized by
company and target role. When this exists:
- Treat it as raw material, not finished product
- Pull relevant bullets and adapt them to the target JD
- Don't assume bullet bank entries are polished — they may need the same accuracy and
  quantification checks as any new bullet

The user may also have multiple resume versions targeting different role types. Confirm which
version is the starting point before editing.

## What This Skill Does NOT Do

- Fabricate metrics or experiences the user hasn't confirmed
- Add tools or technologies the user hasn't verified they use
- Inflate job titles or tenure
- Write cover letters (that's a separate task, though the resume content informs it)
- Make formatting or layout decisions (focus is on content and language)
