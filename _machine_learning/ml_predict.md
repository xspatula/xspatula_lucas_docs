---
title: "Predict and Register"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/ml_predict/
author_profile: false
---

A model saved by [ML modeling][ml_model] is a complete pipeline: give it unprocessed spectra and it
returns the indicator in its native unit. This page covers applying models to new spectra
(`predict_independent`) and keeping the final models in the database registry (`register_model`,
`retire_model`).

## Notebook

**Path**: `xspatula_lucas/lucas/project_lucas_2009/ml_predict.ipynb`

## Predict independent spectra

**Process file**: `project_lucas_2009/process/predict/predict_independent.json`

```json
{
  "process": [
    {
      "process": "predict_independent",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_wetlab_foss-xds_10nm",
        "dataframe": "lucas_2009_wetlab_foss-xds_10nm",
        "model_array": [
          "./project_lucas_2009/data/compiled/lucas_2009_wetlab_foss-xds_10nm/models/lucas_2009_wetlab_foss-xds_10nm_ol_mc_tt/lucas_2009_wetlab_foss-xds_10nm_ol_mc_c-org_ols_tt.joblib",
          "c-org_ols_final"
        ],
        "dst_fp": "./project_lucas_2009/data/predictions/lucas_2009_wetlab_foss-xds_10nm"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | Folder with the spectra to predict |
| `dataframe` | The parquet with the spectra: its name, or `previous` |
| `model_array` | Models to apply: a `.joblib` path (relative to the project root, with its `.manifest.json` next to it) or the name of a [registered model](#register-a-final-model) |
| `dst_fp` | Output folder. Empty = `<project_root_fp>/predictions` |
| `show_observed_predicted` | Show the observed vs predicted plots (models whose indicator has observed values). Default `true` |
| `save_observed_predicted` | Save one plot per model under `<dst_fp>/plot/`. Default `false` |
| `plot_layout` | Arrangement of the shown plots, as in [regression_modeling][ml_model]: `default`, a preset (`scatter-1` … `scatter-4`, `scatter-tt-kf`), a parameter set, a JSON file, or `none` |
| `save_plot_layout` | Also save the grid figures under `<dst_fp>/plot/grid/`. Default `false` |
| `targetfeaturesymbols`, `regressionmodelsymbols` | Plot symbols, as in regression_modeling. Default `default` |

**Give unprocessed spectra.** The models refit and apply their own preprocessing. A dataframe whose
companion JSON shows spectral preprocessing (anything but outlier detection) is refused: it has the
same band names, so it would give wrong predictions without any error.

Before predicting, each model is checked:

- the `.joblib` must match the sha256 in its manifest (and, for a registered model, the sha256
  recorded at registration);
- the spectra must have all the model's bands (any column order) and the same signal type
  (e.g. absorbance);
- a different version of scikit-learn, numpy or scipy than the model was saved with gives a warning.

**Output** in `dst_fp`:

- `<name>_pred.parquet`: the key columns of the input plus one column per model,
  `<indicator>__<regressor>_tt` (a model with covariates: `<indicator>__<regressor>+<covariate>_tt`, e.g.
  `c-org__plsr+biogeo16_tt`), in the indicator's unit;
- `<name>_pred.json`: the source parquet and its sha256, and per model its file, sha256 and the
  metrics from training.

**Independent validation.** If the parquet also holds observed values of the indicator, the metrics
are computed on the samples that were **not** used to train the model (the training sample ids are
in the manifest). Samples that were used for training are excluded, and the run says how many. For a
registered model these metrics are also written to the registry (dataset
`independent:<parquet name>`; a rerun on the same parquet replaces them).

The same samples are plotted observed vs predicted, as in `regression_modeling`. The default layout
puts one indicator per row when several regressors predict it; with a single regressor the indicators
flow four per row; train/test and k-fold models of the same indicator and regressor sit side by side.
Two models with the same indicator, regressor and mode (e.g. different preprocessing) are numbered
(`plsr`, `plsr-2`).

## Register a final model

`register_model` (user stratum 4 and 5) records a model as final in the database schema `ml_model`.
Register only models you intend to use: a registered model is never changed or replaced, only
retired.

**Process file**: `project_lucas_2009/process/ml_model/register_model.json`

```json
{
  "process": [
    {
      "process": "register_model",
      "parameters": {
        "model_array": [
          "./project_lucas_2009/data/compiled/lucas_2009_wetlab_foss-xds_10nm/models/lucas_2009_wetlab_foss-xds_10nm_ol_mc_tt/lucas_2009_wetlab_foss-xds_10nm_ol_mc_c-org_ols_tt.joblib"
        ],
        "name": "c-org_ols_final",
        "abstract": "Organic carbon from FOSS XDS spectra, LUCAS 2009, mean-centred, OLS"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `model_array` | Saved models (`.joblib` with `.manifest.json`), relative to the project root |
| `name` | Registry name (one model only). Empty = the file name without `.joblib`. May contain `_`, not `@` or `#` |
| `suffix` | Appended to the name of every model in `model_array`, e.g. `-swir` or `_run2`. Use it when a new run gives model files the same names as already registered ones |
| `abstract` | What the model is for, the data it was trained on, and its limits |

Registration stores:

| Table | Content |
|---|---|
| `ml_model.model` | Name, indicator, regressor, mode, signal type, number of bands and samples, spectral range (`spectral_domain`, `spectral_unit`, `band_min`, `band_max`, `band_step`), the model file path and sha256, the source data and its sha256, the full manifest, status, who registered it and when |
| `ml_model.model_step` | The preprocessing chain, one row per step with its parameters |
| `ml_model.model_parameter_set` | The [parameter sets][ml_settings] the model used, and in which step |
| `ml_model.model_metric` | The test (or k-fold) metrics, and later the independent metrics |

Every parameter set the model used, in the modeling run and in the preprocessing steps behind its
input, becomes **frozen**: the database refuses to change or delete it, so the record of how the
model was made stays true. Settings read from files are not in the database; their content and
checksum are kept in the manifest.

The model file stays where it is: the registry stores its path and sha256, not the file. Keep
registered model files in a folder you do not clean up (set `model_fp` in `regression_modeling`), and
do not overwrite them: a changed file no longer matches its sha256 and is refused.
`regression_modeling` never overwrites a registered model file: a new model with the same file name
is saved as `<name>-2.joblib` (`-3`, ...) and a warning is printed.

A file that is already registered, or a name that is already taken, is refused.

Keep the name short and human: what the model is built from is stored in columns and tables, not in
the name. `band_min`/`band_max` are the range of the spectra the model needs as input; `band_step`
is the spectral resolution of the spectra before band selection (in the unit of `spectral_unit`:
`nm`, `um` or `cm-1`). Find models by indicator, region and resolution, e.g. organic carbon models
that need nothing outside 1000–2500 nm at 10 nm or finer:

```sql
SELECT m.name, m.regressor, m.band_min, m.band_max, m.band_step
FROM ml_model.model m
JOIN observation_utility.indicator i ON i.id = m.indicator_id
WHERE i.alias = 'c-org' AND m.status = 'registered'
  AND m.spectral_unit = 'nm' AND m.band_min >= 1000 AND m.band_max <= 2500 AND m.band_step <= 10;
```

Add `JOIN ml_model.model_step s ON s.model_id = m.id WHERE s.process = '...'` to filter on a
preprocessing step, and `ml_model.model_metric` to rank by e.g. independent `rmse`.

## Retire a model

**Process file**: `project_lucas_2009/process/ml_model/retire_model.json`

```json
{
  "process": [
    { "process": "retire_model", "parameters": { "model_array": ["c-org_ols_final"] } }
  ]
}
```

A retired model can no longer be applied by name. Its record, metrics and frozen parameter sets
are kept.

[ml_model]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_model/
[ml_settings]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_settings/
