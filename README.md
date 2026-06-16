# BigQuery Agent Prompt

Prompt instructions and context for writing accurate, efficient BigQuery SQL
against Awell's data model. It helps humans and AI agents understand our schema,
tables, and relationships and turn natural-language questions into SQL.

This repo is packaged as an **[Agent Skill](https://code.claude.com/docs/en/skills)**
(`SKILL.md`), so you can load it into Claude Code, the Claude apps, or the Agent
SDK instead of copy-pasting a prompt. The full schema and query patterns live in
[`PROMPT.md`](./PROMPT.md).

---

## 🎯 Purpose

To enable humans and AI agents to:

- Understand our BigQuery schema, tables, and relationships.
- Write efficient, correct, and well-structured SQL queries.
- Answer natural-language questions about our data.

---

## 🧠 Use it as a Skill (recommended)

**Claude Code** — clone this repo into a skills directory so the folder name
matches the skill name (`bigquery-agent-prompt`):

```bash
# project-level (available in one project)
git clone https://github.com/awell-health/bigquery-agent-prompt \
  .claude/skills/bigquery-agent-prompt

# or user-level (available everywhere)
git clone https://github.com/awell-health/bigquery-agent-prompt \
  ~/.claude/skills/bigquery-agent-prompt
```

Then just ask a data question — e.g. *"How many active care flows does customer
`ecma` have in production?"* Claude loads the skill automatically based on its
description.

**Claude apps / Agent SDK** — point your skills directory at this folder, or
upload it (it must contain `SKILL.md` at the root). See the
[Agent Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

---

## 📋 Use it as a plain prompt (legacy)

If your tool doesn't support skills, copy the contents of
[`PROMPT.md`](./PROMPT.md) into your AI assistant as **context**, then fire away.

---

## 📦 What's inside

- **`SKILL.md`** — the skill entry point: when to use it plus the always-on
  rules (project + customer, table reference format, realtime views, the
  `patient_data` tables).
- **`PROMPT.md`** — the canonical reference the skill reads: full terminology,
  table-by-table schema, and worked query patterns.

---

## 🔄 Keeping it accurate

The table schemas mirror the published
[BigQuery data schema](https://developers.awellhealth.com/awell-orchestration/docs/data/bigquery/data-schema).
When the data model changes, update `PROMPT.md` to match.
