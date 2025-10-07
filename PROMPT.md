# Writing queries

When writing queries, always ask the user for two key pieces of information:

1. **Project** — determines which BigQuery environment you’re querying:
  - `awell-sandbox`
  - `awell-production`
  - `awell-production-us`
  - `awell-production-uk`
2. **Customer (dataset)** — every customer in Awell has a dedicated dataset within the project.

## Query structure

To query a table, combine both values following this structure: `{project}.{customer}.{table}`

For example:
```sql
SELECT *
FROM `awell-production.ecma.data_point_definitions`
```

This selects all records from the data_point_definitions table in the ecma dataset (customer) within the awell-production project.

## Realtime views

We also maintain realtime tables, which contain near-real-time data. These should only be used when specifically requested, as they are more resource-intensive and may have slightly different consistency guarantees.

To query the realtime version of a table, simply append _realtime to the table name.

For example:
```sql
SELECT *
FROM `awell-production.zna.data_points_realtime`
```

Pattern: `{project}.{customer}.{table}_realtime`

# Tables

## Activities

table: activities

| Field name              | Type      | Mode      | Description |
|---------------------------|-----------|------------|--------------|
| id                        | STRING    | NULLABLE   | Unique identifier. |
| care_flow_id              | STRING    | NULLABLE   | Identifier of the care flow associated with the activity. Refers to `id` in `care_flow` table. |
| care_flow_definition_id   | STRING    | NULLABLE   | Identifier of the care flow definition (template) from which the care flow was instantiated. |
| status                    | STRING    | NULLABLE   | The current (last) activity status. One of: `active`, `done`, `failed`, `canceled`, `expired`. Status `done` indicates complete resolution of the activity, such as a sent message being read or a form being fully completed. Done refers to completed activity. |
| resolution                | STRING    | NULLABLE   | An internal system status reflecting the outcome of executing the activity, indicating `success`, `failure` (e.g., if a plugin call fails), or `NULL` for activities yet to be resolved or not applicable. |
| date                      | TIMESTAMP | NULLABLE   | Date of the activity (UTC). |
| scheduled_date            | TIMESTAMP | NULLABLE   | Scheduled start time (UTC). Relevant only for scheduled activities. |
| completion_date           | TIMESTAMP | NULLABLE   | Completion date of completed (`done`) activities. |
| sub_activities            | JSON      | NULLABLE   | Array of sub-activities. Each sub-activity is an object various properties, including `action`, `id`, `name`, `type`, etc., though the exact properties for a given sub-activitymay vary. |
| metadata                  | JSON      | NULLABLE   | Metadata of the activity (contextual data). |
| action                    | STRING    | NULLABLE   | Describes the action performed on the object that this activity models. This never changes during the activity lifecycle. One of: `added`, `activate`, `assigned`, `scheduled`, `postponed`, `send`, `complete`, `delegated`, `generated`, `stopped`, `discarded` |
| orchestrated_instance_id  | STRING    | NULLABLE   | Unique identifier of the orchestrated instance (could be an action, step, or track). Can be used to merge with `actions`, `steps`, and `tracks` tables using the `id` field.|
| orchestrated_track_id     | STRING    | NULLABLE   | Unique identifier of the orchestrated track associated with the activity. Present only for objects within a track (steps, actions, etc.). |
| orchestrated_step_id      | STRING    | NULLABLE   | Unique identifier of the orchestrated step associated with the activity. Present only for objects within a step (actions, etc.). |
| action_definition_id      | STRING    | NULLABLE   | Identifier of the action definition from which this action was instantiated. |
| action_component_name     | STRING    | NULLABLE   | Component holding the primary object (e.g., form, message, calculation). |
| object_type               | STRING    | NULLABLE   | Type of primary object this activity relates to. Example values: action, api_call, calculation, form, message, pathway, plugin_action, reminder, step, track. |
| object_name               | STRING    | NULLABLE   | Name of the primary object associated with the activity. |
| object_id                 | STRING    | NULLABLE   | ID of the primary object. |
| indirect_object_type      | STRING    | NULLABLE   | Type of secondary object (e.g., `patient`, `stakeholder`, `plugin`). |
| indirect_object_name      | STRING    | NULLABLE   | Name of the related secondary object. |
| step_name                 | STRING    | NULLABLE   | Name of the step this activity belongs to. |
| track_name                | STRING    | NULLABLE   | Name of the track this activity belongs to. |
| track_id                  | STRING    | NULLABLE   | Identifier of the track this activity belongs to. |
| last_synced_at            | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of importing data to BigQuery. |

