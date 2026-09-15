---
title: "Explore & Select Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/explore_select_data/
author_profile: false
---

The first step of the machine learning pipeline: pull a working subset of the loaded LUCAS 2009
campaign out of the database and onto local disk, then explore it to determine if and what preprocessing steps to apply.

## Notebook

**Path**: `xspatula_lucas/lucas/project_lucas_2009/explore_select_data.ipynb`

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default). The seven
cells below can be run in any order after the first two (find indicators, then select data) —
the plotting cells are independent of each other and only need the selected dataset to exist.

## Find the available indicators

Prints the spectral provisions and indicators available for a dataset/campaign — a discovery
step, nothing is saved to disk.

**Process file**: `project_lucas_2009/process/select_indicator.json`

```json
{
  "process": [
    {
      "process": "select_indicator",
      "overwrite": false,
      "parameters": {
        "dataset_name": "lucas",
        "campaign_name": ""
      }
    }
  ]
}
```

Leave `campaign_name` empty to see everything registered under the `lucas` dataset across all
campaigns, or set it to scope to one campaign (e.g. `lucas_eu_2009`).

## Select spectra and analysis data

Queries the database, resamples the spectra to a target wavelength grid, optionally converts to
absorbance, and writes the result to local disk as a `.parquet` file plus a companion `.json`
metadata file. Skips silently if the output already exists, unless `overwrite: true`.

**Process file**: `project_lucas_2009/process/select_lucas_2009_10nm.json`

