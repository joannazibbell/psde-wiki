# NERIS Silver Schema (`neris`)

Reference for the Silver Delta tables produced by **`NB_DIF_BRZ_to_SLV_NERIS`**, which reads NERIS Bronze Delta tables (all-string columns) from `LH_BRONZE_LAYER` and writes typed, cleansed Silver Delta tables into the `neris` schema in `LH_SILVER_LAYER`.

This document maps every Silver **table** and **column** with its **target data type**, **source Bronze column**, and any **transformations** applied. Source: derived from `Neris_Bronze_Schema.md` and the Silver notebook's schema registry.

> **Schema location:** schema-enabled lakehouse → `Tables/neris/<table>`; non-schema lakehouse → `Tables/neris_<table>`. SQL: `neris.<table>` (or `neris_<table>`).

---

## Conventions

- **Silver columns are strongly typed.** Bronze stores everything as `string`; Silver applies proper types (timestamp, integer, float, boolean) for analytics.
- **`NULL` semantics:** All Bronze empty strings (`''`) become `NULL` at Silver. `NULL` means "no value" — the field was absent or empty in the source.
- **Trimming:** All string columns are trimmed of leading/trailing whitespace.
- **`_json` columns** remain as `string` type but are **validated** — only parseable JSON is kept; unparseable values become `NULL`. Empty arrays `[]` are valid and preserved.
- **SCD2 tables** (`incident_core`, `entity_core`) carry additional tracking columns: `IsCurrent`, `RecordStartDate`, `RecordEndDate`, `RecordModifiedDate`, `IsDeleted`.
- **Type 1 tables** (all others) use MERGE on natural keys — matched rows update in place; new rows insert.
- **Natural key** columns are marked 🔑.
- **Derived** columns are computed at Silver (not directly from a single Bronze column). Marked in the *Transformation* column.
- **`HashedNonKeyColumns`** (MD5 of all non-key business columns) is added to every table for change detection during MERGE.

### Silver lineage columns (present on **every** table)

| Silver Column Name | Data Type | Description |
|---|---|---|
| `_silver_run_id` | string | Unique run identifier for this Silver load (e.g., `neris_slv_20260613T120000Z`) |
| `_silver_timestamp` | string | UTC timestamp when this Silver run executed |
| `HashedNonKeyColumns` | string | MD5 hash of all non-key business columns for change detection |

### SCD2 columns (present on `incident_core` and `entity_core` only)

| Silver Column Name | Data Type | Description |
|---|---|---|
| `IsCurrent` | boolean | `True` for the current version of a record |
| `RecordStartDate` | timestamp | When this version became effective |
| `RecordEndDate` | timestamp | When this version was superseded (`9999-12-31` for current) |
| `RecordModifiedDate` | timestamp | Last modification timestamp |
| `IsDeleted` | boolean | Soft-delete flag |

### Bronze columns NOT carried to Silver

The following Bronze metadata columns are dropped during Silver processing (Silver adds its own lineage):

| Dropped Column | Reason |
|---|---|
| `_source_file_path` | Bronze-specific; Silver uses `_silver_run_id` for lineage |
| `_source_entity_id` | Bronze-specific |
| `_ingest_timestamp` | Replaced by `_silver_timestamp` |
| `_ingest_run_id` | Replaced by `_silver_run_id` |
| `_payload_hash` | Replaced by `HashedNonKeyColumns` |

---

## Client-Requested Transformations

### 1. `type_code` Hierarchy Split (`incident_type`)

The Bronze `type_code` column encodes a 3-level hierarchy delimited by `||` (e.g., `FIRE||STRUCTURE_FIRE||STRUCTURAL_INVOLVEMENT_FIRE`). At Silver this is split into three separate columns and the original `type_code` is dropped:

| Silver Column | Source | Example |
|---|---|---|
| `type_level1_type` | `split(type_code, '\|\|')[0]` | `FIRE` |
| `type_level2_description` | `split(type_code, '\|\|')[1]` | `STRUCTURE_FIRE` |
| `type_level3_detail` | `split(type_code, '\|\|')[2]` | `STRUCTURAL_INVOLVEMENT_FIRE` |

### 2. Displacement Causes Aggregation (`incident_core`)

