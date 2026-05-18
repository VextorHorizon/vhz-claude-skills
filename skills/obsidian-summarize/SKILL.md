---
name: obsidian-summarize
description: Use when the user asks to summarize a conversation, topic, or session for filing into their Obsidian second brain via Claude Code ingest workflow.
---

# Obsidian Second Brain — Summarize Skill

## Overview

Produce a structured summary block that Claude Code can ingest directly into the Obsidian wiki. Output must match the exact ingest format — no free-form prose, no deviation.

**Core principle:** One command → one paste-ready block → Claude Code files it with "file this".

## Step 1 — Scan the FULL Conversation (MANDATORY)

Before writing a single word of output, you MUST mentally traverse the entire conversation from the very first message to the most recent one.

**Do this every time, no exceptions:**
1. Start from message #1 (the oldest message in the thread)
2. Read forward chronologically through every exchange
3. Note: topics introduced, questions asked, answers given, decisions made, tools/concepts mentioned, URLs/specs shared, action items discussed
4. Only after completing this full scan → produce the summary block

**Why this matters:** Summaries that only capture the tail of a conversation miss context established early on. The Obsidian wiki entry must represent the entire session, not just what was said last.

**If the conversation is very long:** Chunk it mentally into phases (opening → middle → end), capture key points from each phase, then synthesize into the output block.

## When to Use

User says any of:
- "สรุปสิ่งที่คุยกัน" / "summarize our conversation"
- "เก็บบทสนทนานี้ไว้" / "file this"
- "สรุปให้หน่อย สำหรับ obsidian"
- Any request to save/summarize for their second brain

## Output Format — EXACT, NO DEVIATION

Always produce this block and nothing else outside it:

````
---
title: [ชื่อหัวข้อสั้นๆ เป็น English]
date: [YYYY-MM-DD วันนี้]
domain: [personal | research | reading | work]
---

Key takeaways:
- [สาระสำคัญข้อที่ 1 — Thai prose, English technical terms inline]
- [สาระสำคัญข้อที่ 2]
- [เพิ่มเท่าที่จำเป็น]

Entities/concepts mentioned:
- [ชื่อ concept, tool, framework, person, book — เฉพาะสิ่งที่ควรมีหน้าใน wiki]

Action items:
- [ ] [สิ่งที่ต้องทำ ถ้ามี]

Raw notes:
[ข้อมูลดิบสำคัญ, quotes, URLs, specs, ตัวเลข — ที่อาจหายไปถ้าไม่จด]
````

## Domain Classification

| Domain | เลือกเมื่อ |
|---|---|
| `personal` | เรื่องตัวเอง เป้าหมาย สุขภาพ จิตวิทยา ชีวิตส่วนตัว |
| `research` | หัวข้อวิชาการ เทคนิค deep dive ไอเดียใหม่ |
| `reading` | หนังสือ บทความ podcast สื่อที่บริโภค |
| `work` | งาน project meeting การตัดสินใจ career |

**ถ้าไม่แน่ใจ:** เลือก `research` สำหรับเทคนิค/ไอเดีย, `personal` สำหรับเรื่องส่วนตัว

## Language Rules

- **Key takeaways:** Thai prose, technical terms คงเป็น English inline
  - ✅ "ระบบนี้ใช้ multi-agent simulation เพื่อ predict future outcomes"
  - ❌ "ระบบนี้ใช้การจำลองหลายตัวแทน"
- **title:** English เสมอ (lowercase, hyphens)
- **Entities/concepts:** English หรือชื่อที่ใช้จริง
- **Raw notes:** ภาษาไหนก็ได้ ขอให้ครบ

## Entities — What to Include

✅ Include:
- Tools, frameworks, libraries ที่กล่าวถึง
- Concepts ที่ควรมีหน้าใน wiki (multi-agent, GraphRAG)
- ชื่อคน ชื่อหนังสือ ชื่อ project
- Methods หรือ patterns ที่น่าจำ

❌ Don't include:
- คำทั่วไป (AI, technology, software)
- สิ่งที่กล่าวถึงแค่ผ่านๆ ไม่มีนัยสำคัญ

## Raw Notes — What to Include

- URLs, links สำคัญ
- ตัวเลข specs ที่แน่นอน (versions, ports, prices)
- Code snippets หรือ config ที่จำเป็น
- Quotes สำคัญ
- ถ้าไม่มีอะไรพิเศษ: เขียน "(none)"

## Edge Cases

**บทสนทนาข้าม domain หลายอัน:**
เลือก domain หลักที่มีน้ำหนักมากสุด หรือแตกเป็นสองบล็อกแยกกัน แล้วระบุว่า "block 1/2, block 2/2"

**บทสนทนายาวมาก:**
เน้น takeaways ที่ actionable หรือ insight ใหม่ ตัด small talk ออก ถ้ามีหลายหัวข้อแยกชัดเจน → แตกเป็นหลายบล็อก

**ไม่มี action items:**
ใส่ `- [ ] (none)` ห้ามเว้นว่างหรือลบ section ออก

**ไม่รู้วันที่:**
ใช้วันที่จริงวันนี้เสมอ ห้ามเว้นว่าง

## Common Mistakes

| ❌ ผิด | ✅ ถูก |
|---|---|
| สรุปเป็น prose ยาวๆ | ใช้ format นี้เท่านั้น |
| ลืม frontmatter `---` | ต้องมี opening และ closing `---` |
| Entities ยาวเกินไป | เฉพาะ concepts ที่ควรมีหน้าใน wiki |
| Raw notes ว่างเปล่า | ใส่ "(none)" ถ้าไม่มี |
| title เป็น Thai | title ต้อง English เสมอ |
| domain ผิด | ดู Domain Classification table |

## Quick Example

บทสนทนาเรื่อง Docker setup สำหรับ web app:

````
---
title: web-app-docker-setup
date: 2026-01-15
domain: research
---

Key takeaways:
- App รัน Docker ได้ด้วย `docker compose up -d` หลังจาก config .env
- ต้องการ API key จาก provider ที่เลือก
- Frontend ที่ port 3000, Backend ที่ port 8000
- ตรวจสอบ logs ด้วย `docker compose logs -f`

Entities/concepts mentioned:
- Docker Compose
- environment variables
- multi-container setup

Action items:
- [ ] ทดสอบ health check endpoint หลัง deploy
- [ ] ตั้ง resource limits ใน compose file

Raw notes:
- ถ้า docker engine ไม่รัน: เปิด Docker Desktop ก่อน
- compose file ต้องมี depends_on สำหรับ service ordering
````
