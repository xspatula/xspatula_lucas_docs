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

**Output**: `./data/lucas_400-2500_10/data-foss xds rapid content analyzer_400-2500_10.parquet`
(the array data) and a matching `.json` (query parameters and a `preprocessing_chain` field — empty
at this point, since nothing has been done to the data yet). This becomes the starting dataframe
every step in [ML preprocessing][ml_preprocess] chains from. The directory name
(`lucas_400-2500_10`) and the file's base name follow `<provision>_<begin>-<end>_<bandwidth>` —
see [ML preprocessing][ml_preprocess] for how later steps extend it.

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