```json
{
  "process": [
    {
      "process": "select_spectra",
      "overwrite": false,
      "parameters": {
        "dataset_name": "lucas",
        "campaign_name": "",
        "provision_name": "foss xds rapid content analyzer",
        "preparation_name": "ds2",
        "begin_wavelength": 400,
        "end_wavelength": 2500,
        "output_bandwidth": 10,
        "min_profile": 0,
        "max_profile": 20,
        "indicator_array": ["cec", "clay", "silt", "sand", "c-org", "n-tot", "ph-water"],
        "data_range": "",
        "as_absorbance": true,
        "output_root_fp": "./data"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `dataset_name` / `campaign_name` | Scope the query — same as above |
| `provision_name` | Spectrometer provision to pull spectra from |
| `preparation_name` | Sample preparation method (`ds2`, matching the observation log) |
| `begin_wavelength` / `end_wavelength` / `output_bandwidth` | Target wavelength grid — resampled from the spectrometer's native `wavelength_array` (see [Observations explained][observations_explained]), not assumed to match it |
| `min_profile` / `max_profile` | Depth profile filter, cm |
| `indicator_array` | Which lab indicators to pull alongside the spectra. Xspatula resolves each entry by either its indicator name or its alias — `ph-water` here and `ph-h2o` in the raw observation records (see [Observations explained][observations_explained]) are the same indicator |
| `as_absorbance` | Convert stored diffuse reflectance to absorbance on the way out |
| `output_root_fp` | Local directory the `.parquet`/`.json` pair is written under |
| `targetfeaturesymbols` | Which `unit` to save each indicator in — see [Indicator units](#indicator-units) below. Optional, defaults to `"default"` |

**Output**: `./data/lucas_400-2500_10/data-foss xds rapid content analyzer_400-2500_10.parquet`
(the array data) and a matching `.json` (query parameters and a `preprocessing_chain` field — empty
at this point, since nothing has been done to the data yet). This becomes the starting dataframe
every step in [ML preprocessing][ml_preprocess] chains from. The directory name
(`lucas_400-2500_10`) and the file's base name follow `<provision>_<begin>-<end>_<bandwidth>` —
see [ML preprocessing][ml_preprocess] for how later steps extend it.

## Indicator units

Different provisions and campaigns can record the same indicator in different units — `select_spectra`
harmonises them at extraction time, so every value in the saved `.parquet` ends up in one consistent
unit per indicator.

**Where the target unit comes from**: `lucas/default/plot/targetfeaturesymbols.json`, keyed by
indicator name, e.g.:

```json
"c-org": {
  "color": "dimgray",
  "alpha": 0.2,
  "label": "Organic carbon (C)",
  "unit": "native"
}
```

Point `targetfeaturesymbols` at a different file (a path, instead of `"default"`) to use your own
per-indicator unit settings — the same file also drives the axis units on the [plot](#plot-indicators)
below.

**Two ways to set an indicator's `unit`:**

- `"native"` — keep whatever unit the DB recorded for that indicator, no translation. Simplest
  option; use it when you don't need to combine this extraction with data recorded in a different
  unit. If the DB has mixed native units for one indicator across records, `select_spectra` prints a
  warning and uses the first one it finds as the saved unit label — the values themselves are left
  untouched, so check this isn't silently mixing units before trusting the result.
- A specific unit name (e.g. `"percent"`) — every record gets translated into that unit via
  `observation_utility.unit_translate`. That translation must already exist as a row in the table —
  see [Adding a new unit translation][foreign_key_explained] if it doesn't. If it's missing,
  `select_spectra` raises an error naming the exact `unit_translate.xlsx` row to add (source unit,
  destination unit) and which notebook to re-run afterwards (`insert_utility.ipynb`).

**Combining datasets later**: if you plan to merge two `select_spectra` extractions (e.g. two
different provisions or campaigns) into one dataframe, set the *same* target unit for every
indicator they share — g/Kg from one and percent from the other won't merge into anything usable.
Point both extractions at `targetfeaturesymbols` files that agree on unit for the shared indicators
(the same file works for both, or two files with matching `unit` values).

**Where it ends up**: the unit actually applied to each column is written to the output `.json`'s
`indicator_units` field, and embedded directly in the `.parquet` file as column metadata — see
[Inspect dataset][inspect_dataset] for how to view it (`columns_units_data` / `columns_units_only`).

## Inspect the dataset

Any `.parquet` file produced along this pipeline — not just the raw selection above, but any
dataframe from [ML preprocessing][ml_preprocess] too — can be inspected with the
`inspect_pandas_dataset` process: column names and units, a sample of the data, and the
dataframe's full size. Point a process file at it with `project_root_fp` and `parquet_file` set to
the file you want to look at, e.g.:

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

See [Inspect dataset][inspect_dataset] for the full parameter reference and an example run.

## Plot indicators

Boxplot and histogram for the chosen indicators — a quick look at their distributions before
modeling.

**Process file**: `project_lucas_2009/process/plot/plot_indicator.json`

```json
{
  "process": [
    {
      "process": "plot_indicators",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/lucas_400-2500_10",
        "indicator_array": ["cec", "clay", "sand", "c-org"],
        "targetfeaturesymbols": "default",
        "standardisation": "default",
        "transformation": "default",
        "boxplot": true,
        "histogram": true,
        "show": true,
        "save": true
      }
    }
  ]
}
```

Saves to `<project_root_fp>/plot/boxplot_<indicator>.png` and `histogram_<indicator>.png`.
`targetfeaturesymbols` is the same file used for [indicator units](#indicator-units) above — here it
only controls plot color/label/axis-unit, no translation happens on the plotted values.

## Plot spectra and scatter correction options

Overlays the raw spectra against each available scatter-correction scaler (SNV, MSC, ...) so you
can visually compare them before picking one for [ML preprocessing][ml_preprocess].

**Process file**: `project_lucas_2009/process/plot/plot_spectra_scattercorrection_scalers.json`

## Plot spectra and scaling method options

Same idea, for the scaling methods (meancentring, autoscaling, paretoscaling, poissonscaling).

**Process file**: `project_lucas_2009/process/plot/plot_spectra_scaling_methods.json`

## Plot spectra

A plain spectra plot of the selected dataset, no processing applied.

**Process file**: `project_lucas_2009/process/plot/plot_spectra.json`

## Plot filter spectra

Overlays the raw spectra against each available smoothing filter (moving-average, gauss,
Savitzky-Golay, lowess).

**Process file**: `project_lucas_2009/process/plot/plot_filter_spectra.json`

## Next step

Proceed to [ML preprocessing][ml_preprocess] to clean and transform the selected data.

[ml_preprocess]: /lucas_2009/machine_learning/ml_preprocess/
[observations_explained]: /lucas_2009/observations_explained/
[inspect_dataset]: /lucas_2009/machine_learning/inspect_dataset/
[foreign_key_explained]: /lucas_2009/foreign_key_explained/