The Bronze table `incident_displacement_cause` (one row per cause per incident) is aggregated into a single `displacement_causes` column on `incident_core`, with values separated by ` | `.

| Example Bronze rows | Silver `displacement_causes` value |
|---|---|
| `SMOKE`, `WATER`, `FIRE_DAMAGE` | `SMOKE \| WATER \| FIRE_DAMAGE` |

### 3. Actions Aggregation (`incident_core`)

The Bronze table `incident_action_tactic` (one row per action per incident) is aggregated into a single `actions` column on `incident_core`, with values separated by ` | `.

| Example Bronze rows | Silver `actions` value |
|---|---|
| `EMERGENCY_MEDICAL_CARE`, `INVESTIGATION` | `EMERGENCY_MEDICAL_CARE \| INVESTIGATION` |

### 4. Response Time Metrics (`incident_dispatch_unit_response`, `incident_unit_response`)

Derived time-based columns computed from dispatch/unit response timestamps:

| Silver Column | Formula | Unit |
|---|---|---|
| `response_time_seconds` | `on_scene_ts` − `dispatch_ts` | seconds (integer) |
| `turnout_time_seconds` | `enroute_to_scene_ts` − `dispatch_ts` | seconds (integer) |
| `travel_time_seconds` | `on_scene_ts` − `enroute_to_scene_ts` | seconds (integer) |

> These are `NULL` when either timestamp in the formula is `NULL`.

---

## Tables NOT Written to Silver (Denormalized)

The following Bronze tables are **not** created as separate Silver tables. Their data is aggregated onto `incident_core` instead:

| Bronze Table | Aggregated As | Target |
|---|---|---|
| `neris.incident_displacement_cause` | `displacement_causes` (pipe-separated) | `neris.incident_core` |
| `neris.incident_action_tactic` | `actions` (pipe-separated) | `neris.incident_core` |

---

## Table Inventory (28 Silver tables)

| Source Endpoint | Tables | Count |
|---|---|---|
| `incident` | `incident_core`, `incident_location`, `incident_type`, `incident_special_modifier`, `incident_aid`, `incident_nonfd_aid`, `incident_dispatch`, `incident_dispatch_unit_response`, `incident_unit_response`, `incident_casualty_rescue`, `incident_medical_detail`, `incident_exposure`, `incident_fire_detail`, `incident_fire_detail_electric_hazard`, `incident_hazsit_detail`, `incident_hazsit_chemical`, `incident_hazsit_medical_detail`, `incident_weather`, `incident_census_tract` | 19 |
| `entity` | `entity_core`, `entity_station`, `entity_station_unit`, `entity_region_set`, `entity_region`, `entity_aid_agreement`, `entity_service` | 7 |
| `incident_history` | `incident_history_event` | 1 |
| `accountenrollment` *(dormant)* | `accountenrollment_core` | 1 |

---

# Endpoint: `incident`

