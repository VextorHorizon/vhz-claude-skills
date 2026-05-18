---
name: obsidian-vault
description: Use when the user wants to work with their Obsidian second brain vault — ingesting notes, querying knowledge, or running a wiki health check. Daily notes are Claude's session diary (not the user's). Vault path is read from the OBSIDIAN_VAULT environment variable.
---

# Obsidian Vault

## Setup

Set `OBSIDIAN_VAULT` to your vault path in your shell profile or Claude Code env settings:

```
# Windows
$env:OBSIDIAN_VAULT = "C:\Users\you\Documents\my-vault"

# macOS / Linux
export OBSIDIAN_VAULT="$HOME/Documents/my-vault"
```

If not set, Claude will ask for the vault path before proceeding.

## Overview

Loads the personal second brain vault at `$OBSIDIAN_VAULT` into context and activates all wiki workflows defined in the vault's CLAUDE.md.

**Vault path:** resolved from `$OBSIDIAN_VAULT` at runtime

**Daily notes are Claude's diary — not the user's.** The user does not write in them. Claude writes session summaries, learnings, task tracking, and observations to the daily note during and after each session.

## Session Start — Run This First

Resolve `VAULT = $OBSIDIAN_VAULT`. If unset, ask user for the path.

When this skill is invoked, execute these five steps in order before responding to any task:

1. **Read CLAUDE.md** — `$VAULT/CLAUDE.md` — load schema and all workflow rules
2. **Read hotcaches.md** — `$VAULT/hotcaches.md` — load cached Q&A pairs
3. **Read index.md** — `$VAULT/index.md` — load the full page catalog
4. **Check today's diary** — `$VAULT/data/daily/YYYY-MM-DD.md` — create from template if missing
5. **Read log.md (last 20 entries)** — `$VAULT/log.md` — understand recent session history

After loading, confirm with a short status line:
```
Vault loaded. [N] pages in index. Today's diary: [exists/created]. Last session: [last log entry date].
```

## Available Workflows

Once loaded, the following workflows are active. Trigger them based on user intent:

| Trigger phrase | Workflow |
|---|---|
| "file this", "ingest this", paste with frontmatter block | **Ingest** — parse, create pages, update index + log + diary Ingested section |
| Question about knowledge in the vault | **Query** — check hot cache → search index → read pages → synthesize |
| "lint", "health check", "clean up wiki" | **Lint** — scan for orphans, gaps, contradictions, missing index entries |
| Any question (before answering) | **Hot Cache** — check hotcaches.md first; cache the answer if new |
| User asks to "add a task", "remind me", "track this" | **Task** — add `- [ ] text dd-mm-yyyy` to diary `## Tasks`, sync to Google Calendar |
| End of session or natural pause | **Diary Write** — Claude writes Session Summary, What We Learned, Observations to today's diary |

All workflow details (steps, formats, templates) live in `CLAUDE.md` — follow that spec exactly.

## Diary Writing — Claude's Responsibility

Claude must proactively write to the daily diary during sessions. Do not wait for the user to ask.

**Write to `## Session Summary`** when the session has covered significant ground — a brief bullet list of what was accomplished.

**Write to `## What We Learned`** when a new insight, discovery, or knowledge is found — something worth remembering next session.

**Write to `## Claude's Observations`** at session end or when you notice a pattern — suggestions for next time, things the user should know, open threads.

**Write to `## Ingested`** after every ingest — this is already part of the Ingest workflow.

**Write to `## Tasks`** when the user asks Claude to track something — `- [ ] task text dd-mm-yyyy`.

## Key Rules (from CLAUDE.md)

- **Daily notes are Claude's diary.** The user does not fill them in. Claude does.
- **index.md:** One line per page. Never list `raw/` or `daily/` files.
- **log.md:** Append-only. Never delete past entries.
- **Frontmatter:** All content pages need `title`, `date`, `domain`, `tags` keys.
- **WikiLinks:** Use `[[domain/filename|Display Title]]` syntax for internal links.
- **Hot cache:** Always check before answering. Cache new answers after answering.

## Quick Paths

```
Vault root:   $OBSIDIAN_VAULT/
Schema:       CLAUDE.md
Catalog:      index.md
Cache:        hotcaches.md
Log:          log.md
Content:      data/{personal,research,reading,work}/
Daily notes:  data/daily/YYYY-MM-DD.md
Raw input:    raw/   (unprocessed; deleted after ingest)
Templates:    data/templates/
```
