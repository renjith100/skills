---
name: worklog
description: Use this skill when the user wants to log work, track a task session, start deep work, capture notes during work, end a session with a review, or resume a previous task. Also use when the user says "I'm going to work on X", "log my work", "what did I do on X", or "continue working on X".
metadata:
  tags: [productivity, worklog, cli, deep-work, session-tracking]
---

# Worklog CLI Skill

A CLI tool for structured deep work session tracking. Built around a 4-phase workflow:
**Plan → Execute → Reflect → Continue**. The core problem it solves is "memory reset" —
losing context when returning to a task after 1–2 days away.

> "Maintaining momentum and reducing restart friction."

---

## Storage

All files live at `~/worklogs/` — fixed home directory, regardless of where the command is run.

```text
~/worklogs/
  <task-name>/
    session-001-2026-03-22.md
    session-002-2026-03-23.md
    session-003-2026-03-24.md
```

One folder per task. One file per deep work session. Sessions are zero-padded and numbered
sequentially (session-001-YYYY-MM-DD, session-002-YYYY-MM-DD, …).

---

## Installation

```bash
cd /path/to/worklog-cli
pnpm install
pnpm build
pnpm link --global
```

After linking, `worklog` is available globally from any directory.

**Dev mode (no build needed):**

```bash
pnpm dev start oracle-ticket
```

**Watch mode (auto-rebuild on changes):**

```bash
pnpm build --watch
```

---

## Commands

### `worklog start <taskName>`

Creates a new numbered session file. Prompts interactively for:

1. `🎯 Goal:` — what this session aims to achieve (free text)
2. `🧩 Steps (comma separated):` — concrete planned steps

Output: `✅ Session created: <file-path>`

---

### `worklog note <taskName> <note text...>`

Appends a note to the `## 🧠 Notes During Work` section of the latest session.
No prompts — all text after the task name becomes the note.

```bash
worklog note oracle-ticket Timeout occurs after 30s on large payloads
```

Use for: blockers, ideas, findings, observations mid-session. Zero friction — no piping needed.

---

### `worklog end <taskName>`

Closes the session with a structured review. Prompts interactively for:

1. `Completed:` — what was actually done
2. `Not Completed:` — what was planned but skipped
3. `Why:` — honest, factual reason
4. `Next Step:` — **the most critical field** — exact, specific next action

> ⚠️ **Next Step must be specific and actionable.**
> Good: `"Open query.sql and optimize the slow JOIN on user_id"`
> Bad: `"Continue debugging"` / `"Fix the bug"` / `"Continue work"`

---

### `worklog resume <taskName>`

Reads the latest session and prints the `▶️ Next Step (CRITICAL)` content.
No prompts. No input needed.

```bash
worklog resume oracle-ticket
```

Output:

```text
=== LAST SESSION ===

/Users/renjith/worklogs/oracle-ticket/session-003-2026-03-22.md

▶️ Next Step:

Open query.sql and optimize the slow JOIN on user_id
```

Use this at the start of any work block to instantly restore context.

---

## Session File Structure

Every session file uses this exact template:

```markdown
# Session N - <taskName>

Date: 22 Mar 2026, 6:45 pm

## 🎯 Goal

<goal>

## 🧩 Steps

[ ] step 1
[ ] step 2
[ ] step 3

## 🧠 Notes During Work

- <notes appended via `worklog note`>

## 📊 Review

### ✅ Completed

<what was done>

### ❌ Not Completed

<what was skipped>

### 🤔 Why

<honest reason>

### ▶️ Next Step (CRITICAL)

<exact next action — drives the resume command>
```

---

## Agent Guide

> **Prerequisite:** The `worklog` CLI must be globally installed. Run `worklog` to verify.
> If the command is not found, stop and ask the user to install it first:
>
> ```bash
> cd /path/to/worklog-cli && pnpm install && pnpm build && pnpm link --global
> ```

### The agent IS the formatting layer

The user will speak naturally — messily, incompletely, with filler words. The agent's
responsibility is to always reformat and restructure the user's input into clean, logical,
well-written text before passing it to the CLI. Never pass raw user speech directly.