## `neris.incident_core`
**Grain:** 1 row per incident (SCD2 — multiple versions possible) · **MERGE type:** SCD2

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `last_modified` | timestamp | `last_modified` | ISO 8601 parse |
| `submitter_account_type` | string | `submitter_account_type` | trim |
| `department_time_zone` | string | `department_time_zone` | trim |
| `base_neris_uid` | string | `base_neris_uid` | trim |
| `base_incident_number` | string | `base_incident_number` | trim |
| `people_present` | integer | `people_present` | cast to int |
| `animals_rescued` | integer | `animals_rescued` | cast to int |
| `displacement_count` | integer | `displacement_count` | cast to int |
| `impediment_narrative` | string | `impediment_narrative` | trim |
| `outcome_narrative` | string | `outcome_narrative` | trim |
| `location_use_type` | string | `location_use_type` | trim |
| `location_use_in_use` | string | `location_use_in_use` | trim |
| `location_use_vacancy_cause` | string | `location_use_vacancy_cause` | trim |
| `incident_status` | string | `incident_status` | trim |
| `incident_status_created_by` | string | `incident_status_created_by` | trim |
| `incident_status_last_modified` | timestamp | `incident_status_last_modified` | ISO 8601 parse |
| `tactic_command_established` | timestamp | `tactic_command_established` | ISO 8601 parse |
| `tactic_completed_sizeup` | timestamp | `tactic_completed_sizeup` | ISO 8601 parse |
| `tactic_suppression_complete` | timestamp | `tactic_suppression_complete` | ISO 8601 parse |
| `tactic_primary_search_begin` | timestamp | `tactic_primary_search_begin` | ISO 8601 parse |
| `tactic_primary_search_complete` | timestamp | `tactic_primary_search_complete` | ISO 8601 parse |
| `tactic_water_on_fire` | timestamp | `tactic_water_on_fire` | ISO 8601 parse |
| `tactic_fire_under_control` | timestamp | `tactic_fire_under_control` | ISO 8601 parse |
| `tactic_fire_knocked_down` | timestamp | `tactic_fire_knocked_down` | ISO 8601 parse |
| `tactic_extrication_complete` | timestamp | `tactic_extrication_complete` | ISO 8601 parse |
| `smoke_alarm_presence` | string | `smoke_alarm_presence` | trim |
| `fire_alarm_presence` | string | `fire_alarm_presence` | trim |
| `other_alarm_presence` | string | `other_alarm_presence` | trim |
| `fire_suppression_presence` | string | `fire_suppression_presence` | trim |
| `medical_oxygen_presence` | string | `medical_oxygen_presence` | trim |
| `point_url` | string | `point_url` | trim |
| `polygon_url` | string | `polygon_url` | trim |
| `displacement_causes` | string | *aggregated from `incident_displacement_cause`* | derived: `collect_list(cause)` → pipe-separated |
| `actions` | string | *aggregated from `incident_action_tactic`* | derived: `collect_list(action_code)` → pipe-separated |

## `neris.incident_location`
**Grain:** 1 row per incident · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `location_neris_uid` | string | `location_neris_uid` | trim |
| `neighborhood_community` | string | `neighborhood_community` | trim |
| `incorporated_municipality` | string | `incorporated_municipality` | trim |
| `county` | string | `county` | trim |
| `state` | string | `state` | trim |
| `postal_code` | string | `postal_code` | trim |
| `postal_code_extension` | string | `postal_code_extension` | trim |
| `street_prefix_direction` | string | `street_prefix_direction` | trim |
| `street` | string | `street` | trim |
| `street_postfix` | string | `street_postfix` | trim |
| `complete_number` | string | `complete_number` | trim |
| `room` | string | `room` | trim |
| `floor` | string | `floor` | trim |
| `unit_value` | string | `unit_value` | trim |
| `structure` | string | `structure` | trim |
| `additional_info` | string | `additional_info` | trim |

## `neris.incident_type`
**Grain:** 1 row per incident type · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `is_primary` | boolean | `is_primary` | boolean parse |
| `type_level1_type` | string | `type_code` | derived: `split(type_code, '\|\|')[0]` |
| `type_level2_description` | string | `type_code` | derived: `split(type_code, '\|\|')[1]` |
| `type_level3_detail` | string | `type_code` | derived: `split(type_code, '\|\|')[2]` |

> **Note:** The Bronze `type_code` column is split and dropped — it does not appear in Silver.

## `neris.incident_special_modifier`
**Grain:** 1 row per modifier · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `type_code` | string | `type_code` | trim |

## `neris.incident_aid`
**Grain:** 1 row per mutual aid · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `type_code` | string | `type_code` | trim |
| `aid_details_json` | string | `aid_details_json` | JSON validated |

## `neris.incident_nonfd_aid`
**Grain:** 1 row per non-FD aid · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `type_code` | string | `type_code` | trim |

## `neris.incident_dispatch`
**Grain:** 1 row per incident · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `dispatch_neris_uid` | string | `dispatch_neris_uid` | trim |
| `dispatch_incident_number` | string | `dispatch_incident_number` | trim |
| `determinant_code` | string | `determinant_code` | trim |
| `incident_code` | string | `incident_code` | trim |
| `automatic_alarm` | boolean | `automatic_alarm` | boolean parse |
| `incident_clear` | timestamp | `incident_clear` | ISO 8601 parse |
| `call_arrival` | timestamp | `call_arrival` | ISO 8601 parse |
| `call_answered` | timestamp | `call_answered` | ISO 8601 parse |
| `call_create` | timestamp | `call_create` | ISO 8601 parse |
| `disposition` | string | `disposition` | trim |
| `dispatch_point_url` | string | `dispatch_point_url` | trim |
| `dispatch_geocode_score` | float | `dispatch_geocode_score` | cast to float |
| `dispatch_geocode_point_url` | string | `dispatch_geocode_point_url` | trim |

