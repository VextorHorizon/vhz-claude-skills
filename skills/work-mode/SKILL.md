---
name: work-mode
description: Project execution mode — loads a specific project's progress from Obsidian, tracks every completed task as a checkpoint, and writes progress back to Obsidian continuously throughout the session. Use when actively working on a project. Invoke with /work-mode [project-name].
---

# /work-mode — Project Execution Mode

## Overview

`/work-mode` turns the session into a **project execution session** for the specified project. Claude will:

1. Load current progress from Obsidian
2. Show status + next tasks
3. **Track throughout the session** — every completed task triggers a checkpoint written to Obsidian immediately
4. **Log every interaction as a raw step log** — appends a summarized entry to `raw/session-YYYY-MM-DD.md` after every exchange, for the user to ingest later
5. Close the session by writing a summary to the diary

Result: the next session can open `/work-mode` and continue without needing to recall context. The raw log is ready to ingest into the vault immediately.

---

## Vault Location

Resolved from `$OBSIDIAN_VAULT` environment variable at runtime.

If not set, ask the user for the vault path before proceeding.

---

## Invocation

```
/work-mode my-project    ← any project name
/work-mode               ← if omitted, show list and ask
```

Project resolution:

1. If argument given → look for `data/work/<argument>-progress.md` in vault
2. If not found → list all `*-progress.md` files in `data/work/` and let user pick
3. If none exist → create new progress file from standard template

No hardcoded project list — fully dynamic from the vault.

---

## Session Start — Phase 1: Load & Briefing

Run once when `/work-mode` is invoked.

**Steps:**

1. Read vault's CLAUDE.md
2. Read progress file for the selected project
3. Read/create today's daily note (`data/daily/YYYY-MM-DD.md`)
4. **Create raw session log file** — create `raw/session-YYYY-MM-DD.md` (if not exists) for step log:

```markdown
# Session Raw Log — YYYY-MM-DD

> Raw interaction log for ingesting into Obsidian vault later
> Records every step of interaction with Claude in this session

---

## Step 1 — Session Start: [Project Name]

**User:** Opened work-mode for [Project Name]
**Claude:** Loaded progress file, daily note, git sync — showing briefing Phase [X]

---
```

   If file already exists (reopening same day), append steps — do not overwrite.

5. **Git sync (if project has a git repo):**

```bash
git fetch --prune && git rebase origin/main
```

   Show user whether current branch is synced. If repo is in a different directory, tell user to run it manually.

5. Show briefing in this format:

```
## /work-mode — [Project Name]
**Current Phase:** Phase X — [phase name]
**Last updated:** DD-MM-YYYY

### Next Tasks (ready to start)
- [ ] Task 1
- [ ] Task 2

### Blocked (waiting on what)
- [blockers from progress file]

### Open Questions
Q1 ❓, Q2 ❓ — [question names]

---
Ready — tell me which task to start with
```

6. Write session start to daily note:

```markdown
## Session Summary
[work-mode started: Project Name — Phase X]
```

---

## During Session — Claude's Tracking Responsibility

**This is the core of the skill** — Claude must do these automatically throughout the session without waiting for the user to ask.

### Raw Step Log — Record every interaction (critical)

**Every time the user sends a message and Claude responds**, append a new entry to `raw/session-YYYY-MM-DD.md` immediately:

```markdown
## Step N — [short topic of this interaction]

**User:** [summary of user intent in 1-2 sentences]
**Claude:** [summary of what was done/decided/found in 2-4 bullets]
- [action/result 1]
- [action/result 2]
- [if applicable: files edited, answers given, decisions made]

---
```

**N** is a continuous step count for the session (continuing from last step in file if reopening same day).

**Record every interaction without exception** — even short answers, explanations, bug hunting, file edits — everything gets a step entry.

### When to update (Obsidian tracking)

| Event | Action |
|---|---|
| Every interaction | Append step entry to raw session file immediately |
| A task is fully complete | Check `- [x]` in progress file + write checkpoint to diary |
| A significant decision is made | Add row to Decisions table in progress file |
| An open question is answered | Update Q&A table in progress file (❓ → ✅ + answer) |
| A blocker or new problem found | Add to Blocked section + write checkpoint |
| Every 3–5 completed tasks | Write mini checkpoint to diary |

### Checkpoint Format (in daily note)

Add under `## Session Summary`:

```markdown
**[HH:MM] Checkpoint — [task name]**
- Done: [completed items]
- Found: [learnings during the work, if any]
- Next: [next task to do]
```

### Progress File Update Format

When task is done:
```markdown
- [ ] just completed task   →   - [x] just completed task ✓ (DD-MM-YYYY)
```

When question answered:
```markdown
| Q1 | question... | — | ❓ |   →   | Q1 | question... | [answer] | ✅ |
```

When decision made:
```markdown
| DD-MM-YYYY | [decision] | [reason] |
```

---

## Session End — Phase: Wrap Up

Run when user says "done", "stopping for today", "finished", or when session is naturally ending.

**Steps:**

1. Update `## Latest Session Notes` in progress file:

```markdown
## Latest Session Notes (DD-MM-YYYY)

- [what was completed today]
- [what was discovered / decided]
- Waiting on: [outstanding blockers]
- Next session: [first task to pick up]
```

2. Update `status:` in progress file frontmatter if phase changed

3. Write full session summary to daily note:

```markdown
## Session Summary
[work-mode: Project Name]
- ✅ Done: [list]
- 🔍 Found: [list]
- ❓ Still open: [list]
- ➡️ Next time: [first task to do]

## What We Learned
[what was learned from today's work]

## Claude's Observations
[observed patterns, risks, suggestions for next session]
```

4. **Close raw session log** — append final entry to `raw/session-YYYY-MM-DD.md`:

```markdown
## Session End — [Project Name]

**Session summary:**
- Done: [list of completed tasks]
- Found: [things learned or decided]
- Open: [blockers or incomplete work]
- Next time: [first task to pick up]

Total steps this session: [N]

---
```

5. Update `log.md`:

```
## [YYYY-MM-DD] update | work-mode: [Project Name]
Session summary: completed [N] tasks. Phase [X] status: [in-progress/complete]. Next: [next task].
```

6. Show closing summary:

```
## Session Done — [Project Name]

✅ Completed today: N tasks
📝 Progress saved to: data/work/[progress-file].md
📅 Daily note updated: data/daily/YYYY-MM-DD.md

Next time type /work-mode [project] to continue
```

---

## Rules

- **Don't wait for the user to ask** — Claude writes checkpoints automatically every time a task is done
- **Log every interaction to raw session log immediately** — no exceptions; every user message = 1 step entry in `raw/session-YYYY-MM-DD.md`
- **Write progress file immediately** — don't batch writes until the end of the session
- **Raw session file is not auto-ingested** — user will ingest it manually; do not delete or move this file
- **If progress file doesn't exist** — create from standard template (frontmatter + Phase + Next Tasks + Blocked + Q&A + Decisions sections)
- **If unsure whether a task is done** — ask before checking the checkbox
- **If user changes mind or reverts work** — uncheck the checkbox and note why
- **Checkpoint timestamps use real time** where possible; if not available, use sequence numbers (Checkpoint 1, 2, 3...)
- **Always git sync before starting (if project has a repo):** `git fetch --prune && git rebase origin/main`
