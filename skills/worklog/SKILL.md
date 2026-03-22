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
pnpm dev start oracle-ticket "<goal>" "<steps>"
```

**Watch mode (auto-rebuild on changes):**

```bash
pnpm build --watch
```

---

## How to handle each command

### Start a session

1. Ask the user: "What is your goal?" and "What are the steps?"
2. Rewrite their answer — fix typos, grammar, remove filler words, make it clear and specific
3. Show the rewritten version and ask: "Here's what I'll log — shall I save this?"
4. After they confirm, run:

```bash
worklog start <taskName> "<rewritten goal>" "<rewritten steps comma separated>"
```

### Add a note

Take what the user says, rewrite it, run directly — no confirmation needed:

```bash
worklog note <taskName> <rewritten note>
```

### End a session

1. Ask the user these 4 questions one by one: "What did you complete?", "What did you not complete?", "Why?", "What is the exact next step?"
2. Rewrite each answer
3. Show all 4 and ask: "Here's what I'll log — shall I save this?"
4. After they confirm, run:

```bash
worklog end <taskName> "<completed>" "<not completed>" "<why>" "<next step>"
```

The next step must be a specific, immediately actionable sentence. Not "continue work" — "Open query.sql and fix the JOIN on user_id".

### Resume a task

```bash
worklog resume <taskName>
```

Read the output and tell the user their next step clearly.

## Rewriting examples

| User says                                       | You pass to CLI                                    |
| ----------------------------------------------- | -------------------------------------------------- |
| "close that oracle thing"                       | "Close the Oracle support ticket"                  |
| "check logs and see whats wrong maybe fix it"   | "Check API logs, identify root cause, apply fix"   |
| "the thing keeps breaking when i send big data" | "Service times out on large payloads"              |
| "continue with the db stuff"                    | "Open query.sql and add index on requests.task_id" |
