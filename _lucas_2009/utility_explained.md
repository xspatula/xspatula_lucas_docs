---
title: "Utility Explained"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/utility_explained/
author_profile: false
---

Reference page — not required reading to complete the [Insert utility][insert_utility] step,
only if you want to know exactly which catalogue tables each of its four cells inserts, and
which ones are disabled by default.

**Path**: `xspatula_lucas/lucas/import_data/insert_utility.ipynb`

Each cell runs a `job_file`, which points at a pilot `.txt` file — a plain-text, ordered list of
per-table process files. Every entry is a single `insert_tabular_data` call: read one Excel file,
translate, and insert, in one step. Lines starting with `#` are comments and are skipped — some
tables are shipped commented out even though their Excel file already exists, because they're not
needed for the LUCAS 2009 campaign; add them by uncommenting the line.

## 1. Insert general utilities

**Job file**: `job_insert_general_utility.json` → pilot `general/insert_general_utility.txt`

| Table | Source | Notes |
|---|---|---|
| `territory` | `general/excel/territory.xlsx` | Territory/country codes — referenced by almost every other table via `territory_id__territory_name` |

Only one table today. `general/excel/foreign_key.xlsx` also exists in this directory but has no
corresponding pilot entry — orphaned, not currently wired up (see `notes/xspatula_lucas.md`).

## 2. Insert observation utilities

**Job file**: `job_insert_observation_utility.json` → pilot `observation/insert_observation_utility.txt`

Two tiers in one pilot file, in this fixed order — independent tables first, then tables that
reference them by foreign key:

**Without foreign-key dependencies** (but referenced by other tables):

`analysis_method`, `apparatus`, `classification_order`, `license`, `location_method`,
`method_tier`, `preparation`, `preservation`, `provider`, `quantity`, `setting_system`,
`spatial_reference`, `storage`, `transportation`, `unit`

**Dependent on the tier above:**

`classification_family` (needs `classification_order`), `classification_genus` (needs
`classification_family`), `indicator` (needs `quantity`), `indicator_parity` (needs `indicator`),
`juxtaposition` (needs `setting_system`), `profiling` (needs `unit`)

**Shipped disabled** (commented out in the pilot file): `classification_species`,
`quantity_default_unit`, and `unit_translate` — `unit_translate.xlsx` and a matching
`unit_translate.json` process file already exist under `observation/`, but the pilot entry is
commented out, so it isn't run by default. Also several `# TG TODO` placeholders with no Excel
file yet at all (coordinate system, monolith extraction, reference proprietor, soil horizon,
sound setup/mic, spectroscopy method, taxa levels/status/function, analysis method translate).

**Classification hierarchy note**: when you add a `classification_order` (e.g. `soil`), the same
name is automatically copied down to family/genus/species so foreign keys further down the
hierarchy always resolve even before you've defined anything more specific — you can always widen
detail later without breaking existing links.

## 3. Insert observation utilities with inherit

**Job file**: `job_insert_observation_utility_inherit.json` → pilot
`observation/insert_observation_utility_inherit.txt`

| Table | Requires (from step 2) |
|---|---|
| `provision` | `apparatus`, `provider`, `method_tier` |
| `provision_indicator` | `provision`, `indicator`, `method_tier`* |
| `provision_serial_nr` | `provision` |

\* the pilot file's comment says "method", the actual foreign key table is `method_tier` — checked
against `provision_indicator.xlsx`'s columns.

Despite the step's name, this has nothing to do with the `inherit`/`auto` *value* mechanism
covered in [Utility → inherit and auto][utility_inherit_auto] — "inherit" here just means these
three tables inherit foreign keys from step 2's tables, so they have to run after it. If you add
a new provision later, edit `provision.xlsx` and re-run this step alone.

## 4. Insert landscape utility

**Job file**: `job_insert_landscape_utility.json` → pilot `landscape/insert_landscape_utility.txt`

**Active by default** (6 tables, in dependency order):

`land_use_order`, `land_cover_order` → `land_use_family`, `land_use_genus`, `land_cover_family`,
`land_cover_genus`

**Shipped disabled**, prefixed `### REMOVE TO RUN###` in the pilot file — their Excel files exist
in `landscape/excel/` already, they're just not activated:

`crop_growth_stage`, `major_landform`, `slope_position`, `sky_conditions`, `ground_conditions`,
`soil_preparation`, `reference_soil_groups`, `soil_texture_classification_USDA`,
`soil_texture_classification_ISSS`

To activate one, open `landscape/insert_landscape_utility.txt` and delete the
`### REMOVE TO RUN###` prefix from its line, then re-run this cell.

## Next step

Once all four cells have run, proceed to [Insert dataset metadata][insert_dataset_meta] — several
of its fields (territory, license, spatial reference) resolve against tables inserted here.

[insert_utility]: /lucas_2009/insert_utility/
[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[utility_inherit_auto]: /lucas_2009/utility_inherit_auto_explained/
