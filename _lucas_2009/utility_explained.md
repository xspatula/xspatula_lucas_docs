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

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_data.ipynb`, block **1. Utilities** —
the four sections below are cells 1.1–1.4.

Each cell runs a `job_file`, which points at a pilot `.txt` file — a plain-text, ordered list of
per-table process files. Every entry is a single `insert_tabular_data` call: read one Excel file,
translate, and insert, in one step. Lines starting with `#` are comments and are skipped — some
tables are shipped commented out even though their Excel file already exists, because they're not
needed for the LUCAS 2009 campaign; add them by uncommenting the line.

## 1. Insert general utilities

**Notebook cell**: 1.1 Insert general utilities

**Job file**: `job_insert_general_utility.json` → pilot `general/insert_general_utility.txt`

| Table | Source | Notes |
|---|---|---|
| `territory` | `general/excel/territory.xlsx` | Territory/country codes — referenced by almost every other table via `territory_id__territory_name` |
| `foreign_key` | `general/excel/foreign_key.xlsx` | Non-standard foreign-key lookups — feeds the `xxx_id__yyy` resolver other processes fall back to when there's no table literally named `xxx` (e.g. `unit_translate`, below). See [Foreign key explained][foreign_key_explained]. |

## 2. Insert observation utilities

**Notebook cell**: 1.2 Insert observation utilities

**Job file**: `job_insert_observation_utility.json` → pilot `observation/insert_observation_utility.txt`

Two tiers in one pilot file, in this fixed order — independent tables first, then tables that
reference them by foreign key:

**Without foreign-key dependencies** (but referenced by other tables):

`analysis_method`, `apparatus`, `classification_order`, `license`, `location_method`,
`method_tier`, `preparation`, `preservation`, `provider`, `quantity`, `spatial_reference`,
`storage`, `transportation`, `unit`

**Dependent on the tier above:**

`classification_family` (needs `classification_order`), `classification_genus` (needs
`classification_family`), `indicator` (needs `quantity`), `indicator_parity` (needs `indicator`),
`profiling` (needs `unit`), `unit_translate` (needs `unit` — and, since its foreign keys don't
follow the plain `xxx_id__yyy` convention, the `foreign_key` table from step 1; see
[Foreign key explained][foreign_key_explained]), `nominal_classes` (needs `indicator`,
`analysis_method` and `method_tier`)

**Derived texture classes**: `nominal_classes.xlsx` holds the 32 soil texture classes (12 USDA,
12 IUSS, 4 major groups for each), all with analysis method `derived`. Their four indicators
(`soil texture usda`, `soil texture usda major group`, `soil texture iuss`,
`soil texture iuss major group`) have quantity `class`, and unit `class` is in `unit.xlsx`. The
classes are computed from clay, silt and sand while the lab observations load — see
[Derived soil texture classes][obs_texture].

**Shipped disabled** (commented out in the pilot file): `quantity_default_unit`. No
`classification_species` records are defined yet — the predefined `unknown` species and the
names copied down from genus (see below) are all there is. A few `# LEFT TODO` placeholders mark
catalogues with no Excel file yet (image, spectroscopy method, analysis method translate).

**Classification hierarchy note**: when you add a `classification_order` (e.g. `soil`), the same
name is automatically copied down to family/genus/species so foreign keys further down the
hierarchy always resolve even before you've defined anything more specific — you can always widen
detail later without breaking existing links. Adding records for `classification_family` and `classification_genus` follow the same pattern of copying downwards in the hierarchy. This has the effect that all classifications are found in the species table - with foreign key links up to the parent level they belong to.

## 3. Insert observation utilities with inherit

**Notebook cell**: 1.3 Insert observation utilities with inheritance

**Job file**: `job_insert_observation_utility_inherit.json` → pilot
`observation/insert_observation_utility_inherit.txt`

| Table | Requires (from step 2) |
|---|---|
| `provision` | `apparatus`, `provider`, `method_tier` |
| `provision_indicator` | `provision`, `indicator`, `method_tier` |
| `provision_serial_nr` | `provision` |

