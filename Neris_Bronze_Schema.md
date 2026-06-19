# NERIS Bronze Schema (`neris`)

Reference for the Bronze Delta tables produced by **`NB_DIF_LZ_to_BRZ_NERIS`**, which flattens NERIS landing-zone JSON payloads (`LH_DATA_LANDINGZONE`) into the `neris` schema in `LH_BRONZE_LAYER`.

This document maps every Bronze **table** and **column** back to the **source endpoint** and the **original field** (JSON path) in the raw API payload. Source: derived from `plan-NB_DIF_LZ_to_BRZ_NERIS.prompt.md` and the notebook's extractor functions.

> **Schema location:** schema-enabled lakehouse → `Tables/neris/<table>`; non-schema lakehouse → `Tables/neris_<table>`. SQL: `neris.<table>` (or `neris_<table>`).

---

## Conventions

- **All Bronze columns are stored as `string`.** This prevents schema drift when the API changes a field's type or adds enum values; typing happens downstream in Silver/Gold.
- **`*_json` columns** preserve a nested object/array as a canonical JSON string. As a rule of thumb, 2nd-level structures are exploded into their own table and deeper (3rd-level+) structures are kept as a JSON blob — but the deciding factor is the *shape* of the data, not depth alone. See [How nested data is handled](#how-nested-data-is-handled).
- **Natural key** columns (used for the Delta MERGE upsert) are marked 🔑.
- **Derived** columns are not copied from a single source field — they are computed by the pipeline (from the filename, file path, the array a value came from, or a fallback). Marked in the *Original Field* column.
- **Renamed** columns differ in name from the source field (e.g. SQL reserved words); the original name is shown in *Original Field*.
- The record **root** noted per table is relative to one payload record (one element of the endpoint's root array, or the single object).

### Metadata / lineage columns (present on **every** table)

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| (all tables) | `_source_file_path` | (all) | *derived* | Full ABFSS path of the source JSON file |
| (all tables) | `_source_entity_id` | (all) | *derived* | Entity id from the file path, or the resolved payload id when the path has none (entity pages) |
| (all tables) | `_ingest_timestamp` | (all) | *derived* | UTC processing time |
| (all tables) | `_ingest_run_id` | (all) | *derived* | Run identifier for lineage (`neris_brz_<ts>` if not supplied) |
| (all tables) | `_payload_hash` | (all) | *derived* | SHA-256 of the non-metadata columns; drives idempotent MERGE |

---

## How nested data is handled

NERIS payloads nest several levels deep, and the same word — "array" — covers very different shapes. Bronze does **not** treat them all the same; how a nested value lands depends on its **shape**, not just how deep it sits. The deciding question:

> **Does each item carry its own bundle of fields we'd want to query separately?**

That gives four cases:

| Source shape | Example (this schema) | How Bronze lands it | Note in tables |
|---|---|---|---|
| **Scalar field** — a single value at any depth | `base.location.county` | Pulled up into a named **column** by its dotted path (one value → one column) | *(plain column)* |
| **Array/object of multiple fields** — "a list of records" | `incident_types[]`, `casualty_rescues[]`, `stations[]` | **Exploded into its own table**, one row per element, with the parent key repeated so each row stands on its own | own table, `Root: …[]` |
| **List of plain strings** — "a list of labels" | `base.displacement_causes[]`, `actions_tactics.action_noaction.actions[]` | Kept as a **simple list**: a slim one-row-per-value table whose single column *is* the string. No other columns are repeated. | **array element value** |
| **Deep multi-field structure** we haven't broken out yet | `feature_flags`, entity `location` geometry, `fire_detail.investigation_types`, `casualty_rescues[].rescue.impediments` | Preserved **whole as a JSON string** (`*_json`) so nothing is lost; broken out later in Silver/Gold | **JSON blob** |

**Two shapes that look alike but aren't.** A *list of strings* and a *JSON blob* both read as "an array," but they are opposites:

- A **list of strings** has no internal structure — it is safe to explode to rows, or to concatenate into one field (e.g. `SMOKE | WATER`). There is nothing to "lose."
- A **JSON blob** is all structure — multiple fields, sometimes nested objects or arrays-of-objects. It is kept intact precisely so that structure isn't flattened away prematurely.

**Shape wins over depth.** The 2nd-level / 3rd-level rule of thumb is only a guide. `actions_tactics.action_noaction.actions[]` sits at the 3rd level yet is exploded to its own table (`incident_action_tactic`) because it is a plain string list; `casualty_rescues[].rescue.impediments` is *also* a string list but is kept as a JSON blob because it lives inside an already-exploded record and isn't broken out further at Bronze. Which structures earn their own table is a curated decision driven by query value and this shape test — not an automatic depth cutoff.

**Bronze lands; Silver reshapes.** Bronze's job is a faithful, lossless copy: scalars as strings, string-lists as slim tables, deep structures as JSON. Reshaping for reporting — splitting a coded string like `MEDICAL||ILLNESS||BREATHING_PROBLEMS` into three columns, concatenating a string list into one field, or exploding a `*_json` blob into its own columns — happens **downstream in Silver/Gold**, where it can change without risk of dropping data if the API evolves.

---

## Notes column legend

The **Notes** column on each table flags *how* a Bronze column relates to its source — i.e. whether it is a straight copy, computed, renamed, coalesced, or a preserved blob. Use this key:

| Term in Notes | What it tells the reader |
|---|---|
| **resolved** · *resolved (path fallback)* · *falls back to path-derived id* | The value can come from more than one place, so the pipeline looks it up rather than copying a single fixed field. It reads the preferred field first; if that field is missing or empty, it uses a backup source so the column is still filled. Main example — `entity_neris_id`: it normally comes from **inside the payload** (`base.department_neris_id` for incident records, the top-level `neris_id` for entity records), but when the payload doesn't include it, the pipeline falls back to the **agency ID taken from the landing-zone folder/file path**. |
| *derived* (shown in the **Original Field** column) · *from the landing-zone path segment* · *…from `incident_history_…` filename* | The pipeline builds this value itself instead of copying it from any field in the JSON. It comes from the file's name or its folder path. Example: an `incident_history` file does not contain its incident ID, so the pipeline reads it from the file name. |
| **renamed** · *renamed from `x`* · *renamed (URL scalar)* · *renamed (`current` is a SQL reserved word)* | Same value as the source, but the Bronze column was given a different name — usually to be clearer or to avoid a reserved word. The original field name is shown in the *Original Field* column. Examples: `is_primary` comes from `primary`; `is_current` from `current` (a reserved SQL word); `location_url` from `location`. |
| **JSON blob** · *full object as JSON blob* · *JSON blob of keys beyond …* · *JSON blob (entity-level geometry)* | Instead of being split into more columns, a nested object or list is kept whole and stored as a single piece of JSON text — so nothing is lost and it can be expanded later if needed. These columns end in `_json`. |
| **array element value** | The source is a plain list of values (not a list of objects), so each item in the list becomes its own row and the column holds that item directly — there is no field name behind it. Example: every value in `base.displacement_causes` becomes one `cause` row. |
| **from entity root** · **carried from parent station** · **carried from parent region set** | This value actually belongs to a parent record (the entity, a station, or a region set). It is copied onto each child row so the row makes sense on its own and you don't have to join back to the parent for everyday queries. |
| **first non-empty** | The pipeline checks the listed fields in order and uses the first one that actually has a value. Example: `point_url` uses `base.location.point` if it is present, otherwise it uses `base.point`. |
| `FF` / `NONFF` · `STRUCTURE` / `OUTSIDE` · `fire` / `ems` / `investigation` | These are sample values the column can hold, shown to give you a feel for the data. They are examples, not the full list of possibilities. |

> The 🔑 marker (on a **column name**, not in the Notes column) flags a **natural key** — the column(s) used to match rows in the Delta MERGE upsert.

---

## Table inventory (30 tables)

| Endpoint | Landing folder | Tables | Count |
|---|---|---|---|
| `incident` | `incident` | `incident_core`, `incident_location`, `incident_displacement_cause`, `incident_type`, `incident_special_modifier`, `incident_aid`, `incident_nonfd_aid`, `incident_action_tactic`, `incident_dispatch`, `incident_dispatch_unit_response`, `incident_unit_response`, `incident_casualty_rescue`, `incident_medical_detail`, `incident_exposure`, `incident_fire_detail`, `incident_fire_detail_electric_hazard`, `incident_hazsit_detail`, `incident_hazsit_chemical`, `incident_hazsit_medical_detail`, `incident_weather`, `incident_census_tract` | 21 |
| `entity` | `entity` | `entity_core`, `entity_station`, `entity_station_unit`, `entity_region_set`, `entity_region`, `entity_aid_agreement`, `entity_service` | 7 |
| `incident_history` | `incident_history` | `incident_history_event` | 1 |
| `accountenrollment` *(dormant)* | `accountenrollment` | `accountenrollment_core` | 1 |

---

# Endpoint: `incident`

Record root = each element of the `incidents[]` array. `entity_neris_id` is resolved from `base.department_neris_id` (falls back to the path-derived id) on every incident table.

## `neris.incident_core`
**Grain:** 1 row per incident · **Root:** incident object

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_core` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_core` | `entity_neris_id` | incident | `base.department_neris_id` | resolved (path fallback) |
| `neris.incident_core` | `last_modified` | incident | `last_modified` | |
| `neris.incident_core` | `submitter_account_type` | incident | `submitter_account_type` | |
| `neris.incident_core` | `department_time_zone` | incident | `department.time_zone` | |
| `neris.incident_core` | `base_neris_uid` | incident | `base.neris_uid` | |
| `neris.incident_core` | `base_incident_number` | incident | `base.incident_number` | |
| `neris.incident_core` | `people_present` | incident | `base.people_present` | |
| `neris.incident_core` | `animals_rescued` | incident | `base.animals_rescued` | |
| `neris.incident_core` | `displacement_count` | incident | `base.displacement_count` | |
| `neris.incident_core` | `impediment_narrative` | incident | `base.impediment_narrative` | |
| `neris.incident_core` | `outcome_narrative` | incident | `base.outcome_narrative` | |
| `neris.incident_core` | `location_use_type` | incident | `base.location_use.use_type` | |
| `neris.incident_core` | `location_use_in_use` | incident | `base.location_use.in_use.in_use` | |
| `neris.incident_core` | `location_use_vacancy_cause` | incident | `base.location_use.vacancy_cause` | |
| `neris.incident_core` | `incident_status` | incident | `incident_status.status` | |
| `neris.incident_core` | `incident_status_created_by` | incident | `incident_status.created_by` | |
| `neris.incident_core` | `incident_status_last_modified` | incident | `incident_status.last_modified` | |
| `neris.incident_core` | `tactic_command_established` | incident | `tactic_timestamps.command_established` | |
| `neris.incident_core` | `tactic_completed_sizeup` | incident | `tactic_timestamps.completed_sizeup` | |
| `neris.incident_core` | `tactic_suppression_complete` | incident | `tactic_timestamps.suppression_complete` | |
| `neris.incident_core` | `tactic_primary_search_begin` | incident | `tactic_timestamps.primary_search_begin` | |
| `neris.incident_core` | `tactic_primary_search_complete` | incident | `tactic_timestamps.primary_search_complete` | |
| `neris.incident_core` | `tactic_water_on_fire` | incident | `tactic_timestamps.water_on_fire` | |
| `neris.incident_core` | `tactic_fire_under_control` | incident | `tactic_timestamps.fire_under_control` | |
| `neris.incident_core` | `tactic_fire_knocked_down` | incident | `tactic_timestamps.fire_knocked_down` | |
| `neris.incident_core` | `tactic_extrication_complete` | incident | `tactic_timestamps.extrication_complete` | |
| `neris.incident_core` | `smoke_alarm_presence` | incident | `smoke_alarm.presence.type` | |
| `neris.incident_core` | `fire_alarm_presence` | incident | `fire_alarm.presence.type` | |
| `neris.incident_core` | `other_alarm_presence` | incident | `other_alarm.presence.type` | |
| `neris.incident_core` | `fire_suppression_presence` | incident | `fire_suppression.presence.type` | |
| `neris.incident_core` | `medical_oxygen_presence` | incident | `medical_oxygen_hazard.presence.type` | |
| `neris.incident_core` | `point_url` | incident | `base.location.point` → `base.point` | first non-empty |
| `neris.incident_core` | `polygon_url` | incident | `base.polygon` | |

## `neris.incident_location`
**Grain:** 1 row per incident · **Root:** `base.location`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_location` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_location` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_location` | `location_neris_uid` | incident | `base.location.neris_uid` | |
| `neris.incident_location` | `neighborhood_community` | incident | `base.location.neighborhood_community` | |
| `neris.incident_location` | `incorporated_municipality` | incident | `base.location.incorporated_municipality` | |
| `neris.incident_location` | `county` | incident | `base.location.county` | |
| `neris.incident_location` | `state` | incident | `base.location.state` | |
| `neris.incident_location` | `postal_code` | incident | `base.location.postal_code` | |
| `neris.incident_location` | `postal_code_extension` | incident | `base.location.postal_code_extension` | |
| `neris.incident_location` | `street_prefix_direction` | incident | `base.location.street_prefix_direction` | |
| `neris.incident_location` | `street` | incident | `base.location.street` | |
| `neris.incident_location` | `street_postfix` | incident | `base.location.street_postfix` | |
| `neris.incident_location` | `complete_number` | incident | `base.location.complete_number` | |
| `neris.incident_location` | `room` | incident | `base.location.room` | |
| `neris.incident_location` | `floor` | incident | `base.location.floor` | |
| `neris.incident_location` | `unit_value` | incident | `base.location.unit_value` | |
| `neris.incident_location` | `structure` | incident | `base.location.structure` | |
| `neris.incident_location` | `additional_info` | incident | `base.location.additional_info` | |

## `neris.incident_displacement_cause`
**Grain:** 1 row per displacement cause · **Root:** `base.displacement_causes[]` (array of strings)

> *Design note: this string list stays as one row per value at Bronze (confirmed); its final reporting shape — rows vs. a single concatenated field — is defined in Silver.*

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_displacement_cause` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_displacement_cause` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_displacement_cause` | `cause` 🔑 | incident | `base.displacement_causes[]` | array element value |

## `neris.incident_type`
**Grain:** 1 row per incident type · **Root:** `incident_types[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_type` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_type` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_type` | `neris_uid` 🔑 | incident | `incident_types[].neris_uid` | |
| `neris.incident_type` | `is_primary` | incident | `incident_types[].primary` | renamed from `primary` |
| `neris.incident_type` | `type_code` | incident | `incident_types[].type` | |

## `neris.incident_special_modifier`
**Grain:** 1 row per modifier · **Root:** `special_modifiers[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_special_modifier` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_special_modifier` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_special_modifier` | `neris_uid` 🔑 | incident | `special_modifiers[].neris_uid` | |
| `neris.incident_special_modifier` | `type_code` | incident | `special_modifiers[].type` | |

## `neris.incident_aid`
**Grain:** 1 row per mutual aid · **Root:** `aids[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_aid` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_aid` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_aid` | `neris_uid` 🔑 | incident | `aids[].neris_uid` | |
| `neris.incident_aid` | `type_code` | incident | `aids[].type` | |
| `neris.incident_aid` | `aid_details_json` | incident | `aids[]` (extra keys) | JSON blob of keys beyond `neris_uid`/`last_modified`/`type` |

## `neris.incident_nonfd_aid`
**Grain:** 1 row per non-FD aid · **Root:** `nonfd_aids[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_nonfd_aid` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_nonfd_aid` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_nonfd_aid` | `neris_uid` 🔑 | incident | `nonfd_aids[].neris_uid` | |
| `neris.incident_nonfd_aid` | `type_code` | incident | `nonfd_aids[].type` | |

## `neris.incident_action_tactic`
**Grain:** 1 row per action · **Root:** `actions_tactics.action_noaction.actions[]`

> *Design note: this string list stays as one row per value at Bronze (confirmed); its final reporting shape — rows vs. a single concatenated field — is defined in Silver.*

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_action_tactic` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_action_tactic` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_action_tactic` | `action_noaction_type` | incident | `actions_tactics.action_noaction.type` | |
| `neris.incident_action_tactic` | `action_code` 🔑 | incident | `actions_tactics.action_noaction.actions[]` | array element value |

## `neris.incident_dispatch`
**Grain:** 1 row per incident · **Root:** `dispatch`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_dispatch` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_dispatch` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_dispatch` | `dispatch_neris_uid` | incident | `dispatch.neris_uid` | |
| `neris.incident_dispatch` | `dispatch_incident_number` | incident | `dispatch.incident_number` | |
| `neris.incident_dispatch` | `determinant_code` | incident | `dispatch.determinant_code` | |
| `neris.incident_dispatch` | `incident_code` | incident | `dispatch.incident_code` | |
| `neris.incident_dispatch` | `automatic_alarm` | incident | `dispatch.automatic_alarm` | |
| `neris.incident_dispatch` | `incident_clear` | incident | `dispatch.incident_clear` | |
| `neris.incident_dispatch` | `call_arrival` | incident | `dispatch.call_arrival` | |
| `neris.incident_dispatch` | `call_answered` | incident | `dispatch.call_answered` | |
| `neris.incident_dispatch` | `call_create` | incident | `dispatch.call_create` | |
| `neris.incident_dispatch` | `disposition` | incident | `dispatch.disposition` | |
| `neris.incident_dispatch` | `dispatch_point_url` | incident | `dispatch.point` | |
| `neris.incident_dispatch` | `dispatch_geocode_score` | incident | `dispatch.location.geocoded_location.score` | |
| `neris.incident_dispatch` | `dispatch_geocode_point_url` | incident | `dispatch.location.geocoded_location.point` | |

## `neris.incident_dispatch_unit_response`
**Grain:** 1 row per dispatched unit · **Root:** `dispatch.unit_responses[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_dispatch_unit_response` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_dispatch_unit_response` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_dispatch_unit_response` | `neris_uid` 🔑 | incident | `dispatch.unit_responses[].neris_uid` | |
| `neris.incident_dispatch_unit_response` | `unit_neris_id` | incident | `dispatch.unit_responses[].unit_neris_id` | |
| `neris.incident_dispatch_unit_response` | `reported_unit_id` | incident | `dispatch.unit_responses[].reported_unit_id` | |
| `neris.incident_dispatch_unit_response` | `staffing` | incident | `dispatch.unit_responses[].staffing` | |
| `neris.incident_dispatch_unit_response` | `unable_to_dispatch` | incident | `dispatch.unit_responses[].unable_to_dispatch` | |
| `neris.incident_dispatch_unit_response` | `dispatch_ts` | incident | `dispatch.unit_responses[].dispatch` | renamed from `dispatch` |
| `neris.incident_dispatch_unit_response` | `enroute_to_scene_ts` | incident | `dispatch.unit_responses[].enroute_to_scene` | |
| `neris.incident_dispatch_unit_response` | `on_scene_ts` | incident | `dispatch.unit_responses[].on_scene` | |
| `neris.incident_dispatch_unit_response` | `canceled_enroute_ts` | incident | `dispatch.unit_responses[].canceled_enroute` | |
| `neris.incident_dispatch_unit_response` | `staging_ts` | incident | `dispatch.unit_responses[].staging` | |
| `neris.incident_dispatch_unit_response` | `unit_clear_ts` | incident | `dispatch.unit_responses[].unit_clear` | |
| `neris.incident_dispatch_unit_response` | `response_mode` | incident | `dispatch.unit_responses[].response_mode` | |
| `neris.incident_dispatch_unit_response` | `transport_mode` | incident | `dispatch.unit_responses[].transport_mode` | |

## `neris.incident_unit_response`
**Grain:** 1 row per top-level unit response · **Root:** `unit_responses[]` (same shape as dispatch unit responses, but the fire department's own reporting)

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_unit_response` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_unit_response` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_unit_response` | `neris_uid` 🔑 | incident | `unit_responses[].neris_uid` | |
| `neris.incident_unit_response` | `unit_neris_id` | incident | `unit_responses[].unit_neris_id` | |
| `neris.incident_unit_response` | `reported_unit_id` | incident | `unit_responses[].reported_unit_id` | |
| `neris.incident_unit_response` | `staffing` | incident | `unit_responses[].staffing` | |
| `neris.incident_unit_response` | `unable_to_dispatch` | incident | `unit_responses[].unable_to_dispatch` | |
| `neris.incident_unit_response` | `dispatch_ts` | incident | `unit_responses[].dispatch` | renamed from `dispatch` |
| `neris.incident_unit_response` | `enroute_to_scene_ts` | incident | `unit_responses[].enroute_to_scene` | |
| `neris.incident_unit_response` | `on_scene_ts` | incident | `unit_responses[].on_scene` | |
| `neris.incident_unit_response` | `canceled_enroute_ts` | incident | `unit_responses[].canceled_enroute` | |
| `neris.incident_unit_response` | `staging_ts` | incident | `unit_responses[].staging` | |
| `neris.incident_unit_response` | `unit_clear_ts` | incident | `unit_responses[].unit_clear` | |
| `neris.incident_unit_response` | `response_mode` | incident | `unit_responses[].response_mode` | |
| `neris.incident_unit_response` | `transport_mode` | incident | `unit_responses[].transport_mode` | |

## `neris.incident_casualty_rescue`
**Grain:** 1 row per casualty/rescue person · **Root:** `casualty_rescues[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_casualty_rescue` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_casualty_rescue` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_casualty_rescue` | `neris_uid` 🔑 | incident | `casualty_rescues[].neris_uid` | |
| `neris.incident_casualty_rescue` | `cr_type` | incident | `casualty_rescues[].type` | `FF` / `NONFF` |
| `neris.incident_casualty_rescue` | `rank` | incident | `casualty_rescues[].rank` | |
| `neris.incident_casualty_rescue` | `years_of_service` | incident | `casualty_rescues[].years_of_service` | |
| `neris.incident_casualty_rescue` | `birth_month_year` | incident | `casualty_rescues[].birth_month_year` | |
| `neris.incident_casualty_rescue` | `gender` | incident | `casualty_rescues[].gender` | |
| `neris.incident_casualty_rescue` | `race` | incident | `casualty_rescues[].race` | |
| `neris.incident_casualty_rescue` | `casualty_neris_uid` | incident | `casualty_rescues[].casualty.neris_uid` | |
| `neris.incident_casualty_rescue` | `casualty_type` | incident | `casualty_rescues[].casualty.injury_or_noninjury.type` | |
| `neris.incident_casualty_rescue` | `casualty_cause` | incident | `casualty_rescues[].casualty.injury_or_noninjury.cause` | |
| `neris.incident_casualty_rescue` | `rescue_neris_uid` | incident | `casualty_rescues[].rescue.neris_uid` | |
| `neris.incident_casualty_rescue` | `rescue_type` | incident | `casualty_rescues[].rescue.ffrescue_or_nonffrescue.type` | |
| `neris.incident_casualty_rescue` | `rescue_presence_known_type` | incident | `casualty_rescues[].rescue.presence_known.presence_known_type` | |
| `neris.incident_casualty_rescue` | `rescue_impediments_json` | incident | `casualty_rescues[].rescue.ffrescue_or_nonffrescue.impediments` | JSON blob |
| `neris.incident_casualty_rescue` | `rescue_actions_json` | incident | `casualty_rescues[].rescue.ffrescue_or_nonffrescue.actions` | JSON blob |
| `neris.incident_casualty_rescue` | `rescue_removal_type` | incident | `casualty_rescues[].rescue.ffrescue_or_nonffrescue.removal_or_nonremoval.type` | |
| `neris.incident_casualty_rescue` | `mayday` | incident | `casualty_rescues[].rescue.mayday` | |

## `neris.incident_medical_detail`
**Grain:** 1 row per medical detail · **Root:** `medical_details[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_medical_detail` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_medical_detail` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_medical_detail` | `neris_uid` 🔑 | incident | `medical_details[].neris_uid` | |
| `neris.incident_medical_detail` | `patient_care_evaluation` | incident | `medical_details[].patient_care_evaluation` | |
| `neris.incident_medical_detail` | `patient_status` | incident | `medical_details[].patient_status` | |
| `neris.incident_medical_detail` | `transport_disposition` | incident | `medical_details[].transport_disposition` | |
| `neris.incident_medical_detail` | `patient_care_report_id` | incident | `medical_details[].patient_care_report_id` | |

## `neris.incident_exposure`
**Grain:** 1 row per exposure · **Root:** `exposures[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_exposure` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_exposure` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_exposure` | `neris_uid` 🔑 | incident | `exposures[].neris_uid` | |
| `neris.incident_exposure` | `exposure_json` | incident | `exposures[]` | full object as JSON blob |

## `neris.incident_fire_detail`
**Grain:** 1 row per incident (fire incidents only) · **Root:** `fire_detail`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_fire_detail` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_fire_detail` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_fire_detail` | `neris_uid` | incident | `fire_detail.neris_uid` | |
| `neris.incident_fire_detail` | `last_modified` | incident | `fire_detail.last_modified` | |
| `neris.incident_fire_detail` | `location_type` | incident | `fire_detail.location.type` | `STRUCTURE` / `OUTSIDE` |
| `neris.incident_fire_detail` | `progression_evident` | incident | `fire_detail.progression_evident` | |
| `neris.incident_fire_detail` | `floor_of_origin` | incident | `fire_detail.floor_of_origin` | |
| `neris.incident_fire_detail` | `arrival_condition` | incident | `fire_detail.arrival_condition` | |
| `neris.incident_fire_detail` | `damage_type` | incident | `fire_detail.damage_type` | |
| `neris.incident_fire_detail` | `room_of_origin_type` | incident | `fire_detail.room_of_origin_type` | |
| `neris.incident_fire_detail` | `cause` | incident | `fire_detail.cause` | |
| `neris.incident_fire_detail` | `acres_burned` | incident | `fire_detail.acres_burned` | |
| `neris.incident_fire_detail` | `water_supply` | incident | `fire_detail.water_supply` | |
| `neris.incident_fire_detail` | `investigation_needed` | incident | `fire_detail.investigation_needed` | |
| `neris.incident_fire_detail` | `investigation_types_json` | incident | `fire_detail.investigation_types` | JSON blob |
| `neris.incident_fire_detail` | `suppression_appliances_json` | incident | `fire_detail.suppression_appliances` | JSON blob |

## `neris.incident_fire_detail_electric_hazard`
**Grain:** 1 row per electric hazard · **Root:** `electric_hazards[]` (incident root; named for its fire context)

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_fire_detail_electric_hazard` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_fire_detail_electric_hazard` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_fire_detail_electric_hazard` | `neris_uid` 🔑 | incident | `electric_hazards[].neris_uid` | |
| `neris.incident_fire_detail_electric_hazard` | `type` | incident | `electric_hazards[].type` | |
| `neris.incident_fire_detail_electric_hazard` | `source_or_target` | incident | `electric_hazards[].source_or_target` | |
| `neris.incident_fire_detail_electric_hazard` | `involved_in_crash` | incident | `electric_hazards[].involved_in_crash` | |
| `neris.incident_fire_detail_electric_hazard` | `fire_details_json` | incident | `electric_hazards[].fire_details` | JSON blob |

## `neris.incident_hazsit_detail`
**Grain:** 1 row per incident (hazsit incidents only) · **Root:** `hazsit_detail`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_hazsit_detail` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_hazsit_detail` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_hazsit_detail` | `neris_uid` | incident | `hazsit_detail.neris_uid` | |
| `neris.incident_hazsit_detail` | `last_modified` | incident | `hazsit_detail.last_modified` | |
| `neris.incident_hazsit_detail` | `evacuated` | incident | `hazsit_detail.evacuated` | |
| `neris.incident_hazsit_detail` | `disposition` | incident | `hazsit_detail.disposition` | |
| `neris.incident_hazsit_detail` | `dot_class` | incident | `hazsit_detail.dot_class` | |

## `neris.incident_hazsit_chemical`
**Grain:** 1 row per chemical · **Root:** `hazsit_detail.chemicals[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_hazsit_chemical` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_hazsit_chemical` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_hazsit_chemical` | `neris_uid` 🔑 | incident | `hazsit_detail.chemicals[].neris_uid` | |
| `neris.incident_hazsit_chemical` | `name` | incident | `hazsit_detail.chemicals[].name` | |
| `neris.incident_hazsit_chemical` | `release_occurred` | incident | `hazsit_detail.chemicals[].release_occurred` | |
| `neris.incident_hazsit_chemical` | `dot_class` | incident | `hazsit_detail.chemicals[].dot_class` | |
| `neris.incident_hazsit_chemical` | `release_json` | incident | `hazsit_detail.chemicals[].release` | JSON blob |

## `neris.incident_hazsit_medical_detail`
**Grain:** 1 row per medical detail · **Root:** `hazsit_detail.medical_details[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_hazsit_medical_detail` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_hazsit_medical_detail` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_hazsit_medical_detail` | `neris_uid` 🔑 | incident | `hazsit_detail.medical_details[].neris_uid` | |
| `neris.incident_hazsit_medical_detail` | `patient_care_evaluation` | incident | `hazsit_detail.medical_details[].patient_care_evaluation` | |
| `neris.incident_hazsit_medical_detail` | `patient_status` | incident | `hazsit_detail.medical_details[].patient_status` | |
| `neris.incident_hazsit_medical_detail` | `transport_disposition` | incident | `hazsit_detail.medical_details[].transport_disposition` | |
| `neris.incident_hazsit_medical_detail` | `patient_care_report_id` | incident | `hazsit_detail.medical_details[].patient_care_report_id` | |

## `neris.incident_weather`
**Grain:** 1 row per incident · **Root:** `weather`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_weather` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_weather` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_weather` | `timestamp` | incident | `weather.timestamp` | |
| `neris.incident_weather` | `main` | incident | `weather.main` | |
| `neris.incident_weather` | `description` | incident | `weather.description` | |
| `neris.incident_weather` | `temp` | incident | `weather.temp` | |
| `neris.incident_weather` | `dew_point` | incident | `weather.dew_point` | |
| `neris.incident_weather` | `feels_like` | incident | `weather.feels_like` | |
| `neris.incident_weather` | `pressure` | incident | `weather.pressure` | |
| `neris.incident_weather` | `humidity` | incident | `weather.humidity` | |
| `neris.incident_weather` | `temp_min` | incident | `weather.temp_min` | |
| `neris.incident_weather` | `temp_max` | incident | `weather.temp_max` | |
| `neris.incident_weather` | `heat_index` | incident | `weather.heat_index` | |
| `neris.incident_weather` | `windchill` | incident | `weather.windchill` | |
| `neris.incident_weather` | `visibility` | incident | `weather.visibility` | |
| `neris.incident_weather` | `wind_speed` | incident | `weather.wind_speed` | |
| `neris.incident_weather` | `wind_deg` | incident | `weather.wind_deg` | |
| `neris.incident_weather` | `cloud_percent` | incident | `weather.cloud_percent` | |
| `neris.incident_weather` | `rain_accum_1h` | incident | `weather.rain_accum_1h` | |
| `neris.incident_weather` | `snow_vol_1h` | incident | `weather.snow_vol_1h` | |
| `neris.incident_weather` | `day_part` | incident | `weather.day_part` | |
| `neris.incident_weather` | `air_quality_co` | incident | `weather.air_quality_co` | |
| `neris.incident_weather` | `air_quality_o3` | incident | `weather.air_quality_o3` | |
| `neris.incident_weather` | `air_quality_no2` | incident | `weather.air_quality_no2` | |
| `neris.incident_weather` | `air_quality_so2` | incident | `weather.air_quality_so2` | |
| `neris.incident_weather` | `air_quality_pm2_5` | incident | `weather.air_quality_pm2_5` | |
| `neris.incident_weather` | `air_quality_pm10` | incident | `weather.air_quality_pm10` | |
| `neris.incident_weather` | `air_quality_us_epa_index` | incident | `weather.air_quality_us_epa_index` | |
| `neris.incident_weather` | `air_quality_gb_defra_index` | incident | `weather.air_quality_gb_defra_index` | |
| `neris.incident_weather` | `pollen_hazel` | incident | `weather.pollen_hazel` | |
| `neris.incident_weather` | `pollen_alder` | incident | `weather.pollen_alder` | |
| `neris.incident_weather` | `pollen_birch` | incident | `weather.pollen_birch` | |
| `neris.incident_weather` | `pollen_oak` | incident | `weather.pollen_oak` | |
| `neris.incident_weather` | `pollen_grass` | incident | `weather.pollen_grass` | |
| `neris.incident_weather` | `pollen_mugwort` | incident | `weather.pollen_mugwort` | |
| `neris.incident_weather` | `pollen_ragweed` | incident | `weather.pollen_ragweed` | |

## `neris.incident_census_tract`
**Grain:** 1 row per incident · **Root:** `census_tract`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_census_tract` | `incident_neris_id` 🔑 | incident | `neris_id` | |
| `neris.incident_census_tract` | `entity_neris_id` | incident | `base.department_neris_id` | resolved |
| `neris.incident_census_tract` | `fips_code` | incident | `census_tract.fips_code` | |
| `neris.incident_census_tract` | `area` | incident | `census_tract.area` | |
| `neris.incident_census_tract` | `population_density` | incident | `census_tract.population_density` | |

---

# Endpoint: `entity`

Record root = each entity object. Entity lands either as a discovery page (`{"entities":[...]}`) or a single unwrapped object; `entity_neris_id` comes from the entity's own `neris_id` in both shapes.

## `neris.entity_core`
**Grain:** 1 row per entity · **Root:** entity object

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_core` | `entity_neris_id` 🔑 | entity | `neris_id` | |
| `neris.entity_core` | `name` | entity | `name` | |
| `neris.entity_core` | `version` | entity | `version` | |
| `neris.entity_core` | `last_modified` | entity | `last_modified` | |
| `neris.entity_core` | `onboarding_status` | entity | `onboarding_status` | |
| `neris.entity_core` | `department_type` | entity | `department_type` | |
| `neris.entity_core` | `entity_type` | entity | `entity_type` | |
| `neris.entity_core` | `address_line_1` | entity | `address_line_1` | |
| `neris.entity_core` | `address_line_2` | entity | `address_line_2` | |
| `neris.entity_core` | `city` | entity | `city` | |
| `neris.entity_core` | `state` | entity | `state` | |
| `neris.entity_core` | `zip_code` | entity | `zip_code` | |
| `neris.entity_core` | `mail_address_line_1` | entity | `mail_address_line_1` | |
| `neris.entity_core` | `mail_address_line_2` | entity | `mail_address_line_2` | |
| `neris.entity_core` | `mail_city` | entity | `mail_city` | |
| `neris.entity_core` | `mail_state` | entity | `mail_state` | |
| `neris.entity_core` | `mail_zip_code` | entity | `mail_zip_code` | |
| `neris.entity_core` | `email` | entity | `email` | |
| `neris.entity_core` | `website` | entity | `website` | |
| `neris.entity_core` | `time_zone` | entity | `time_zone` | |
| `neris.entity_core` | `internal_id` | entity | `internal_id` | |
| `neris.entity_core` | `rms_software` | entity | `rms_software` | |
| `neris.entity_core` | `continue_edu` | entity | `continue_edu` | |
| `neris.entity_core` | `population_protected` | entity | `population.protected` | |
| `neris.entity_core` | `population_source` | entity | `population.source` | |
| `neris.entity_core` | `dispatch_avl_usage` | entity | `dispatch.avl_usage` | |
| `neris.entity_core` | `dispatch_center_id` | entity | `dispatch.center_id` | |
| `neris.entity_core` | `dispatch_cad_software` | entity | `dispatch.cad_software` | |
| `neris.entity_core` | `dispatch_psap_type` | entity | `dispatch.psap_type` | |
| `neris.entity_core` | `dispatch_protocol_fire` | entity | `dispatch.protocol_fire` | |
| `neris.entity_core` | `dispatch_protocol_med` | entity | `dispatch.protocol_med` | |
| `neris.entity_core` | `staffing_ff_career_ft` | entity | `staffing.active_firefighters_career_ft` | |
| `neris.entity_core` | `staffing_ff_career_pt` | entity | `staffing.active_firefighters_career_pt` | |
| `neris.entity_core` | `staffing_ff_volunteer` | entity | `staffing.active_firefighters_volunteer` | |
| `neris.entity_core` | `staffing_ems_career_ft` | entity | `staffing.active_ems_only_career_ft` | |
| `neris.entity_core` | `staffing_ems_career_pt` | entity | `staffing.active_ems_only_career_pt` | |
| `neris.entity_core` | `staffing_ems_volunteer` | entity | `staffing.active_ems_only_volunteer` | |
| `neris.entity_core` | `staffing_civ_career_ft` | entity | `staffing.active_civilians_career_ft` | |
| `neris.entity_core` | `staffing_civ_career_pt` | entity | `staffing.active_civilians_career_pt` | |
| `neris.entity_core` | `staffing_civ_volunteer` | entity | `staffing.active_civilians_volunteer` | |
| `neris.entity_core` | `assessment_iso_rating` | entity | `assessment.iso_rating` | |
| `neris.entity_core` | `assessment_cpse_accredited` | entity | `assessment.cpse_accredited` | |
| `neris.entity_core` | `assessment_caas_accredited` | entity | `assessment.caas_accredited` | |
| `neris.entity_core` | `shift_count` | entity | `shift.count` | |
| `neris.entity_core` | `shift_duration` | entity | `shift.duration` | |
| `neris.entity_core` | `shift_signup` | entity | `shift.signup` | |
| `neris.entity_core` | `feature_flags_json` | entity | `feature_flags` | JSON blob |
| `neris.entity_core` | `location_json` | entity | `location` | JSON blob (entity-level geometry) |

## `neris.entity_station`
**Grain:** 1 row per station · **Root:** `stations[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_station` | `entity_neris_id` 🔑 | entity | `neris_id` | from entity root |
| `neris.entity_station` | `station_neris_id` 🔑 | entity | `stations[].neris_id` | |
| `neris.entity_station` | `station_id` | entity | `stations[].station_id` | |
| `neris.entity_station` | `internal_id` | entity | `stations[].internal_id` | |
| `neris.entity_station` | `version` | entity | `stations[].version` | |
| `neris.entity_station` | `address_line_1` | entity | `stations[].address_line_1` | |
| `neris.entity_station` | `address_line_2` | entity | `stations[].address_line_2` | |
| `neris.entity_station` | `city` | entity | `stations[].city` | |
| `neris.entity_station` | `state` | entity | `stations[].state` | |
| `neris.entity_station` | `zip_code` | entity | `stations[].zip_code` | |
| `neris.entity_station` | `staffing` | entity | `stations[].staffing` | |
| `neris.entity_station` | `location_url` | entity | `stations[].location` | renamed (URL scalar) |

## `neris.entity_station_unit`
**Grain:** 1 row per unit per station · **Root:** `stations[].units[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_station_unit` | `entity_neris_id` 🔑 | entity | `neris_id` | from entity root |
| `neris.entity_station_unit` | `station_neris_id` 🔑 | entity | `stations[].neris_id` | carried from parent station |
| `neris.entity_station_unit` | `unit_neris_id` 🔑 | entity | `stations[].units[].neris_id` | |
| `neris.entity_station_unit` | `version` | entity | `stations[].units[].version` | |
| `neris.entity_station_unit` | `type` | entity | `stations[].units[].type` | |
| `neris.entity_station_unit` | `staffing` | entity | `stations[].units[].staffing` | |
| `neris.entity_station_unit` | `dedicated_staffing` | entity | `stations[].units[].dedicated_staffing` | |
| `neris.entity_station_unit` | `cad_designation_1` | entity | `stations[].units[].cad_designation_1` | |
| `neris.entity_station_unit` | `cad_designation_2` | entity | `stations[].units[].cad_designation_2` | |

## `neris.entity_region_set`
**Grain:** 1 row per region set · **Root:** `region_sets[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_region_set` | `entity_neris_id` 🔑 | entity | `neris_id` | from entity root |
| `neris.entity_region_set` | `name` 🔑 | entity | `region_sets[].name` | |
| `neris.entity_region_set` | `type` 🔑 | entity | `region_sets[].type` | |
| `neris.entity_region_set` | `is_primary` | entity | `region_sets[].primary` | renamed from `primary` |
| `neris.entity_region_set` | `is_coverage` | entity | `region_sets[].coverage` | renamed from `coverage` |
| `neris.entity_region_set` | `is_juris` | entity | `region_sets[].juris` | renamed from `juris` |

## `neris.entity_region`
**Grain:** 1 row per region per region set · **Root:** `region_sets[].regions[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_region` | `entity_neris_id` 🔑 | entity | `neris_id` | from entity root |
| `neris.entity_region` | `region_set_name` 🔑 | entity | `region_sets[].name` | carried from parent region set |
| `neris.entity_region` | `region_set_type` | entity | `region_sets[].type` | carried from parent region set |
| `neris.entity_region` | `region_name` | entity | `region_sets[].regions[].name` | |
| `neris.entity_region` | `internal_id` | entity | `region_sets[].regions[].internal_id` | |
| `neris.entity_region` | `url` 🔑 | entity | `region_sets[].regions[].url` | |

## `neris.entity_aid_agreement`
**Grain:** 1 row per aid partner · **Root:** `aid_agreements[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_aid_agreement` | `entity_neris_id` 🔑 | entity | `neris_id` | from entity root |
| `neris.entity_aid_agreement` | `partner_neris_id` 🔑 | entity | `aid_agreements[].neris_id_dept` | renamed |
| `neris.entity_aid_agreement` | `type` | entity | `aid_agreements[].type` | |
| `neris.entity_aid_agreement` | `confirmed` | entity | `aid_agreements[].confirmed` | |

## `neris.entity_service`
**Grain:** 1 row per service code · **Root:** `fire_services[]` / `ems_services[]` / `investigation_services[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.entity_service` | `entity_neris_id` 🔑 | entity | `neris_id` | from entity root |
| `neris.entity_service` | `service_category` 🔑 | entity | *derived* | `fire` / `ems` / `investigation` (which array the code came from) |
| `neris.entity_service` | `service_code` 🔑 | entity | `fire_services[]` \| `ems_services[]` \| `investigation_services[]` | array element value |

---

# Endpoint: `incident_history`

Record root = each element of `history[]`. The incident and entity ids are **not** in the payload — they are derived from the filename and path.

## `neris.incident_history_event`
**Grain:** 1 row per status history event · **Root:** `history[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.incident_history_event` | `incident_neris_id` 🔑 | incident_history | *derived (filename)* | parsed `FD…\|dispatch\|epoch` from `incident_history_…` filename |
| `neris.incident_history_event` | `entity_neris_id` | incident_history | *derived (path)* | from the landing-zone path segment |
| `neris.incident_history_event` | `last_modified` 🔑 | incident_history | `history[].last_modified` | |
| `neris.incident_history_event` | `is_current` | incident_history | `history[].current` | renamed (`current` is a SQL reserved word) |
| `neris.incident_history_event` | `account_id` | incident_history | `history[].account_id` | |
| `neris.incident_history_event` | `sub` | incident_history | `history[].sub` | |
| `neris.incident_history_event` | `client_id` | incident_history | `history[].client_id` | |
| `neris.incident_history_event` | `status` 🔑 | incident_history | `history[].status` | |

---

# Endpoint: `accountenrollment` *(dormant)*

Not produced by the current ingest (`NB_NERIS_API_DIF_incremental`); owned by `NB_NERIS_ENROLLED_ENTITIES_CONFIG_v1`. The table/extractor exist but stay empty until such files land. Record root = each element of `enrollments[]`.

## `neris.accountenrollment_core`
**Grain:** 1 row per enrollment · **Root:** `enrollments[]`

| Bronze Table Name | Bronze Column Name | Endpoint | Original Field | Notes |
|---|---|---|---|---|
| `neris.accountenrollment_core` | `enrollment_id` 🔑 | accountenrollment | `enrollments[].id` | renamed |
| `neris.accountenrollment_core` | `entity_neris_id` | accountenrollment | `enrollments[].neris_id` | falls back to path-derived id |
| `neris.accountenrollment_core` | `client_id` | accountenrollment | `enrollments[].client_id` | |
| `neris.accountenrollment_core` | `entity_name` | accountenrollment | `enrollments[].entity_name` | |
| `neris.accountenrollment_core` | `created_by` | accountenrollment | `enrollments[].created_by` | |
| `neris.accountenrollment_core` | `approved_by` | accountenrollment | `enrollments[].approved_by` | |
| `neris.accountenrollment_core` | `integration_title` | accountenrollment | `enrollments[].integration_title` | |