## `neris.incident_dispatch_unit_response`
**Grain:** 1 row per dispatched unit · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `unit_neris_id` | string | `unit_neris_id` | trim |
| `reported_unit_id` | string | `reported_unit_id` | trim |
| `staffing` | integer | `staffing` | cast to int |
| `unable_to_dispatch` | boolean | `unable_to_dispatch` | boolean parse |
| `dispatch_ts` | timestamp | `dispatch_ts` | ISO 8601 parse |
| `enroute_to_scene_ts` | timestamp | `enroute_to_scene_ts` | ISO 8601 parse |
| `on_scene_ts` | timestamp | `on_scene_ts` | ISO 8601 parse |
| `canceled_enroute_ts` | timestamp | `canceled_enroute_ts` | ISO 8601 parse |
| `staging_ts` | timestamp | `staging_ts` | ISO 8601 parse |
| `unit_clear_ts` | timestamp | `unit_clear_ts` | ISO 8601 parse |
| `response_mode` | string | `response_mode` | trim |
| `transport_mode` | string | `transport_mode` | trim |
| `response_time_seconds` | integer | *derived* | `on_scene_ts` − `dispatch_ts` (seconds) |
| `turnout_time_seconds` | integer | *derived* | `enroute_to_scene_ts` − `dispatch_ts` (seconds) |
| `travel_time_seconds` | integer | *derived* | `on_scene_ts` − `enroute_to_scene_ts` (seconds) |

## `neris.incident_unit_response`
**Grain:** 1 row per top-level unit response · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `unit_neris_id` | string | `unit_neris_id` | trim |
| `reported_unit_id` | string | `reported_unit_id` | trim |
| `staffing` | integer | `staffing` | cast to int |
| `unable_to_dispatch` | boolean | `unable_to_dispatch` | boolean parse |
| `dispatch_ts` | timestamp | `dispatch_ts` | ISO 8601 parse |
| `enroute_to_scene_ts` | timestamp | `enroute_to_scene_ts` | ISO 8601 parse |
| `on_scene_ts` | timestamp | `on_scene_ts` | ISO 8601 parse |
| `canceled_enroute_ts` | timestamp | `canceled_enroute_ts` | ISO 8601 parse |
| `staging_ts` | timestamp | `staging_ts` | ISO 8601 parse |
| `unit_clear_ts` | timestamp | `unit_clear_ts` | ISO 8601 parse |
| `response_mode` | string | `response_mode` | trim |
| `transport_mode` | string | `transport_mode` | trim |
| `response_time_seconds` | integer | *derived* | `on_scene_ts` − `dispatch_ts` (seconds) |
| `turnout_time_seconds` | integer | *derived* | `enroute_to_scene_ts` − `dispatch_ts` (seconds) |
| `travel_time_seconds` | integer | *derived* | `on_scene_ts` − `enroute_to_scene_ts` (seconds) |

## `neris.incident_casualty_rescue`
**Grain:** 1 row per casualty/rescue person · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `cr_type` | string | `cr_type` | trim |
| `rank` | string | `rank` | trim |
| `years_of_service` | integer | `years_of_service` | cast to int |
| `birth_month_year` | string | `birth_month_year` | trim |
| `gender` | string | `gender` | trim |
| `race` | string | `race` | trim |
| `casualty_neris_uid` | string | `casualty_neris_uid` | trim |
| `casualty_type` | string | `casualty_type` | trim |
| `casualty_cause` | string | `casualty_cause` | trim |
| `rescue_neris_uid` | string | `rescue_neris_uid` | trim |
| `rescue_type` | string | `rescue_type` | trim |
| `rescue_presence_known_type` | string | `rescue_presence_known_type` | trim |
| `rescue_impediments_json` | string | `rescue_impediments_json` | JSON validated |
| `rescue_actions_json` | string | `rescue_actions_json` | JSON validated |
| `rescue_removal_type` | string | `rescue_removal_type` | trim |
| `mayday` | boolean | `mayday` | boolean parse |

