# BigQuery Agent Prompt

An [Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
that teaches an AI agent Awell's BigQuery data model. With it, an agent can turn a
question about care flow data into SQL, run it if it can reach BigQuery, and
explain the answer.

The skill is two files: [`SKILL.md`](./SKILL.md) tells the agent when to use it
and the rules that always apply, and [`PROMPT.md`](./PROMPT.md) describes every
table and column, with worked queries. It holds no data.

---

## Before you start

- **Access to your organization's Awell BigQuery dataset.** See
  [Get access to BigQuery](https://docs.awellhealth.com/docs/data/getting-access).
- **The project and dataset names.** The skill asks for them before it writes a
  query. If you don't have them, ask
  [support@awellhealth.com](mailto:support@awellhealth.com).

---

## Install it in Claude

### claude.ai and the Claude desktop app

1. Download
   [`bigquery-agent-prompt.zip`](https://github.com/awell-health/bigquery-agent-prompt/releases/latest/download/bigquery-agent-prompt.zip)
   from the latest release. Upload it as it is: the folder inside it must keep
   the skill's name.
2. Check that code execution is on, under **Settings > Capabilities**. Skills
   need it.
3. Open **Customize > Skills**, select **+**, then **Create skill** and
   **Upload a skill**, and choose the ZIP.
4. Turn the skill on in the list.

On a Team or Enterprise plan, an owner can provision the skill for every member
of the organization instead. See
[Use skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude).

### Claude Code

Clone the repository into a skills directory. The folder name must match the
skill's name:

```bash
# for every project
git clone https://github.com/awell-health/bigquery-agent-prompt ~/.claude/skills/bigquery-agent-prompt

# or for one project, from its root
git clone https://github.com/awell-health/bigquery-agent-prompt .claude/skills/bigquery-agent-prompt
```

`git pull` in that folder picks up later changes. If you already uploaded the
skill on claude.ai, Claude Code signed in with the same Claude account (not an
API key) loads it from there.

---

## Use it with another agent

If the agent supports Agent Skills (Codex and Cursor, for instance), install it
with [`skills`](https://github.com/vercel-labs/skills), an open-source CLI that
detects the agent and puts the files where it expects them:

```bash
npx skills add https://github.com/awell-health/bigquery-agent-prompt
```

If it doesn't, give it the contents of [`PROMPT.md`](./PROMPT.md) as context,
for example as its system prompt or a project instruction file, then ask your
question.

---

## Running the queries

The skill writes SQL. To run it, the agent needs its own way into BigQuery:

- **In Claude Code or another agent with a terminal:** the `bq` command-line
  tool from the [Google Cloud CLI](https://cloud.google.com/sdk/docs/install),
  signed in with the Google account that has access.
- **Through a BigQuery connector** (an MCP server), if your organization uses
  one.
- **Without either:** copy the query into BigQuery Studio and give the agent the
  result.

---

## Try it

Name the project and dataset in your first question, so the skill doesn't have
to ask:

- *"In awell-production, dataset `example_clinic`: how many care flows started
  last month, per care flow definition?"*
- *"How many patients completed the post-operative follow-up care flow in the
  last 90 days, and how long did it take them, on average?"*
- *"Which forms do patients leave open the longest before completing them?"*
- *"What's the latest recorded weight for each patient in the weight
  management care flow?"*

---

## Your data

- The skill is instructions and a description of the tables. It contains no
  data and sends nothing to Awell.
- The agent's queries run with the access you give it. Read access is enough:
  use a Google account or service account that can only read.
- What the agent reads from a query result goes to its model provider, under
  your organization's agreement with that provider. Check that the agreement
  covers health data before you ask for patient-level rows.

---

## Keeping it accurate

[`PROMPT.md`](./PROMPT.md) is the column-level reference. The
[Data schema](https://docs.awellhealth.com/docs/data/data-schema) page explains
the model behind it. When the data model changes, update `PROMPT.md`. Every push
to `main` republishes `bigquery-agent-prompt.zip` as the latest release.