## Care flows

table: care_flows

| Field name          | Type      | Mode      | Description |
|----------------------|-----------|------------|--------------|
| id                   | STRING    | NULLABLE   | Unique identifier. |
| patient_id           | STRING    | NULLABLE   | Identifier of the patient enrolled in the care flow. Refers to the `id` column in the `patient` table. |
| definition_id        | STRING    | NULLABLE   | Identifier of the care flow definition (designed care flow template) from which this care flow was instantiated. |
| title                | STRING    | NULLABLE   | Title (name) of the care flow definition. |
| release_id           | STRING    | NULLABLE   | An internal identifier for the published version_number of the care flow definition. Refers to the `release_id` in the `published_careflows` table, serving as a foreign key, together with `definition_id` for connecting to the proper care flow definition. |
| status               | STRING    | NULLABLE   | Current care flow status. Possible values: `active`, `stopped`, `completed`, etc. |
| status_explanation   | STRING    | NULLABLE   | Explanation of the current care flow status, often a human-readable justification or reason. |
| start_date           | TIMESTAMP | NULLABLE   | Recorded start date of the care flow (UTC). |
| stop_date            | TIMESTAMP | NULLABLE   | Recorded stop date of the care flow (UTC). Only populated for stopped flows. |
| complete_date        | TIMESTAMP | NULLABLE   | Recorded completion date of the care flow (UTC). Populated only for completed flows. |
| created_by_user_name | STRING    | NULLABLE   | Name of the user who created the care flow instance. |
| created_by_user_email| STRING    | NULLABLE   | Email of the user who created the care flow instance. |
| created_by_user_id   | STRING    | NULLABLE   | Identifier of the user who created the care flow instance. |
| last_synced_at       | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of importing data to BigQuery. |

## Data point definitions

table: data_point_definitions

| Field name          | Type      | Mode      | Description |
|----------------------|-----------|------------|--------------|
| id                   | STRING    | NULLABLE   | Unique identifier. Not to be used as foreign key when joining with other tables. |
| definition_id        | STRING    | NULLABLE   | Version-agnostic identifier of designed data point. |
| release_id           | STRING    | NULLABLE   | An internal identifier for the published version number of the care flow definition. Refers to the `release_id` in the `published_careflows` table. |
| source_definition_id | STRING    | NULLABLE   | Identifier of the originating data point definition, if any. |
| category             | STRING    | NULLABLE   | Identifies how/where the data point is collected (e.g., `pathway`, `form`, `calculation`, `step`). |
| key                  | STRING    | NULLABLE   | Human-readable qualified key defining the meaning of the collected data. |
| options              | RECORD    | REPEATED   | Nested field with an array of objects, each representing a valid option with value and label. Example: "value": "1", "label": "Yes", "value": "0", "label": "No" . |
| value_type           | STRING    | NULLABLE   | The expected primitive type for the collected data (boolean, date, number, string, numbers_array). |
| last_synced_at       | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of importing data to BigQuery. |

Category `pathway` represent data points that are baseline info / baseline data points.

## Data points

table: data_points

