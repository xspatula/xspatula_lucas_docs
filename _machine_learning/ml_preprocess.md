---
title: "ML Preprocessing"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/ml_preprocess/
author_profile: false
---

Cleans and transforms the dataset [Select data][explore_select_data] pulled out of the
database and [compiled][compile] — outlier removal, scaling, scatter correction, derivatives, decomposition, filtering,
band agglomeration, and feature selection — each step saving a new, separately named dataframe
rather than overwriting the last.

## How output is saved and chained

**Path**: `xspatula_lucas/lucas/project_lucas_2009/ml_preprocess.ipynb`

Every preprocessing step reads one dataframe and writes a new one, as a `.parquet` file (the
array data) plus a companion `.json` file (the query/step parameters and a `preprocessing_chain`
log of every step applied so far, in order). Nothing is ever overwritten in place — each step
appends an abbreviation to the previous file's base name:

| Suffix | Step | Meaning |
|---|---|---|
| `_ol` | Outlier detection | Rows flagged as outliers removed |
| `_mc` / `_as` / `_ps` / `_poi` | Scaling | Meancentring / autoscaling / paretoscaling / poissonscaling |
| `_snv`, `_msc`, `_l1`, `_l2`, `_max` (chainable, e.g. `_snvmsc`) | Scatter correction | Which scaler(s) applied |
| `_d<N>` (or `_d<N>app` if appended rather than replacing) | Derivative | Order-`N` spectral derivative |
| `_pca<N>` | Decomposition | PCA with `N` components |
| `_sg` / `_ma` / `_gf` / `_lw` | Filtering | Savitzky-Golay / moving-average / gauss / lowess |
| `_wc` | Agglomeration | Ward-clustered bands |
| `_vt` | Selection | Variance-threshold selected |

Derived files keep the stem of the dataset plus the step suffixes (`DERIVED_STEM_RE` in
`src/ai4sh/parquet_units.py` recognises them, so a derived file is never mistaken for the
selection itself). So starting from `lucas_2009_10nm.parquet` (the compiled dataset from
[Compile the project dataset][compile], in `data/compiled/lucas_2009_10nm/`), meancentring produces `..._mc.parquet`, a
first derivative on that produces `..._mc_d1.parquet`, and a 5-component PCA on that produces
`..._mc_d1_pca5.parquet` — each a real file on disk, all three keepable side by side.

### The hidden `.previous_dataframe` file

Rather than typing out the exact chained filename in every process file, set
`"dataframe": "previous"` and the framework resolves it from `.previous_dataframe`, a small JSON
file at the project's data root (`data/compiled/lucas_2009_10nm/.previous_dataframe`). Every step
appends a record to its `history` after it runs; the last record is the previous dataframe. Compile
registers the compiled dataset there too, so right after compiling, "previous" is the compiled
dataset. The default `"dataframe": "raw"` reads the latest selected (not derived) file. See
[The previous dataframe][previous_dataframe] for the file format. The last record looks like:

```json
{
  "process": "spectra_decomposition",
  "created_at": "2026-09-05T14:07:10",
  "entries": [
    {
      "dataframe": "lucas_2009_10nm_mc_d1_pca5.parquet",
      "indicator": null,
      "regressor": null,
      "selector": null
    }
  ]
}
```

Each record also holds the **provenance** of the step: the process parameters, every settings
set or file it read (with version and checksum), the software versions and the git commit. A
[registered model][ml_predict] collects the parameter sets of all steps behind its input from these
records.

Settings parameters (`chemometrics`, `filter_fp`, `multi_filter_fp`, `model_parameters_fp`,
`colormap`) take `none`, `default`, the name of a parameter set or a JSON file path; see
[ML settings][ml_settings].

The framework prints the resolved filename and asks you to confirm it interactively before
proceeding — useful as a sanity check that you're chaining from the step you think you are. This
is convenient for a linear chain but easy to point at the wrong thing if you've been branching
(e.g. comparing `_mc_snv` against `_mc_d1` and then continuing from "previous") — when in doubt,
name the exact dataframe explicitly instead of using `"previous"`.

## Available algorithms

Checked directly against `src/ai4sh/machine_learning_preprocess.py`, `chemometrics.py`, and
`filter.py`:

| Category | Process | Options |
|---|---|---|
| Outlier detection | `detect_outliers` | `iforest` (Isolation Forest), `ee` (Elliptic Envelope), `lof` (Local Outlier Factor) |
| Scaling | `spectra_scaling` | `meancentring`, `autoscaling`, `paretoscaling`, `poissonscaling` |
| Scatter correction | `spectra_scatter_correction` | `snv`, `msc`, `l1`, `l2`, `max` — pass a list to chain more than one |
| Derivative | `spectra_derivative` | any integer `derivative` order; `append: true` keeps the raw columns alongside instead of replacing them |
| Decomposition | `spectra_decomposition` | `pca` (only method currently implemented), `n_components` configurable |
| Filtering | `filter_spectra` | `moving-average`, `gauss`, `savitzky-golay`, `lowess` |
| Agglomeration | `ward_clustering` | Ward hierarchical clustering of spectral bands into fewer, wider bands |
| Selection | `select_variance_threshold` | scikit-learn `VarianceThreshold` on a chosen `scaler` (`minmax`, `standard`, or `robust`) |
| Selection | `spectra_indicator_univariate_selection` | scikit-learn `SelectKBest` with `f_regression` |
| Selection | `spectra_indicator_permutation_selection` | `permutation`, `rfe` (recursive feature elimination), and tree-based importance — each ranks bands using a chosen regressor (see [ML modeling][ml_model] for the regressor list) |

## Detect indicator outliers

Flags and removes outlier rows for the chosen indicators.

**Process file**: `project_lucas_2009/process/ml_preprocess/outlier_detection.json`

```json
{
  "process": [
    {
      "process": "detect_outliers",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "indicator_array": ["sand", "c-org"],
        "detector": "iforest",
        "threshold": 0.1
      }
    }
  ]
}
```

## Meancentring

"Always good to start with meancentring" — two chained sub-steps in one process file: scaling,
then scatter correction reading `"dataframe": "previous"` (the just-written meancentred output).

**Process file**: `project_lucas_2009/process/ml_preprocess/meancentring.json`

```json
{
  "process": [
    {
      "process": "spectra_scaling",
      "parameters": { "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm" }
    },
    {
      "process": "spectra_scatter_correction",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "previous",
        "scaler": ["snv"]
      }
    }
  ]
}
```

## Select spectral bands from variance

Drops low-variance bands to reduce overfitting risk, saving a new dataframe.

**Process file**: `project_lucas_2009/process/ml_preprocess/variance_threshold_selection.json`

```json
{
  "process": [
    {
      "process": "select_variance_threshold",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "lucas_2009_10nm_ol",
        "scaler": "minmax",
        "threshold": 0.035
      }
    }
  ]
}
```

## Chemometric scatter correction

**Process file**: `project_lucas_2009/process/ml_preprocess/chemometric_scatter_correction.json`

```json
{
  "process": [
    {
      "process": "spectra_scatter_correction",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "raw",
        "scaler": ["snv", "snv"]
      }
    }
  ]
}
```

**Fix needed**: the notebook cell's `process_file` currently reads
`project_neospectra/process/chemometric_scatter_correction.json` — that directory doesn't exist
in this repo. Shown above pointing at the matching file that does exist,
`project_lucas_2009/process/ml_preprocess/chemometric_scatter_correction.json`. See `notes/xspatula_lucas.md`.

## Chemometric derivatives on scatter correction

**Process file**: `project_lucas_2009/process/ml_preprocess/chemometric_derivatives.json`

```json
{
  "process": [
    {
      "process": "spectra_derivative",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "lucas_2009_10nm_mc",
        "derivative": 1
      }
    }
  ]
}
```

## Chemometric decomposition on derivatives

**Process file**: `project_lucas_2009/process/ml_preprocess/chemometric_decomposition.json`

```json
{
  "process": [
    {
      "process": "spectra_decomposition",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "lucas_2009_10nm_mc_d1",
        "method": "pca",
        "n_components": 5
      }
    }
  ]
}
```

## Chemometric chain

Runs scaling, scatter correction, and decomposition as one combined step instead of three
separate cells — a shortcut for the common case, using the same defaults as running them one at a
time.

**Process file**: `project_lucas_2009/process/ml_preprocess/chemometric_default_chain.json`

```json
{
  "process": [
    {
      "process": "chemometric_default",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "raw",
        "chemometrics": "default",
        "chemometrics_array": ["meancentring", "scatter_correction", "decomposition"]
      }
    }
  ]
}
```