## `neris.incident_medical_detail`
**Grain:** 1 row per medical detail · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `patient_care_evaluation` | string | `patient_care_evaluation` | trim |
| `patient_status` | string | `patient_status` | trim |
| `transport_disposition` | string | `transport_disposition` | trim |
| `patient_care_report_id` | string | `patient_care_report_id` | trim |

## `neris.incident_exposure`
**Grain:** 1 row per exposure · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `exposure_json` | string | `exposure_json` | JSON validated |

## `neris.incident_fire_detail`
**Grain:** 1 row per incident (fire incidents only) · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` | string | `neris_uid` | trim |
| `last_modified` | timestamp | `last_modified` | ISO 8601 parse |
| `location_type` | string | `location_type` | trim |
| `progression_evident` | boolean | `progression_evident` | boolean parse |
| `floor_of_origin` | integer | `floor_of_origin` | cast to int |
| `arrival_condition` | string | `arrival_condition` | trim |
| `damage_type` | string | `damage_type` | trim |
| `room_of_origin_type` | string | `room_of_origin_type` | trim |
| `cause` | string | `cause` | trim |
| `acres_burned` | float | `acres_burned` | cast to float |
| `water_supply` | string | `water_supply` | trim |
| `investigation_needed` | boolean | `investigation_needed` | boolean parse |
| `investigation_types_json` | string | `investigation_types_json` | JSON validated |
| `suppression_appliances_json` | string | `suppression_appliances_json` | JSON validated |

## `neris.incident_fire_detail_electric_hazard`
**Grain:** 1 row per electric hazard · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `type` | string | `type` | trim |
| `source_or_target` | string | `source_or_target` | trim |
| `involved_in_crash` | boolean | `involved_in_crash` | boolean parse |
| `fire_details_json` | string | `fire_details_json` | JSON validated |

## `neris.incident_hazsit_detail`
**Grain:** 1 row per incident (hazsit incidents only) · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` | string | `neris_uid` | trim |
| `last_modified` | timestamp | `last_modified` | ISO 8601 parse |
| `evacuated` | integer | `evacuated` | cast to int |
| `disposition` | string | `disposition` | trim |
| `dot_class` | string | `dot_class` | trim |

## `neris.incident_hazsit_chemical`
**Grain:** 1 row per chemical · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `name` | string | `name` | trim |
| `release_occurred` | boolean | `release_occurred` | boolean parse |
| `dot_class` | string | `dot_class` | trim |
| `release_json` | string | `release_json` | JSON validated |

## `neris.incident_hazsit_medical_detail`
**Grain:** 1 row per medical detail · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `neris_uid` 🔑 | string | `neris_uid` | trim |
| `patient_care_evaluation` | string | `patient_care_evaluation` | trim |
| `patient_status` | string | `patient_status` | trim |
| `transport_disposition` | string | `transport_disposition` | trim |
| `patient_care_report_id` | string | `patient_care_report_id` | trim |

## `neris.incident_weather`
**Grain:** 1 row per incident · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `timestamp` | timestamp | `timestamp` | ISO 8601 parse |
| `main` | string | `main` | trim |
| `description` | string | `description` | trim |
| `temp` | float | `temp` | cast to float (°F) |
| `dew_point` | float | `dew_point` | cast to float (°F) |
| `feels_like` | float | `feels_like` | cast to float (°F) |
| `pressure` | float | `pressure` | cast to float (hPa) |
| `humidity` | integer | `humidity` | cast to int (%) |
| `temp_min` | float | `temp_min` | cast to float (°F) |
| `temp_max` | float | `temp_max` | cast to float (°F) |
| `heat_index` | float | `heat_index` | cast to float |
| `windchill` | float | `windchill` | cast to float |
| `visibility` | float | `visibility` | cast to float (meters) |
| `wind_speed` | float | `wind_speed` | cast to float |
| `wind_deg` | integer | `wind_deg` | cast to int (degrees) |
| `cloud_percent` | integer | `cloud_percent` | cast to int (%) |
| `rain_accum_1h` | float | `rain_accum_1h` | cast to float |
| `snow_vol_1h` | float | `snow_vol_1h` | cast to float |
| `day_part` | string | `day_part` | trim |
| `air_quality_co` | float | `air_quality_co` | cast to float |
| `air_quality_o3` | float | `air_quality_o3` | cast to float |
| `air_quality_no2` | float | `air_quality_no2` | cast to float |
| `air_quality_so2` | float | `air_quality_so2` | cast to float |
| `air_quality_pm2_5` | float | `air_quality_pm2_5` | cast to float |
| `air_quality_pm10` | float | `air_quality_pm10` | cast to float |
| `air_quality_us_epa_index` | float | `air_quality_us_epa_index` | cast to float |
| `air_quality_gb_defra_index` | float | `air_quality_gb_defra_index` | cast to float |
| `pollen_hazel` | integer | `pollen_hazel` | cast to int |
| `pollen_alder` | integer | `pollen_alder` | cast to int |
| `pollen_birch` | integer | `pollen_birch` | cast to int |
| `pollen_oak` | integer | `pollen_oak` | cast to int |
| `pollen_grass` | integer | `pollen_grass` | cast to int |
| `pollen_mugwort` | integer | `pollen_mugwort` | cast to int |
| `pollen_ragweed` | integer | `pollen_ragweed` | cast to int |

