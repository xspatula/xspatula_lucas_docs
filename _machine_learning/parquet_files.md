---
title: "Parquet Files"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/parquet_files/
author_profile: false
---

Every [select process][select_data] saves its result locally as a Parquet file. This page explains what
is in it, how it is named, which unit each column is in, why some values are missing, and how to inspect
it and compile several selections into one project dataset.

## What a select process saves

Two files with the same name (no `data-`/`params-` prefix):

- `<name>.parquet`: the data, with the unit of each column embedded as column metadata.
- `<name>.json`: the companion file, recording how the selection was made.

The selection is also registered in the folder's
[.previous_dataframe](#the-previous-dataframe-previous_dataframe).

## Output names

Output paths are built from tags (`_Ml_output_paths` in `select.py`). The defaults:

```
{output_root_fp}/{role}/{provision tag}/{preparation alias}/{scope tag}/{begin}-{end}_{bandwidth}[__{filter}].parquet   spectra
{output_root_fp}/{role}/{scope tag}/{aliases}_{min}-{max}[__{filter}].parquet                                        indicator, covariate
{output_root_fp}/compiled/{output_name}/{output_name}.parquet                                                         compile
```

The **scope tag** is the most specific one given: sampling_log > campaign > dataset. The LUCAS 2009
files:

```
data/spectra/foss-xds-rca/ds2/lucas_eu_2009/400-2500_10.parquet
data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet
data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet
data/compiled/lucas_2009_10nm/lucas_2009_10nm.parquet
```

- An indicator or covariate list longer than 60 characters is cut and given an 8-character hash of the
  full list.
- A covariate rolled up to a rank gets the rank in the name (`landcover.order`).
- Set `output_sub_fp` and/or `output_name` to override the defaults.

**Filter part.** A [class filtered][class_filters] selection gets `__<filter part>` appended to the
default name: per filter its abbreviation (`tx`, `lc`, `lu`, `bg`, `jx`, `cp`, `px`) and entries,
lower-cased, every character outside `a-z0-9.` replaced by `-`, filters joined by `_`. Longer than 60
characters, it is cut and given an 8-character hash. Examples:

```
clay-c-org-ph-water_0-20__lc-cropland-grassland      indicators, land cover filter
400-2500_10__lc-cropland-grassland                   spectra, filter inherited via sample_source
```

`output_suffix` replaces the automatic filter part. With a given `output_name` there is no filter part,
but an `output_suffix` is still appended (`__<suffix>`). `none` leaves it out. Default `default`.

Unfiltered names are unchanged.

## Columns

**Key columns**, in every output: `sample_id` (the join key; the same LUCAS point exists in 2009 and
2015, so sample names are not unique across sampling logs), `sample_name`, `campaign_tag`,
`sampling_log_tag`, `latitude_dd`, `longitude_dd`, `profile_min`, `profile_max`.

**Class columns**: one `class_<filter>` column per class filter, e.g. `class_landcover`. The value is the
filter entry the sample matched (a wheat sample under `["cropland", "grassland"]` gets `cropland`).

**Data columns**: the spectral bands, indicators or covariates.

`campaign_tag` and `sampling_log_tag` are there for plotting and evaluating by campaign or survey round
(e.g. 2009 vs 2015) later. ML preprocessing and modelling treat key and `class_*` columns as metadata,
never as features.

## The companion JSON

Records the scope, the filters and the result. From a filtered indicator selection (shortened: the real
file also has the `*_name` fields, `covariate_array`, `covariate_rank` and `class_labels`):

```json
{
  "role": "indicator",
  "dataset_tag": "lucas",
  "sampling_log_tag": "lucas_eu_2009",
  "min_profile": 0,
  "max_profile": 20,
  "indicator_array": ["clay", "c-org", "ph-water"],
  "indicator_units": {"clay": "weight percent", "ph-water": "ph", "c-org": "g*kg^-1"},
  "data_range": {},
  "sample_filter": {"landcover": ["cropland", "grassland"]},
  "sample_source": "",
  "class_columns": ["class_landcover"],
  "output_name": "clay-c-org-ph-water_0-20__lc-cropland-grassland",
  "n_samples": 37
}
```

`sample_filter` is the record of how the file was made (with `texture_classification` added when texture
is filtered). A selection made with [`sample_source`][sample_source] inherits the source's
`sample_filter`.

A covariate selection also records `covariate_rank` and `class_labels` (tag → `name`, `display_name`,
`alias`, `rank`). From `data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.json`:

```json
"covariate_rank": {
  "landcover": "order",
  "landuse": "order",
  "biogeo16": "lowest"
},
"class_labels": {
  "landcover": {
    "cropland": {
      "name": "cropland",
      "display_name": "Cropland",
      "alias": "b",
      "rank": "order"
    },
    ...
  },
  "landuse": {
    "prim-prod": {
      "name": "primary production",
      "display_name": "Primary production",
      "alias": "u100",
      "rank": "order"
    },
    ...
  },
  "biogeo16": {
    "continental": {
      "name": "continental biogeographic region",
      "display_name": "Continental",
      "alias": "continental",
      "rank": null
    },
    ...
  }
},
"indicator_units": {
  "landcover": "class",
  "landuse": "class",
  "biogeo16": "class"
}
```

See [Covariates and ranks][covariates_and_ranks] for `covariate_rank`.

## Indicator units

Different provisions and campaigns can record the same indicator in different units. The select
processes harmonise them at extraction time, so every value in the saved `.parquet` ends up in one
consistent unit per indicator.

**Where the target unit comes from**: `lucas/default/plot/targetfeaturesymbols.json`, keyed by indicator
name, e.g.:

```json
"c-org": {
  "color": "dimgray",
  "alpha": 0.2,
  "label": "Organic carbon (C)",
  "unit": "native"
}
```

All indicators in the default file use `"unit": "native"`. Point `targetfeaturesymbols` at a different
file (a path, instead of `"default"`) to use your own per-indicator unit settings. The same file also
drives the axis units and colours of the [plots][plot_data].

**Two ways to set an indicator's `unit`:**

- `"native"`: keep whatever unit the DB recorded for that indicator, no translation. Simplest option; use
  it when you don't need to combine this extraction with data recorded in a different unit. If the DB has
  mixed native units for one indicator across records, the process prints a warning and uses the first
  one it finds as the saved unit label. The values themselves are left untouched, so check this isn't
  silently mixing units before trusting the result.
- A specific unit name (e.g. `"percent"`): every record gets translated into that unit via
  `observation_utility.unit_translate`. That translation must already exist as a row in the table; see
  [Adding a new unit translation][foreign_key_explained] if it doesn't. If it's missing, the process
  raises an error naming the exact `unit_translate.xlsx` row to add (source unit, destination unit). Add
  the row, then re-run cell 1.2 (Insert observation utilities) of `insert_lucas_data.ipynb`.

**Combining datasets later**: if you plan to compile selections from different provisions or sampling
logs (e.g. 2009 and 2015), set the *same* target unit for every indicator they share. g/kg from one and
percent from the other won't merge into anything usable.

**Where it ends up**: the unit applied to each column is written to the `.json`'s `indicator_units` and
embedded in the `.parquet` as column metadata (see [Inspect the dataset](#inspect-the-dataset),
`columns_units_data` / `columns_units_only`). Nominal columns (classes) get the unit `class`.

## Invalid values

Two safety nets keep invalid values out of the models. They apply to all datasets, and come on top of
the [LUCAS 2009 import cleaning][data_cleaning].

**Selection** (`_Screen_values` in `select.py`): every numeric indicator or covariate is checked after
unit translation. A value is set to missing if it is

- a missing-value code: `-9999`, `-999`, `-888` (`MISSING_VALUE_CODES`), or
- outside the physical range of its unit (`Valid_unit_range`): percent 0–100, permill 0–1000,
  `g*kg^-1` 0–1000, `mg*kg^-1` 0–1e6, pH 0–14, and no negative contents, concentrations,
  conductivities, lengths or areas.

Each removal is printed per indicator, e.g.
`⚠️  clay: 3 value(s) are missing codes or outside the valid range for weight percent, set to missing: [-999.0]`.

**Transformation** (`chemometrics.py`): log of values ≤ 0, square root of values < 0 and the reciprocal
of 0 give a missing value with a warning, instead of ±inf that crashes the plots.

## The previous dataframe (`.previous_dataframe`)

Every select process, `compile_dataset` and every ML preprocessing step appends a record to the hidden
file `.previous_dataframe` in the folder it saved to: process, time, note and file.

- The last record is **the previous dataframe**. Inspect reads it when `parquet_file` is empty, plot
  reads the latest selected file in it, and ML preprocessing reads it with `"dataframe": "previous"`.
  A notebook thus runs select → inspect → plot → preprocess without repeating file names.
- The earlier records are the folder's history, oldest first.
- Spectra, indicator and covariate folders each keep their own history.

Two selections in the same indicator folder:

```json
{
  "history": [
    {"process": "select_indicator_only", "created_at": "2026-10-19T08:49:03",
     "note": "indicator: 199 samples",
     "entries": [{"dataframe": "clay-c-org_0-20.parquet", "indicator": null, "regressor": null, "selector": null}]},
    {"process": "select_indicator_only", "created_at": "2026-10-19T08:49:04",
     "note": "indicator: 23 samples, filter {'landcover': ['cropland']}",
     "entries": [{"dataframe": "clay-c-org_0-20__lc-cropland.parquet", "indicator": null, "regressor": null, "selector": null}]}
  ]
}
```

A folder created before this history existed has none until a select runs there again. Inspect then says
`ERROR: no .previous_dataframe found in ...`.

[Plot data][plot_data] and ML preprocessing pick the latest *selected* (not derived) file from the
history (`Find_selected_parquet`). Without a history they fall back to the newest file by date and list
the others in a warning. Derived files (with step suffixes like `_ol`, `_mc`) are never candidates.
Delete stale files, e.g. old `data-*.parquet` files from before the tag-based names, to silence the
warning.

## Inspect the dataset

`inspect_pandas_dataset` shows any `.parquet` along the pipeline (a selection, the compiled dataset, or
any dataframe from [ML preprocessing][ml_preprocess]): column names and units, a sample of the data, and
the dataframe's full size.

**Process file**: `project_lucas_2009/process/inspect/inspect_latest_indicator_df.json`

```json
{
  "process": [
    {
      "process": "inspect_pandas_dataset",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/indicator/lucas_eu_2009",
        "parquet_file": "",
        "column_array": "",
        "max_rows": 10
      }
    }
  ]
}
```

An empty `parquet_file` (or `none`/`previous`) reads the previous dataframe and names it first:

```
    Previous: select_indicator_only — clay-c-org_0-20__lc-cropland.parquet (2026-10-19T08:49:04)
```

To inspect a specific file, name it. `inspect_dataset.ipynb` does this with
`project_lucas_2009/process/pandas/inspect_pandas_df.json`:

```json
{
  "process": [
    {
      "process": "inspect_pandas_dataset",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "parquet_file": "lucas_2009_10nm.parquet",
        "column_array": "cec,clay,silt,sand,c-org,n-tot,ph-water",
        "max_rows": 10
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | Folder of the parquet file |
| `parquet_file` | File relative to `project_root_fp`; empty, `none` or `previous` reads `.previous_dataframe` |
| `column_array` | CSV list of columns to show; leave empty (or omit) to show every column |
| `columns_units_data` | Show column names, units and the data (default `true`) |
| `columns_data` | Show column names and the data, without units |
| `columns_only` | Show only the column names |
| `columns_units_only` | Show only the column names and units |
| `max_rows` | Maximum number of rows to print; `0` lets Pandas pick automatically (default) |
| `show_size` | Print the dataframe's full row/column count alongside the sample (default `true`) |

Only set one of `columns_units_data` / `columns_data` / `columns_only` / `columns_units_only` at a time.
They are different views of the same inspection, not steps to combine.

**Example output** (the file above, against the compiled dataset):

```
column            cec           clay           silt           sand   c-org  \
unit      cmolc*kg^-1 weight percent weight percent weight percent g*kg^-1
0                12.3            7.0           45.0           48.0    91.1
1                 8.7           13.0           27.0           60.0    21.4
...

column    n-tot ph-water
unit    g*kg^-1       ph
0                5.3     4.00
1                2.1     6.53
...

    display columns: 7  display rows: 10
    dataframe columns: 224  dataframe rows: 25
```

The `unit` row comes from the column metadata: whatever [Indicator units](#indicator-units) resolved for
that column at selection time (here all `"native"`, so the units `lucas wetlab 2009` recorded them in;
see [Observations explained][observations_explained]). `display columns`/`display rows` report what was
printed; `dataframe columns`/`dataframe rows` the full size.

A name in `column_array` that isn't a column is skipped with a warning, and the available columns are
listed. That is useful for finding the exact indicator or band name.

## Compile the project dataset

Joins saved selections into one dataset. The compiled folder is the `project_root_fp` of all later ML
steps.

**Process file**: `project_lucas_2009/process/compile_lucas_2009_10nm.json`

```json
{
  "process": [
    {
      "process": "compile_dataset",
      "overwrite": false,
      "parameters": {
        "input_array": [
          "./project_lucas_2009/data/spectra/foss-xds-rca/ds2/lucas_eu_2009/400-2500_10.parquet",
          "./project_lucas_2009/data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet"
        ],
        "output_root_fp": "./project_lucas_2009/data",
        "output_name": "lucas_2009_10nm"
      }
    }
  ]
}
```

- `input_array` needs at least two files; `output_name` is required.
- The inputs are **inner joined** on `sample_id`: only samples present in all inputs are kept.
- Key columns are kept once. `class_*` columns repeated across inputs are kept once. Any other column
  found in two inputs is an error.
- Units, `class_columns`, `covariate_rank` and `class_labels` are merged, and the companion JSON lists
  each input with its role and sample count.
- Compile registers its output in `.previous_dataframe`.

To add the covariates, append
`./project_lucas_2009/data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet`
to `input_array`.

**Output**: `data/compiled/lucas_2009_10nm/lucas_2009_10nm.parquet` (+ `.json`).

## Next step

[Plot data][plot_data]: look at the selection before preprocessing.

[select_data]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/
[class_filters]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/#class-filters
[sample_source]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/#reuse-a-selections-samples-sample_source
[covariates_and_ranks]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/#covariates-and-ranks
[plot_data]: {{ site.baseurl }}/lucas_2009/machine_learning/plot_data/
[ml_preprocess]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_preprocess/
[observations_explained]: {{ site.baseurl }}/lucas_2009/observations_explained/
[foreign_key_explained]: {{ site.baseurl }}/lucas_2009/foreign_key_explained/#adding-a-new-unit-translation
[data_cleaning]: {{ site.baseurl }}/lucas_2009/prepare_data/#data-cleaning
