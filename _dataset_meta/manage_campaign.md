---
title: "Manage Campaign"
layout: single
sidebar:
  nav: "dataset_meta"
permalink: /dataset_meta/manage_campaign/
author_profile: false
---

A campaign is one bounded LUCAS survey round — `lucas_eu_2009`, `lucas_eu_2015`, and so on. Each
row in `campaign.xlsx` can carry more than one provision (lab and spectrometer) in a single array
field.

## Prerequisites

- [Manage dataset] must be complete.

## Notebook cell

In `insert_lucas_dataset_meta.ipynb`, the **Insert campaigns** cell runs:

```python
process_file = 'import_data/dataset_meta/insert_process/campaign.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/insert_process/campaign.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_campaign",
        "tabular_data_path": "import_data/dataset_meta/excel/campaign.xlsx",
        "dst_path": "import_data/dataset_meta/insert_process/staging"
      }
    }
  ]
}
```

## Source data

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/campaign.xlsx` — one row per survey
round, e.g.:

| Column | `lucas_eu_2009` value |
|---|---|
| `dataset_id__dataset_name` | `lucas` (FK to [dataset][Manage dataset]) |
| `name` | `lucas_eu_2009` |
| `display_name` | `LUCAS 2009` |
| `begun_at` / `ended_at` | `20090501` / `20091031` |
| `territory_id__territory_name` / `site` | `EU` / `Europe` |
| `location_method_id__location_method_name` | `gps` |
| `location_error` / `location_error_unit_id__unit_name` | `1000` / `m` |
| `provision_id__provision_name_array` | `foss xds rca,lucas-wetlab-2009` — **both** provisions, one array |

This is the campaign's single source of truth — `lucas_2009_to_xspatula.py` (used in
[Prepare data][prepare_data]) only generates the sampling log, geolocation, sample, and
observation records for a campaign that must already exist by the time
[Load LUCAS 2009][load_lucas_2009] runs.

## Next step

Proceed to [Load LUCAS 2009][load_lucas_2009] to load the campaign's sampling, sample, and
observation data.

[Manage dataset]: /dataset_meta/manage_dataset/
[prepare_data]: /lucas_2009/prepare_data/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