## `neris.incident_census_tract`
**Grain:** 1 row per incident · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `fips_code` | string | `fips_code` | trim |
| `area` | float | `area` | cast to float |
| `population_density` | float | `population_density` | cast to float |

---

# Endpoint: `entity`

## `neris.entity_core`
**Grain:** 1 row per entity (SCD2 — multiple versions possible) · **MERGE type:** SCD2

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `name` | string | `name` | trim |
| `version` | string | `version` | trim |
| `last_modified` | timestamp | `last_modified` | ISO 8601 parse |
| `onboarding_status` | string | `onboarding_status` | trim |
| `department_type` | string | `department_type` | trim |
| `entity_type` | string | `entity_type` | trim |
| `address_line_1` | string | `address_line_1` | trim |
| `address_line_2` | string | `address_line_2` | trim |
| `city` | string | `city` | trim |
| `state` | string | `state` | trim |
| `zip_code` | string | `zip_code` | trim |
| `mail_address_line_1` | string | `mail_address_line_1` | trim |
| `mail_address_line_2` | string | `mail_address_line_2` | trim |
| `mail_city` | string | `mail_city` | trim |
| `mail_state` | string | `mail_state` | trim |
| `mail_zip_code` | string | `mail_zip_code` | trim |
| `email` | string | `email` | trim |
| `website` | string | `website` | trim |
| `time_zone` | string | `time_zone` | trim |
| `internal_id` | string | `internal_id` | trim |
| `rms_software` | string | `rms_software` | trim |
| `continue_edu` | string | `continue_edu` | trim |
| `population_protected` | integer | `population_protected` | cast to int |
| `population_source` | string | `population_source` | trim |
| `dispatch_avl_usage` | string | `dispatch_avl_usage` | trim |
| `dispatch_center_id` | string | `dispatch_center_id` | trim |
| `dispatch_cad_software` | string | `dispatch_cad_software` | trim |
| `dispatch_psap_type` | string | `dispatch_psap_type` | trim |
| `dispatch_protocol_fire` | string | `dispatch_protocol_fire` | trim |
| `dispatch_protocol_med` | string | `dispatch_protocol_med` | trim |
| `staffing_ff_career_ft` | integer | `staffing_ff_career_ft` | cast to int |
| `staffing_ff_career_pt` | integer | `staffing_ff_career_pt` | cast to int |
| `staffing_ff_volunteer` | integer | `staffing_ff_volunteer` | cast to int |
| `staffing_ems_career_ft` | integer | `staffing_ems_career_ft` | cast to int |
| `staffing_ems_career_pt` | integer | `staffing_ems_career_pt` | cast to int |
| `staffing_ems_volunteer` | integer | `staffing_ems_volunteer` | cast to int |
| `staffing_civ_career_ft` | integer | `staffing_civ_career_ft` | cast to int |
| `staffing_civ_career_pt` | integer | `staffing_civ_career_pt` | cast to int |
| `staffing_civ_volunteer` | integer | `staffing_civ_volunteer` | cast to int |
| `assessment_iso_rating` | string | `assessment_iso_rating` | trim |
| `assessment_cpse_accredited` | boolean | `assessment_cpse_accredited` | boolean parse |
| `assessment_caas_accredited` | boolean | `assessment_caas_accredited` | boolean parse |
| `shift_count` | integer | `shift_count` | cast to int |
| `shift_duration` | float | `shift_duration` | cast to float (hours) |
| `shift_signup` | string | `shift_signup` | trim |
| `feature_flags_json` | string | `feature_flags_json` | JSON validated |
| `location_json` | string | `location_json` | JSON validated |