`provision_indicator.xlsx` registers, per provision, which indicators it measures, with analysis
method and unit. Besides the measured lab indicators, `lucas wetlab 2009` and `lucas wetlab 2015`
each register the four derived texture indicators with analysis method `derived` and unit
`class`. Removing one of these rows switches that derived class off for the provision.

## 4. Insert landscape utility

**Notebook cell**: 1.4 Insert landscape utility

**Job file**: `job_insert_landscape_utility.json` → pilot `landscape/insert_landscape_utility.txt`

**Active by default** (3 inserts, in this order): `biogeo16` → `landuse`, `landcover`

`biogeo16` holds the EEA biogeographic region names that the BIOGEO16 observations in
[Load LUCAS 2009][load_lucas_2009] resolve against (`biogeo16.xlsx` has a `tag` column).

### Land cover and land use: one ranked table per scheme

Land cover and land use are ranked like a taxonomy: **order > family > genus**. Each scheme is one
table, `landscape_utility.landcover` and `landscape_utility.landuse`, with the columns
`parent_<scheme>_id`, `class_rank_id`, `name`, `alias`, `tag`, `display_name` and `abstract`. The
ranks themselves sit in the shared catalogue `landscape_utility.class_rank` (`order` = 1,
`family` = 2, `genus` = 3), which is created and filled by [Setup database][setup_db], before any
of these inserts. An observation references the class at the rank actually recorded: about half
of the LUCAS 2009 points are recorded at family or order rank, not genus.

**Excel**: `landscape/excel/landcover.xlsx` and `landuse.xlsx`, one row per class, ordered so that
a parent comes before its children. From `landcover.xlsx`:

| class_rank_id__class_rank_name | parent_landcover_id__landcover_alias | name | alias | tag | display_name |
|---|---|---|---|---|---|
| order | | cropland | b | cropland | Cropland |
| family | b | cultivated cropland | b10 | cultivated | Cultivated cropland |
| genus | b10 | common wheat | b11 | wheat | Common wheat |

- **Alias** is the LUCAS code (`b`, `b10`, `b11`; `u100`, `u110`, `u111` for land use). Alias and
  tag are unique across all ranks of a scheme.
- **Name** only needs to be unique under its parent: `water` and `wetland` are both an order and
  a family.
- **Parent**: an order has no parent (empty cell, inserted as NULL). A genus may sit directly
  under an order (`w00` under `w`).
- The parent is resolved through `utility.foreign_key`, searching `alias`, then `tag`; see
  [Foreign key explained][foreign_key_explained].

**Shipped disabled**, prefixed `### REMOVE TO RUN ###` in the pilot file — their Excel files exist
in `landscape/excel/` already, they're just not activated as the LUCAS 2009 data assembled for this project do not include these:

`crop_growth_stage`, `major_landform`, `slope_position`, `sky_conditions`, `ground_conditions`,
`soil_preparation`, `reference_soil_groups`, `soil_texture_classification_USDA`,
`soil_texture_classification_ISSS`

To activate one, open `landscape/insert_landscape_utility.txt` and delete the
`### REMOVE TO RUN ###` prefix from its line, then re-run this cell.

## Next step

Once all four cells (1.1–1.4) have run, proceed to [Insert dataset metadata][insert_dataset_meta] — several
of its fields (territory, license, spatial reference) resolve against tables inserted here.

[insert_utility]: {{ site.baseurl }}/lucas_2009/insert_utility/
[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[utility_inherit_auto]: {{ site.baseurl }}/lucas_2009/utility_inherit_auto_explained/
[foreign_key_explained]: {{ site.baseurl }}/lucas_2009/foreign_key_explained/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[setup_db]: https://xspatula.github.io/xspatula_core_docs/setup_db/
[obs_texture]: {{ site.baseurl }}/lucas_2009/observations_explained/#derived-soil-texture-classes