```text
User:  yeah so like the thing keeps breaking whenever i send big data,
       seems like maybe 30 seconds or something and then it just dies

Agent: [formats, then runs]
       worklog note oracle-ticket Timeout after ~30s on large payloads — likely a query or connection limit
```

This applies to every command — goals, steps, notes, review answers. The agent cleans,
condenses, and makes the text logically coherent before the CLI stores it.

---

### Starting a session

`worklog start` is an interactive command. Collect the information from the user first,
format it, then pipe it via stdin.

**Step 1 — Ask the user:**

> "What is your goal for this session?"
> "What are the steps? List them and I'll format them for you."

**Step 2 — Format the user's answers, then run:**

```bash
printf "Close the oracle support ticket\nReproduce the bug, check the API logs, find the root cause, apply the fix\n" \
  | worklog start oracle-ticket
```

**Step 3 — Confirm back to the user:**

> "Session started. Here's what was logged:
>
> - **Task:** oracle-ticket
> - **Goal:** Close the oracle support ticket
> - **Steps:** Reproduce the bug / Check the API logs / Find the root cause / Apply the fix
>
> Good luck. Tell me when you want to log a note during your session."

**Full example conversation:**

```text
User:  I want to start a work session on my oracle ticket.

Agent: Sure. Two quick questions:
       1. What is your goal for this session?
       2. What are the steps?

User:  Goal is to close the oracle support ticket.
       Steps — reproduce the bug, check the API logs, find the root cause, apply the fix.

Agent: [formats and runs internally]
       printf "Close the oracle support ticket\nReproduce the bug, check the API logs, find the root cause, apply the fix\n" \
         | worklog start oracle-ticket

Agent: Session started. Here's what was logged:
       - Task: oracle-ticket
       - Goal: Close the oracle support ticket
       - Steps: Reproduce the bug / Check the API logs / Find the root cause / Apply the fix

       Good luck. Tell me when you want to log a note.
```

---

### Adding notes during work

No interaction needed — format and run directly:

```bash
worklog note oracle-ticket Found root cause — missing index on requests table
```

Always reformat the user's raw input into a clean, concise note before running:

```text
User:  note that the thing is timing out after 30 seconds when the payload is big

Agent: [formats, then runs]
       worklog note oracle-ticket Timeout after ~30s on large payloads — likely a query or connection limit
```

---

### Ending a session

`worklog end` is an interactive command. Collect, format, then pipe all four answers.

**Step 1 — Ask the user:**

> "What did you complete?"
> "What did you not complete?"
> "Why?"
> "What is the exact next step?" _(must be specific — not "continue work")_

**Step 2 — Format the answers, then run:**

```bash
printf "Found root cause — missing index on requests table\nFix not applied\nRan out of time\nOpen query.sql and add index on requests.task_id\n" \
  | worklog end oracle-ticket
```

**Step 3 — Confirm back to the user:**

> "Session closed. Next step saved:
> _Open query.sql and add index on requests.task_id_"

---

### Resuming / reading context

```bash
worklog resume oracle-ticket
```

Read the output and surface the Next Step clearly to the user.

---

## Reading All Sessions

For analysis, summarization, or pattern detection:

```bash
# All sessions for a task
ls ~/worklogs/oracle-ticket/
cat ~/worklogs/oracle-ticket/session-001-2026-03-22.md

# All tasks
ls ~/worklogs/
```

---

## Design Philosophy

- **Light discipline, not heavy process** — simple forced questions, consistent answers
- **Filesystem = database** — plain markdown, no hidden state, always readable
- **Next Step is everything** — the entire continuity mechanism depends on one specific sentence
- **Notes prevent memory reset** — capture any thought mid-session, no friction
- **Accountability by design** — session end forces honest reflection on what actually happened

---

## Future Roadmap (not yet implemented)

- Voice input support
- Weekly summary generator agent
- Pattern detection (e.g. "you often get blocked at the debugging stage")
