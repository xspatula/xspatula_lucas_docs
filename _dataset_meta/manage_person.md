---
title: "Manage Person"
layout: single
sidebar:
  nav: "dataset_meta"
permalink: /dataset_meta/manage_person/
author_profile: false
---

Person records identify named contacts linked to a data source. LUCAS ships one placeholder
contact tied to ESDAC-JRC — edit the Excel row before running this step for a production import.

## Prerequisites

- [Manage data source] must be complete.

## Notebook cell

In `insert_lucas_dataset_meta.ipynb`, the **Insert persons** cell runs:

```python
process_file = 'import_data/dataset_meta/insert_process/person.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/insert_process/person.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_person",
        "tabular_data_path": "import_data/dataset_meta/excel/person.xlsx",
        "dst_path": "import_data/dataset_meta/insert_process/staging"
      }
    }
  ]
}
```

## Source data

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/person.xlsx`

| Column | Value (shipped placeholder) |
|---|---|
| `data_source_id__data_source_name` | `esdac-jrc` (FK to [data source][Manage data source]) |
| `first_name` / `middle_name` / `last_name` | `fn` / `mn` / `ln` — placeholder, edit before real use |
| `email` | `ec-esdac@jrc.ec.europa.eu` |
| `territory_id__territory_name` | `eu` |

## Next step

Proceed to [Manage dataset].

[Manage data source]: /dataset_meta/manage_data_source/
[Manage dataset]: /dataset_meta/manage_dataset/