| Field name              | Type      | Mode      | Description |
|---------------------------|-----------|------------|--------------|
| id                        | STRING    | NULLABLE   | Unique identifier. |
| definition_id             | STRING    | NULLABLE   | Version agnostic identifier linking to the `definition_id` column in the `data_point_definitions` table, acting as a foreign key to this table, together with `release_id`. |
| release_id                | STRING    | NULLABLE   | An internal identifier for the published version_number of the care flow definition. Refers to the `release_id` in the `published_careflows` table, serving as a foreign key, together with `definition_id` for connecting to the proper care flow definition. |
| care_flow_id              | STRING    | NULLABLE   | Identifier of the care flow in which the data point was collected. Refers to the `id` column in the `care_flows` table, serving as a foreign key to the `care_flows` table. |
| care_flow_definition_id   | STRING    | NULLABLE   | Identifier of the care flow definition (designed care flow template) from which the care flow was instantiated. Refers to the `definition_id` in the `published_careflows` table, serving as a foreign key, together with `release_id` for connecting to the proper care flow definition. |
| activity_id               | STRING    | NULLABLE   | Identifier of the activity in which the data point was collected. Refers to the `id` column in the `activities` table, acting as a foreign key to the `activities` table. |
| value_raw                 | STRING    | NULLABLE   | Serialized value of the data point. |
| value_boolean             | BOOLEAN   | NULLABLE   | Boolean value of the data point (populated when the data point type is boolean). |
| value_numeric             | NUMERIC   | NULLABLE   | Numeric value of the data point (populated when the data point type is numeric). |
| value_date                | TIMESTAMP | NULLABLE   | Date/time value of the data point (populated when the data point type is date or timestamp). |
| label                     | STRING    | NULLABLE   | Descriptive label associated with the value, providing a human-readable description. Example: for value_numeric 0, the label might be "Female" or "Ocassionally". Especially useful for data points collected in a form. |
| value_type                | STRING    | NULLABLE   | Primitive type of the value before serialization (`boolean`, `number`, `string`, etc.). |
| date                      | TIMESTAMP | NULLABLE   | Timestamp when the data point was collected (UTC). |
| last_synced_at            | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of importing data to BigQuery. |
| status                    | STRING    | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Usually `created`; not relevant for analytical queries. |

## Event logs

table: event_logs

| Field name             | Type      | Mode      | Description |
|--------------------------|-----------|------------|--------------|
| event_id                | STRING    | NULLABLE   | Unique identifier for the event (UUID4). |
| timestamp               | TIMESTAMP | NULLABLE   | When the event occurred (UTC). |
| event_type              | STRING    | NULLABLE   | Type of event. Default is `generic`; can include specific event categories. |
| severity                | STRING    | NULLABLE   | Event severity level. One of: `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`. |
| operation               | STRING    | NULLABLE   | The operation or action that generated this event. |
| message                 | STRING    | NULLABLE   | Human-readable description of the event. |
| source_system           | STRING    | NULLABLE   | System component that generated the event. |
| correlation_id          | STRING    | NULLABLE   | Most specific ID available for correlation—typically `activity_id` for activities. |
| care_flow_definition_id | STRING    | NULLABLE   | Identifier of the care flow definition associated with the event. |
| care_flow_id            | STRING    | NULLABLE   | Identifier of the care flow associated with the event. |
| activity_id             | STRING    | NULLABLE   | Identifier of the activity associated with the event. |
| data                    | JSON      | NULLABLE   | Optional data (in JSON form) associated with the event. |
| error                   | JSON      | NULLABLE   | Error details when applicable (structured object containing code, message, etc.). |
| last_synced_at          | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of importing data to BigQuery. |

## Forms

table: forms

| Field name     | Type      | Mode      | Description |
|-----------------|-----------|------------|--------------|
| id              | STRING    | NULLABLE   | Unique identifier. Relates to `object_id` in the `activities` table. |
| definition_id   | STRING    | NULLABLE   | Version-agnostic identifier of the form. |
| release_id      | STRING    | NULLABLE   | An internal identifier for the published version_number of the care flow definition. Refers to the `release_id` in the `published_careflows` table. |
| key             | STRING    | NULLABLE   | Human readable qualified key. Usually camelCase representation of the form title. Used to form data_points key for collected answers. |
| title           | STRING    | NULLABLE   | Title (name) of the form. |
| metadata        | STRING    | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Optional metadata for the form. Typically contains structure, layout, or styling details. |
| last_synced_at  | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of importing data to BigQuery. |

## Patients

table: patients

| Field name    | Type      | Mode      | Description |
|----------------|-----------|------------|--------------|
| id             | STRING    | NULLABLE   | Unique identifier. |
| profile_id     | STRING    | NULLABLE   | Unique identifier of the associated patient profile. Acts as foreign key for joins. |
| status         | STRING    | NULLABLE   | Indicates patient status within the system. Currently, `active_record` is the only available value, indicating that patient is present/not deleted. |
| last_synced_at | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of last sync. |

## Patient profiles

table: patient_profiles

