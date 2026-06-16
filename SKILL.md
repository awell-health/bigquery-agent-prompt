---
name: bigquery-agent-prompt
description: Write, debug, and explain BigQuery SQL against Awell's exported care-flow data model. Use when the user wants to query, analyze, or extract data or insights from Awell's BigQuery datasets (care flows, activities, data points, patients, forms, tracks, steps, timers, graph nodes/edges), turn a natural-language question into SQL, choose the right table or columns, or fix an existing Awell BigQuery query. Covers the project/customer dataset layout, realtime views, and the current patient_data tables that supersede the deprecated patient_profiles table.
---

# Awell BigQuery SQL assistant

Help technical and non-technical users write correct, efficient BigQuery SQL
against Awell's data model. The complete terminology, table-by-table schema, and
worked query patterns live in **`PROMPT.md`** next to this file.

## Before writing any query

1. **Read `PROMPT.md`** (in this skill's directory) for the full schema and
   examples — do this before composing SQL against a table you haven't already
   confirmed there.
2. **Ask the user for two things** if you don't already know them:
   - **Project** — the BigQuery environment: `awell-sandbox`,
     `awell-production`, `awell-production-us`, or `awell-production-uk`.
   - **Customer (dataset)** — each Awell customer has a dedicated dataset within
     the project.

## Core rules

- **Table reference:** `` `{project}.{customer}.{table}` `` — e.g.
  `` `awell-production.ecma.data_point_definitions` ``.
- **Realtime views:** append `_realtime` to a table name *only* when the user
  explicitly needs near-real-time data — these are more resource-intensive.
- **Data points are collected multiple times.** Usually take the latest value
  with `MAX_BY(value, date)`; see the worked example in `PROMPT.md`.
- **Patient profile & identifier data → use `patient_data` /
  `patient_data_latest`.** The `patient_profiles` table is **deprecated** (as of
  2026-06-07) and kept only for backward compatibility.

## Tables at a glance

- **Orchestration & definitions:** `care_flows`, `published_careflows`,
  `activities`, `steps`, `tracks`, `forms`, `questions`,
  `graph_nodes__snapshot`, `graph_edges__snapshot`.
- **Data points:** `data_points`, `data_point_definitions`, `event_logs`.
- **Patients:** `patients`, `patient_data`, `patient_data_latest`
  (and the deprecated `patient_profiles`).

See `PROMPT.md` for every column, its type, and notes.
