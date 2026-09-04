---
title: "Manage Dataset"
layout: single
sidebar:
  nav: "dataset_meta"
permalink: /dataset_meta/manage_dataset/
author_profile: false
---

The dataset record is the umbrella for all LUCAS topsoil data, spanning every survey round. One
row covers the whole dataset; individual survey rounds (2009, 2015, ...) are separate
[campaign][Manage campaign] records underneath it.

## Prerequisites

- [Manage data source] must be complete.

## Notebook cell

In `insert_lucas_dataset_meta.ipynb`, the **Insert dataset meta** cell runs:

```python
process_file = 'import_data/dataset_meta/insert_process/dataset.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/insert_process/dataset.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_dataset",
        "tabular_data_path": "import_data/dataset_meta/excel/dataset.xlsx",
        "dst_path": "import_data/dataset_meta/insert_process/staging"
      }
    }
  ]
}
```

## Source data

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/dataset.xlsx` — one row:

| Column | Value |
|---|---|
| `name` | `land use and coverage area frame survey (lucas) topsoil data` |
| `alias` | `lucas` |
| `data_source_id__data_source_name` | `esdac-jrc` (FK to [data source][Manage data source]) |
| `begun_at` / `ended_at` | `20090101` / `20221231` — spans every survey round shipped so far |
| `species_id__species_name` | `soil` |
| `license_id__license_name` | `jrc-lucas` |
| `territory_id__territory_name` | `EU` |
| `spatial_reference_id__spatial_reference_name` | `geographic` |

`alias` (`lucas`) is the value referenced by `dataset_id__dataset_name` in `campaign.xlsx`.

## Next step

Proceed to [Manage campaign].

[Manage data source]: /dataset_meta/manage_data_source/
[Manage campaign]: /dataset_meta/manage_campaign/