| Field name               | Type      | Mode      | Description |
|---------------------------|-----------|------------|--------------|
| id                        | STRING    | NULLABLE   | Unique identifier. |
| name                      | STRING    | NULLABLE   | Concatenation of the first name and last name. |
| first_name                | STRING    | NULLABLE   | First name of the patient. |
| last_name                 | STRING    | NULLABLE   | Last name of the patient. |
| email                     | STRING    | NULLABLE   | Email address of the patient. |
| birth_date                | DATE      | NULLABLE   | Birth date of the patient. |
| sex                       | STRING    | NULLABLE   | Sex of the patient in ISO IEC 5218 format. One of `0` (Not known), `1` (Male), `2` (Female), or `9` (Not applicable). |
| preferred_language        | STRING    | NULLABLE   | Preferred language of the patient in ISO 639-1 (2-letter) format. |
| national_registry_number  | STRING    | NULLABLE   | National registry number of the patient. |
| patient_code              | STRING    | NULLABLE   | Arbitrary identifier associated to the patient, used to facilitate external references. |
| phone                     | STRING    | NULLABLE   | Phone number in the E.164 format. |
| mobile_phone              | STRING    | NULLABLE   | Mobile phone number in the E.164 format. |
| address_street            | STRING    | NULLABLE   | Street address of the patient. |
| address_city              | STRING    | NULLABLE   | City of the patient's address. |
| address_zip               | STRING    | NULLABLE   | ZIP or postal code of the patient's address. |
| address_state             | STRING    | NULLABLE   | State or region of the patient's address. |
| address_country           | STRING    | NULLABLE   | Country of the patient's address (ISO 3166-1 alpha-2 format recommended). |
| last_synced_at            | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of last sync. |
| status                    | STRING    | NULLABLE   | Current status of the patient profile (e.g., `active`, `archived`). |
| identifiers               | JSON      | REPEATED   | Array of patient identifiers associated to the patient. You can use this to facilitate the reconciliation of patient records between Awell and your domain. Each identifier is an object with `system` and `value` fields. |

## Published care flows

table: published_careflows

| Field name       | Type      | Mode      | Description |
|-------------------|-----------|------------|--------------|
| id                | STRING    | NULLABLE   | Unique identifier. Not to be used as foreign key when joining. |
| definition_id     | STRING    | NULLABLE   | Identifier of the care flow definition (designed care flow). |
| title             | STRING    | NULLABLE   | Title (name) of the care flow definition. |
| release_id        | STRING    | NULLABLE   | Internal identifier for the published version number. |
| version_number    | STRING    | NULLABLE   | Version number of the care flow definition, showing evolution over time. |
| last_synced_at    | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of last sync. |
| publish_time      | TIMESTAMP | NULLABLE   | Publish (release) time of the care flow definition release. |
| source_rank       | INTEGER   | NULLABLE   | Ranking of the source (if applicable). |

## Questions

table: questions

| Field name         | Type      | Mode      | Description |
|---------------------|-----------|------------|--------------|
| id                  | STRING    | NULLABLE   | Unique identifier. |
| definition_id       | STRING    | NULLABLE   | Identifier of the care flow definition (designed care flow template). Refers to the `definition_id` in the `care_flows` table, serving as a foreign key, together with `release_id` for connecting orchestrated care flow with a proper care flow definition. |
| form_definition_id  | STRING    | NULLABLE   | Form identifier. Refers to the `definition_id` in the form table. |
| release_id          | STRING    | NULLABLE   | An internal identifier for the published version_number of the care flow definition. Refers to the `release_id` in the `care_flows` table, serving as a foreign key, together with `definition_id` for connecting orchestrated care flow with a proper care flow definitio (version). |
| key                 | STRING    | NULLABLE   | Human-readable qualified key. Used to form data mappings. |
| title               | STRING    | NULLABLE   | Title (name) of the question. |
| metadata            | STRING    | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Optional metadata about the question. |
| question_type       | STRING    | NULLABLE   | Type of the question (e.g. `short_text`, `long_text`, `numeric`, etc.). |
| last_synced_at      | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of last sync. |

## Steps

table: steps

