# BigQuery Agent Prompt

Prompt instructions and context for writing accurate, efficient BigQuery SQL
against Awell's data model. It helps humans and AI agents understand our schema,
tables, and relationships and turn natural-language questions into SQL.

This repo is packaged as an **[Agent Skill](https://code.claude.com/docs/en/skills)**
(`SKILL.md`), so you can install it in one command instead of copy-pasting a
prompt:

```bash
npx skills add https://github.com/awell-health/bigquery-agent-prompt
```

It works with Claude Code, the Claude apps, the Agent SDK, and other agents. The
full schema and query patterns live in [`PROMPT.md`](./PROMPT.md).

---

## 🎯 Purpose

To enable humans and AI agents to:

- Understand our BigQuery schema, tables, and relationships.
- Write efficient, correct, and well-structured SQL queries.
- Answer natural-language questions about our data.

---

## 🧠 Use it as a Skill (recommended)

Install it with [`skills`](https://github.com/vercel-labs/skills), the open
agent-skills CLI (supports Claude Code, Codex, Cursor, and many more):

```bash
# add to the current project → ./.claude/skills/
npx skills add https://github.com/awell-health/bigquery-agent-prompt

# or install for every project → ~/.claude/skills/
npx skills add https://github.com/awell-health/bigquery-agent-prompt -g
```

`skills` auto-detects your agent and installs `SKILL.md` along with the bundled
[`PROMPT.md`](./PROMPT.md) reference. Then just ask a data question — e.g.
*"How many active care flows does customer `ecma` have in production?"* — and
your agent loads the skill automatically based on its description.

<details>
<summary>Manual install (no CLI)</summary>

Clone this repo into a skills directory so the folder name matches the skill
name (`bigquery-agent-prompt`):

```bash
git clone https://github.com/awell-health/bigquery-agent-prompt \
  .claude/skills/bigquery-agent-prompt        # project scope
# or ~/.claude/skills/bigquery-agent-prompt   # user scope
```

For the Claude apps, Agent SDK, and other tools, see the
[Agent Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

</details>

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
