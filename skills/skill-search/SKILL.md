---
name: skill-search
description: Use when the user asks "what skill should I use for X", "find me a skill for Y", "what's the best skill for Z", or invokes /skill-search with a query. Searches installed skills first, then skillsmp.com and GitHub if no strong match is found.
---

# Skill Search

## Overview

Find the best installed skill for a user's need. If installed skills don't cover it well enough, search skillsmp.com and GitHub for community skills, then compare options.

## How to Run This Skill

The user invokes this as `/skill-search <query>` or asks "what skill is best for X?".

Follow this exact 4-phase flow:

---

## Phase 1 — Search Installed Skills

Read the full list of available skills from the `<system-reminder>` block at session start (the list under "The following skills are available for use with the Skill tool:").

For each skill in the list, score it against the user's query:

| Score | Meaning |
|-------|---------|
| **Direct** | Skill is built exactly for this need |
| **Related** | Skill partially covers this need |
| **Weak** | Only tangentially relevant |

**Build a ranked list** — include all Direct and Related matches. Write each entry as:

```
### [Skill Name]
**Match:** Direct / Related
**What it does:** One sentence.
**Why it fits your need:** Specific reason tied to what the user asked.
**How to invoke:** `/skill-name` or "Skill tool: plugin:skill-name"
```

**Show all Direct matches first, then Related.**

If you find **2 or more Direct matches**, go to Phase 4 (skip Phases 2 and 3).

If you find **0–1 Direct matches**, continue to Phase 2.

---

## Phase 2 — Search skillsmp.com

Use `WebSearch` or `WebFetch` to search https://skillsmp.com/ for skills matching the user's query.

Search query format: `site:skillsmp.com <user's topic keywords>`

Or fetch the marketplace search page directly if the site supports it.

For each result found on skillsmp.com, format as:

```
### [Skill Name] *(from skillsmp.com — not installed)*
**What it does:** One sentence from the listing.
**Why it might help:** Specific reason.
**Install:** Link to the skill page on skillsmp.com
```

**Clearly label these as NOT YET INSTALLED.**

---

## Phase 3 — Search GitHub

If Phase 2 found fewer than 2 strong matches, search GitHub for Claude Code skill plugins matching the user's query.

**Search strategy — run these in order, stop when you have 3+ candidates:**

1. GitHub Code Search via WebFetch:
   ```
   https://github.com/search?q=<keywords>+SKILL.md+claude&type=code
   ```

2. GitHub Repository Search via WebFetch:
   ```
   https://github.com/search?q=<keywords>+claude+skill&type=repositories
   ```

3. WebSearch fallback:
   ```
   site:github.com SKILL.md claude "<keyword>"
   ```

**For each GitHub result, verify it's actually a Claude Code skill** by checking that the repo contains a `SKILL.md` file with frontmatter `name:` and `description:` fields.

Format each confirmed GitHub result as:

```
### [Skill Name] *(from GitHub — not installed)*
**Repo:** [owner/repo-name] — [GitHub URL]
**What it does:** Description from the SKILL.md frontmatter.
**Why it might help:** Specific reason tied to the user's query.
**Install hint:** `/plugin marketplace add <owner/repo-name>` then `/plugin install <skill-name>`
```

**If the repo has no SKILL.md or isn't a Claude Code plugin, skip it.**

---

## Phase 4 — Optional Comparison (use your judgment)

If the user's need is partially met by an installed skill but a skillsmp.com or GitHub skill would do it significantly better, add a **Comparison** section:

```
## Comparison: Installed vs Available

| | Installed Skill | External Skill (source) |
|-|-----------------|------------------------|
| **Best for** | ... | ... |
| **Gap** | ... | ... |
| **Recommendation** | Use installed if ... | Install if ... |
```

**Only include this section if the gap is meaningful.** Skip it if the installed skill is sufficient.

---

## Output Format

Structure your response as:

```
## Skills Found for: "[user's query]"

### Installed Skills
[ranked list — Direct first, then Related]

### From skillsmp.com  *(only if Phase 2 was run)*
[list of external skills]

### From GitHub  *(only if Phase 3 was run)*
[list of GitHub skills]

### Comparison  *(only if Phase 4 applies)*
[comparison table]

---
💡 **Recommendation:** [1–2 sentence summary of what to use and why]
```

---

## Rules

- **Always search installed skills first** — never go external if 2+ Direct matches exist
- **skillsmp.com before GitHub** — check skillsmp first, only go to GitHub if still no strong match
- **Be specific in the "Why it fits" field** — reference the user's exact words
- **Never invent skills** — only list what actually appears in the system-reminder, skillsmp.com, or GitHub results
- **Verify GitHub results** — must have a real SKILL.md with name/description frontmatter to qualify
- **Label all external skills clearly** — user must know what is installed vs what requires installation
- If the query is vague, ask one clarifying question before searching