| Field name              | Type      | Mode      | Description |
|--------------------------|-----------|------------|--------------|
| id                       | STRING    | NULLABLE   | Unique identifier of the step. |
| name                     | STRING    | NULLABLE   | Name of the step. |
| definition_id            | STRING    | NULLABLE   | Identifier of the step definition (template) from which the step was instantiated. |
| care_flow_definition_id  | STRING    | NULLABLE   | Identifier of the care flow definition associated with the step. Refers to the `definition_id` in the `care_flows` table. |
| care_flow_id             | STRING    | NULLABLE   | Identifier of the care flow in which the step exists. Refers to the `id` column in the `care_flows` table, serving as a foreign key. |
| track_id                 | STRING    | NULLABLE   | Identifier of the track the step belongs to. Refers to the `id` column in the `tracks` table. |
| started_at               | TIMESTAMP | NULLABLE   | Timestamp indicating when the step was started. |
| completed_at             | TIMESTAMP | NULLABLE   | Timestamp indicating when the step was completed. |
| scheduled_at             | TIMESTAMP | NULLABLE   | The date and time when the step is scheduled to start. |
| duration_in_seconds      | FLOAT     | NULLABLE   | Duration of the step in seconds, calculated as `completed_at - started_at`. |
| status                   | STRING    | NULLABLE   | Current status of the step. Possible values: `active`, `completed`, `stopped`, `deleted`, or other statuses derived from actions. |
| last_synced_at           | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Recorded timestamp of last sync. |

## Tracks

table: tracks

| Field name              | Type      | Mode        | Description |
|--------------------------|-----------|------------|--------------|
| id                       | STRING    | NULLABLE   | Unique identifier of the track. |
| name                     | STRING    | NULLABLE   | Name of the track. |
| definition_id            | STRING    | NULLABLE   | Identifier of the track definition (template) from which the track was instantiated. |
| care_flow_definition_id  | STRING    | NULLABLE   | Identifier of the care flow definition associated with the track. Refers to the `definition_id` in the `care_flows` table. |
| care_flow_id             | STRING    | NULLABLE   | Identifier of the care flow in which the track exists. Refers to the `id` column in the `care_flows` table, serving as a foreign key. |
| started_at               | TIMESTAMP | NULLABLE   | Timestamp indicating when the track started. |
| completed_at             | TIMESTAMP | NULLABLE   | Timestamp indicating when the track was completed. |
| scheduled_at             | TIMESTAMP | NULLABLE   | The date and time when the track was scheduled to start. |
| duration_in_seconds      | FLOAT     | NULLABLE   | Duration of the track in seconds. |
| status                   | STRING    | NULLABLE   | Current status of the track. Possible values: `active`, `completed`, `stopped`, `deleted`, or other statuses derived from actions. |
| last_synced_at           | TIMESTAMP | NULLABLE   | [IRRELEVANT FOR ANALYSIS] Last time the record was synced. |

# Common patterns

## Fetching a data point value

Data points can be collected multiple times so usually you want to grab the last / most recent value.

```sql
WITH
  data_points AS (
    SELECT
      definition.key AS KEY,
      definition.category as category,
      data_point.date AS date,
      data_point.care_flow_id AS care_flow_id,
      data_point.value_raw AS value_raw,
      data_point.value_numeric AS value_numeric
    FROM
      `awell-sandbox.foobar_care_realtime.data_points` AS data_point
    LEFT JOIN
      `awell-sandbox.foobar_care_realtime.data_point_definitions` AS definition
    ON
      data_point.definition_id = definition.definition_id
      AND data_point.release_id = definition.release_id
  )
SELECT
  data_points.care_flow_id AS care_flow_id,
  MAX_BY(data_points.value_raw, data_points.date) AS latest_value_raw
FROM
  data_points
WHERE
  KEY = 'diagnosis'
GROUP BY care_flow_id
```

Often, you'd like to join this with the care flow table:

```
WITH
  data_points AS (
    SELECT
      definition.key AS KEY,
      definition.category as category,
      data_point.date AS date,
      data_point.care_flow_id AS care_flow_id,
      data_point.value_raw AS value_raw,
      data_point.value_numeric AS value_numeric
    FROM
      `awell-sandbox.foobar.data_points` AS data_point
    LEFT JOIN
      `awell-sandbox.foobar.data_point_definitions` AS definition
    ON
      data_point.definition_id = definition.definition_id
      AND data_point.release_id = definition.release_id
  ),
  diagnosis AS (
    SELECT
      data_points.care_flow_id AS care_flow_id,
      MAX_BY(data_points.value_raw, data_points.date) AS latest_value_raw
    FROM
      data_points
    WHERE
      KEY = 'diagnosis'
    GROUP BY care_flow_id
  )
SELECT
  care_flow.id AS careflow_id,
  diagnosis.latest_value_raw AS diagnosis,
FROM
  `awell-sandbox.foobar.care_flows` AS care_flow
LEFT JOIN
  diagnosis
ON
  care_flow.id = diagnosis.care_flow_id
```

