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

## Known overlap — read before running the LUCAS 2009 pipeline

`xspatula_ai4sh/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py` (mirrored into
`xspatula_lucas`) **also** generates a campaign record named `lucas_eu_2009`, independently of
this Excel row — see [Prepare data][prepare_data]. The two don't agree:

- **This page's route** (`campaign.xlsx` via `insert_tabular_data`): one `lucas_eu_2009` row with
  **both** provisions in `provision_id__provision_name_array`.
- **The script's route** (`process_lab/campaign/manage_process/lucas_eu_2009_campaign.json`, run
  from `load_LUCAS.ipynb`): a `manage_campaign` call for the same name `lucas_eu_2009` with
  `provision_id__provision_name_array` set to **only** `lucas-wetlab-2009`.

If `manage_campaign` is UPDATE-capable and you run this Excel-driven insert *and then* the
script-generated `load_LUCAS.ipynb` campaign step against the same database, the second call may
overwrite the campaign's provision array down to a single value, silently dropping
`foss xds rca`. This hasn't been tested end-to-end — verify the resulting
`observation.campaign.provision_id__provision_name_array` after running both, and prefer running
this Excel-driven step as the source of truth for the campaign record if you hit that. Recorded
in `notes/xspatula_lucas.md` for a possible fix upstream.

## Next step

Proceed to [LUCAS 2009][lucas_2009] to load the campaign's sampling, sample, and observation data.

[Manage dataset]: /dataset_meta/manage_dataset/
[prepare_data]: /lucas_2009/prepare_data/
[lucas_2009]: /lucas_2009/