**Fix needed**: same as the scatter correction step above — the notebook cell currently points at
`project_neospectra/process/chemometric_default_chain.json`; shown above pointing at
`project_lucas_2009/process/ml_preprocess/chemometric_default_chain.json`, which exists with correct content.

## Filter raw data

Smooths the raw spectra with a configurable filter before further processing.

**Process file**: `project_lucas_2009/process/ml_preprocess/filter_spectra.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "raw",
        "filter_fp": "./project_lucas_2009/filter/savitzky-golay_neospectra.json"
      }
    }
  ]
}
```

**Fix needed**: notebook cell points at `project_neospectra/process/filter_spectra.json`; shown
above pointing at `project_lucas_2009/process/ml_preprocess/filter_spectra.json`, which exists with correct
content. The `filter_fp` value itself, `savitzky-golay_neospectra.json`, is just an oddly-named
filename left over from the same AI4SH template — it does exist at that path and works fine, no
fix needed there.

## Agglomerate spectra using Ward clustering

Groups adjacent spectral bands into fewer, wider bands.

**Process file**: `project_lucas_2009/process/ml_preprocess/agglomerate_spectra.json`

```json
{
  "process": [
    {
      "process": "ward_clustering",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "lucas_2009_10nm_sg",
        "ward_clustering_fp": "./project_lucas_2009/agglomeration/ward_clustering_neospectra.json"
      }
    }
  ]
}
```

**Fix needed**: the notebook cell points at
`project_neospectra/process/agglomerate_spectra.json` (shown above corrected to
`project_lucas_2009/...`). `ward_clustering_fp` itself is fine as shipped — same as the
filter config above, just an oddly-named file that does exist.

## Autoscale the Ward-clustered output

**Process file**: `project_lucas_2009/process/ml_preprocess/autoscale_agglomeration_output.json`

```json
{
  "process": [
    {
      "process": "spectra_scaling",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "lucas_2009_10nm_sg_wc",
        "method": "meancentring"
      }
    }
  ]
}
```

**Fix needed**: same as the agglomeration step — the notebook cell points at
`project_neospectra/`; shown above corrected to `project_lucas_2009/`.

## Select best bands using univariate selection

Ranks bands per indicator using `SelectKBest`/`f_regression`.

**Process file**: `project_lucas_2009/process/ml_preprocess/univariate_selection.json`

```json
{
  "process": [
    {
      "process": "spectra_indicator_univariate_selection",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "indicator_array": ["c-org", "sand"],
        "separate_selections": true,
        "dataframe": "raw",
        "univariate_selector_fp": "./project_lucas_2009/spectral_band_selector/univariate_selector_neospectra.json"
      }
    }
  ]
}
```

`test_size` (default 0.3) holds out the same test samples as [ML modeling][ml_model] (from a hash of
the `sample_id`) and selects the bands from the training samples only. Use the same `test_size` in
the selection and the modeling, otherwise the model's test metrics include samples the selection has
seen. The selected bands and the holdout are recorded in the companion JSON.

**Fix needed**: notebook cell points at `project_neospectra/process/univariate_selection.json`;
shown above corrected to `project_lucas_2009/...`. `univariate_selector_fp` itself is fine as
shipped.

## Select best bands from permutation, RFE, or decision tree

Ranks bands using a chosen regressor as the underlying estimator — one or more of `permutation`,
`rfe`, or tree-based importance.

**Process file**: `project_lucas_2009/process/ml_preprocess/permutation_selector.json`

```json
{
  "process": [
    {
      "process": "spectra_indicator_permutation_selection",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "dataframe": "raw",
        "indicator_array": ["c-org", "sand"],
        "selector_array": ["permutation", "rfe"],
        "regressor_array": ["knn"],
        "model_parameters_fp": "default",
        "separate_selections": true,
        "n_top_features_in_plot": 16
      }
    }
  ]
}
```

`test_size` works as for univariate selection above. `model_parameters_fp: "default"` uses the
`standard` regressor set; the selector settings come from the `standard` selector set.

**Fix needed**: notebook cell points at `project_neospectra/process/permutation_selector.json`;
shown above corrected to `project_lucas_2009/...`.

## Next step

Once you have a dataframe you're happy with, proceed to [ML modeling][ml_model] to train and
evaluate regressors against it.

[explore_select_data]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/
[compile]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#compile-the-project-dataset
[previous_dataframe]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#the-previous-dataframe-previous_dataframe
[ml_model]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_model/
[ml_settings]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_settings/
[ml_predict]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_predict/
