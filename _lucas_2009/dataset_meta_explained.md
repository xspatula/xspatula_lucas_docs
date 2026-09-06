---
title: "Dataset Metadata Explained"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/dataset_meta_explained/
author_profile: false
---

Reference page — not required reading to complete the [Insert dataset metadata][insert_dataset_meta]
step, only if you want the detail behind the data source, person, dataset, and campaign records
it inserts.

Dataset metadata describes the provenance of the LUCAS soil observations: who published them, who
the named contact is, and under what campaign they were collected. This chain of records must be
in place before any sample or observation data can be entered — and before it, the
[utility catalogues][utility_explained] must already exist, since these records reference several
of them by foreign key (territory, license, spatial reference).

## The hierarchy

```
data_source              (ESDAC-JRC, the publisher)
    └── dataset          (LUCAS topsoil data — one row per LUCAS survey round)
          └── campaign   (one bounded round, e.g. lucas_eu_2009)
```

Persons are registered independently, linked to a data source, and referenced from other tables
by contact name/email.

| Table | Description |
|---|---|
| `observation.data_source` | The organisation providing the data — here, ESDAC-JRC |
| `observation.person` | Named contacts responsible for the data |
| `observation.dataset` | Top-level grouping — the LUCAS topsoil dataset as a whole |
| `observation.campaign` | A single bounded LUCAS survey round (2009, 2015, ...) |

All four are inserted in one step each via `insert_tabular_data` — translate and insert in a
single call, no separate translate/manage split. Every process file follows the same shape, e.g.
data source's:

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_data_source",
        "tabular_data_path": "import_data/dataset_meta/excel/data_source.xlsx",
        "dst_path": "import_data/dataset_meta/insert_process/staging"
      }
    }
  ]
}
```

## Data source

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/data_source.xlsx` — one row:

| Column | Value |
|---|---|
| `name` | `european soil data centre (esdac), european commission, joint research centre jrc)` |
| `alias` | `esdac-jrc` |
| `display_name` | `ESDAC-JRC` |
| `url` | `https://esdac.jrc.ec.europa.eu` |
| `territory_id__territory_name` | `eu` |
| `contact_name` | `ESDAC - European Commission` |
| `contact_email` | `ec-esdac@jrc.ec.europa.eu` |

`alias` (`esdac-jrc`) is the value later referenced by `data_source_id__data_source_name` foreign
keys in `person.xlsx` and `dataset.xlsx`.

## Person

LUCAS ships one placeholder contact tied to ESDAC-JRC — edit the Excel row before running this
step for a production import.

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/person.xlsx`

| Column | Value (shipped placeholder) |
|---|---|
| `data_source_id__data_source_name` | `esdac-jrc` (FK to data source, above) |
| `first_name` / `middle_name` / `last_name` | `fn` / `mn` / `ln` — placeholder, edit before real use |
| `email` | `ec-esdac@jrc.ec.europa.eu` |
| `territory_id__territory_name` | `eu` |

## Dataset

The dataset record is the umbrella for all LUCAS topsoil data, spanning every survey round. One
row covers the whole dataset; individual survey rounds (2009, 2015, ...) are separate campaign
records underneath it.

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/dataset.xlsx` — one row:

| Column | Value |
|---|---|
| `name` | `land use and coverage area frame survey (lucas) topsoil data` |
| `alias` | `lucas` |
| `data_source_id__data_source_name` | `esdac-jrc` (FK to data source, above) |
| `begun_at` / `ended_at` | `20090101` / `20221231` — spans every survey round shipped so far |
| `species_id__species_name` | `soil` |
| `license_id__license_name` | `jrc-lucas` |
| `territory_id__territory_name` | `EU` |
| `spatial_reference_id__spatial_reference_name` | `geographic` |

`alias` (`lucas`) is the value referenced by `dataset_id__dataset_name` in `campaign.xlsx`.

## Campaign

A campaign is one bounded LUCAS survey round — `lucas_eu_2009`, `lucas_eu_2015`, and so on. Each
row can carry more than one provision (lab and spectrometer) in a single array field. This is the
campaign's single source of truth — `lucas_2009_to_xspatula.py` (used in
[Prepare data][prepare_data]) only generates the sampling log, geolocation, sample, and
observation records for a campaign that must already exist by the time
[Load LUCAS 2009][load_lucas_2009] runs.

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/campaign.xlsx` — one row per survey
round, e.g. `lucas_eu_2009`:

| Column | Value |
|---|---|
| `dataset_id__dataset_name` | `lucas` (FK to dataset, above) |
| `name` | `lucas_eu_2009` |
| `display_name` | `LUCAS 2009` |
| `contact_name` / `contact_email` | `inherit` — pulled from the parent dataset record, see [Utility → inherit and auto][utility_inherit_auto] |
| `begun_at` / `ended_at` | `20090501` / `20091031` |
| `territory_id__territory_name` / `site` | `EU` / `Europe` |
| `location_method_id__location_method_name` | `gps` |
| `location_error` / `location_error_unit_id__unit_name` | `1000` / `m` |
| `provision_id__provision_name_array` | `foss xds rca,lucas-wetlab-2009` — **both** provisions, one array |

[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[utility_explained]: /lucas_2009/utility_explained/
[utility_inherit_auto]: /lucas_2009/utility_inherit_auto_explained/
[prepare_data]: /lucas_2009/prepare_data/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
