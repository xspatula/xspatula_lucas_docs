---
title: "Inspect Dataset"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/inspect_dataset/
author_profile: false
---

A quick look at the structure and contents of any `.parquet` file produced along the machine
learning pipeline — column names, units, a sample of the data, and the dataframe's size — without
opening a plotting tool or writing throwaway code.

## Notebook

**Path**: `xspatula_lucas/lucas/project_lucas_2009/inspect_dataset.ipynb`

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default), then run the
inspect cell.

## Inspect the dataset

**Process file**: `project_lucas_2009/process/pandas/inspect_pandas_df.json`

```json
{
  "process": [
    {
      "process": "inspect_pandas_dataset",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/lucas_400-2500_10",
        "parquet_file": "data-foss xds rapid content analyzer_400-2500_10.parquet",
        "column_array": "cec,clay,silt,sand,c-org,n-tot,ph-water",
        "max_rows": 10
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | Root directory of the project data (same value used throughout [Explore & select data][explore_select_data] and [ML preprocessing][ml_preprocess]) |
| `parquet_file` | The `.parquet` file to inspect, relative to `project_root_fp` — required |
| `column_array` | CSV list of columns to show; leave empty (or omit) to show every column |
| `columns_units_data` | Show column names, units, and the data (default `true`) |
| `columns_data` | Show column names and the data, without units |
| `columns_only` | Show only the column names |
| `columns_units_only` | Show only the column names and units |
| `max_rows` | Maximum number of rows to print; `0` lets Pandas pick automatically (default) |
| `show_size` | Print the dataframe's full row/column count alongside the sample (default `true`) |

Only set one of `columns_units_data` / `columns_data` / `columns_only` / `columns_units_only` at a
time — they're different views of the same inspection, not steps to combine.

**Example output** (`column_array` above, `max_rows: 10`, against the raw selection from
[Explore & select data][explore_select_data]):

```
column           clay           silt           sand   c-org          n-tot  \
unit   weight percent weight percent weight percent g*kg^-1 weight percent
0                 7.0           45.0           48.0    91.1            5.3
1                13.0           27.0           60.0    21.4            2.1
...

column ph-water
unit
0          4.00
1          6.53
...

    display columns: 6  display rows: 10
    dataframe columns: 224  dataframe rows: 25
```

The `unit` row comes straight from the dataframe's column metadata — the same units used
throughout [Observations explained][observations_explained]. `display columns`/`display rows`
report what was actually printed (narrowed by `column_array` and `max_rows`); `dataframe
columns`/`dataframe rows` report the full underlying size regardless of what was printed.

If a name in `column_array` isn't a column on the dataframe, it's skipped with a warning and the
full list of available columns is printed instead — useful for finding the exact indicator or
spectral band name to inspect next.

[explore_select_data]: /lucas_2009/machine_learning/explore_select_data/
[ml_preprocess]: /lucas_2009/machine_learning/ml_preprocess/
[observations_explained]: /lucas_2009/observations_explained/
