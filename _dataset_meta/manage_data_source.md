---
title: "Manage Data Source"
layout: single
sidebar:
  nav: "dataset_meta"
permalink: /dataset_meta/manage_data_source/
author_profile: false
---

The data source record identifies who published the data — for LUCAS, the European Soil Data
Centre (ESDAC), part of the European Commission's Joint Research Centre.

## Notebook cell

In `insert_lucas_dataset_meta.ipynb`, the **Insert data source** cell runs:

```python
process_file = 'import_data/dataset_meta/insert_process/data_source.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/insert_process/data_source.json`

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

`insert_tabular_data` translates the Excel row(s) to JSON and immediately applies them via
`manage_data_source` — translate and insert in one call.

## Source data

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

## Next step

Proceed to [Manage person].

[Manage person]: /dataset_meta/manage_person/