## `neris.entity_station`
**Grain:** 1 row per station · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `station_neris_id` 🔑 | string | `station_neris_id` | trim |
| `station_id` | string | `station_id` | trim |
| `internal_id` | string | `internal_id` | trim |
| `version` | string | `version` | trim |
| `address_line_1` | string | `address_line_1` | trim |
| `address_line_2` | string | `address_line_2` | trim |
| `city` | string | `city` | trim |
| `state` | string | `state` | trim |
| `zip_code` | string | `zip_code` | trim |
| `staffing` | integer | `staffing` | cast to int |
| `location_url` | string | `location_url` | trim |

## `neris.entity_station_unit`
**Grain:** 1 row per unit per station · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `station_neris_id` 🔑 | string | `station_neris_id` | trim |
| `unit_neris_id` 🔑 | string | `unit_neris_id` | trim |
| `version` | string | `version` | trim |
| `type` | string | `type` | trim |
| `staffing` | integer | `staffing` | cast to int |
| `dedicated_staffing` | integer | `dedicated_staffing` | cast to int |
| `cad_designation_1` | string | `cad_designation_1` | trim |
| `cad_designation_2` | string | `cad_designation_2` | trim |

## `neris.entity_region_set`
**Grain:** 1 row per region set · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `name` 🔑 | string | `name` | trim |
| `type` 🔑 | string | `type` | trim |
| `is_primary` | boolean | `is_primary` | boolean parse |
| `is_coverage` | boolean | `is_coverage` | boolean parse |
| `is_juris` | boolean | `is_juris` | boolean parse |

## `neris.entity_region`
**Grain:** 1 row per region per region set · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `region_set_name` 🔑 | string | `region_set_name` | trim |
| `region_set_type` | string | `region_set_type` | trim |
| `region_name` | string | `region_name` | trim |
| `internal_id` | string | `internal_id` | trim |
| `url` 🔑 | string | `url` | trim |

## `neris.entity_aid_agreement`
**Grain:** 1 row per aid partner · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `partner_neris_id` 🔑 | string | `partner_neris_id` | trim |
| `type` | string | `type` | trim |
| `confirmed` | boolean | `confirmed` | boolean parse |

## `neris.entity_service`
**Grain:** 1 row per service code · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `entity_neris_id` 🔑 | string | `entity_neris_id` | trim |
| `service_category` 🔑 | string | `service_category` | trim |
| `service_code` 🔑 | string | `service_code` | trim |

---

# Endpoint: `incident_history`

## `neris.incident_history_event`
**Grain:** 1 row per status history event · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `incident_neris_id` 🔑 | string | `incident_neris_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `last_modified` 🔑 | timestamp | `last_modified` | ISO 8601 parse |
| `is_current` | boolean | `is_current` | boolean parse |
| `account_id` | string | `account_id` | trim |
| `sub` | string | `sub` | trim |
| `client_id` | string | `client_id` | trim |
| `status` 🔑 | string | `status` | trim |

---

# Endpoint: `accountenrollment` *(dormant)*

## `neris.accountenrollment_core`
**Grain:** 1 row per enrollment · **MERGE type:** Type 1

| Silver Column Name | Data Type | Bronze Source | Transformation |
|---|---|---|---|
| `enrollment_id` 🔑 | string | `enrollment_id` | trim |
| `entity_neris_id` | string | `entity_neris_id` | trim |
| `client_id` | string | `client_id` | trim |
| `entity_name` | string | `entity_name` | trim |
| `created_by` | string | `created_by` | trim |
| `approved_by` | string | `approved_by` | trim |
| `integration_title` | string | `integration_title` | trim |
