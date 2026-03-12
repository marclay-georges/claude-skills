---
name: conversation-context-export
description: >
  Export conversation context so it can be continued in a new chat. Use this skill whenever the user
  wants to save, export, snapshot, or package the current conversation's context for use in another chat.
  Triggers include: "export this conversation", "save context", "context handoff", "continue this later",
  "package this up", "conversation snapshot", "I want to pick this up in a new chat", "context export",
  or any indication the user wants to preserve the current conversation state for continuation elsewhere.
  Also use when the user says "let's wrap up" or "bookmark this" in a way that implies future continuation.
---

# Conversation Context Export

Generate a context document that enables seamless continuation of the current conversation in a new chat.

## What This Skill Is For

Conversations build up shared state — decisions, discoveries, constraints, work products, failed attempts,
open questions. When a user starts a new chat to continue the same work, all of that state is lost. This
skill captures what matters so the next session can pick up without backtracking, re-explaining, or
re-discovering things.

## The Audience

The export is written for a future Claude instance that has access to the user's memory (persistent
preferences, background, key facts) but knows nothing about this specific conversation. This distinction
matters:

- **Don't re-state** what memory already covers — the user's job, tools, communication preferences, etc.
- **Do capture** everything conversation-specific: what was being worked on, what was tried, what was
  decided, what's still open, and what was produced.

Think of it as the difference between who someone *is* (memory) and what they were *doing* (conversation state).

## What Makes a Good Context Export

### Continuity Over Completeness

The goal isn't to summarize the entire conversation — it's to preserve the minimum state needed for
unbroken continuity. A good export lets the next session start with "Okay, picking up where we left off"
rather than "Can you remind me what we were doing?"

Ask yourself: *If I were the next Claude instance reading this, could I act immediately without asking
clarifying questions about what happened before?*

### Specificity Over Generality

Vague: "We worked on the user's Notion setup."
Specific: "We restructured the Life Assistant Inbox page (ID: 2df0bbbb-5a82-81e0-bc3a-cfa8d036adc3) to
use a linked database with filters for priority and status. The database was created but filters haven't
been configured yet."

Always include identifiers, paths, IDs, URLs, error messages, code snippets, and configuration values.
These are the things that are impossible to reconstruct from a general description.

### Capture Negative Results

What was tried and didn't work is often more valuable than what succeeded. Failed approaches prevent
the next session from wasting time re-attempting them. Include the *why* — "We tried X, it failed because
Y" is useful; "We tried X" alone isn't.

### Decisions Need Rationale (When Non-Obvious)

"We chose PostgreSQL" is incomplete. "We chose PostgreSQL over SQLite because the app needs concurrent
write access from multiple workers" lets the next session understand the constraint, not just the choice.
If the reason is obvious, skip the rationale. Use judgment.

### State, Not Story

Write the export as a reference document, not a narrative of the conversation. Nobody needs to know the
order things were discussed, who suggested what, or how the conversation flowed. They need to know where
things stand *now*.

- Present tense for current state and open items
- Past tense for completed work and decisions
- No conversation meta-commentary ("the user asked about..." / "we then discussed...")

## How to Scale the Export

The export should be proportional to the conversation's complexity. Not every conversation needs every
kind of information. Use judgment:

**Simple conversation** (quick question, single task completed): A few sentences may be enough — what
was done, any relevant output, anything left open.

**Medium conversation** (multi-step task, some decisions): A structured document with clear sections
covering the work done, decisions, and next steps.

**Complex conversation** (long project work, many artifacts, branching decisions): A thorough document
with detailed state for each workstream, artifact inventory, and prioritized open items.

Don't force structure where it isn't needed. Don't omit structure where it is. The right length is
whatever it takes to preserve continuity — no more, no less.

## Information Categories

These are the *types* of information a context export might contain. Not all apply to every conversation.
Include what's relevant, omit what isn't, and organize in whatever way makes the content most scannable
and actionable.

- **What we're working on** — The project, task, or question at hand
- **What's been done** — Completed work, including how and where it lives
- **What was decided** — Choices made, options rejected, rationale
- **What didn't work** — Failed approaches, dead ends, and why
- **What's in progress** — Partially completed work, current blockers
- **What's still open** — Unresolved questions, deferred items, pending decisions
- **What was produced** — Files, pages, code, configurations, with paths/IDs/links
- **What was discovered** — Constraints, limitations, undocumented behaviors, surprises
- **What to do next** — Recommended actions for the next session, in rough priority order

## Delivery

Create the export as a markdown file and present it to the user. Also display the content inline so
they can copy-paste directly if that's easier. Include a timestamp and a brief title derived from the
conversation topic at the top for future reference.

If the user asks for something quick or brief, compress aggressively — just the essentials needed to
resume. Note that a full export is available if they want it.