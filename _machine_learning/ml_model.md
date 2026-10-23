---
title: "ML Modeling"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/ml_model/
author_profile: false
---

Trains and evaluates regressors on any dataframe produced in [ML preprocessing][ml_preprocess]:
raw, meancentred, derivative, PCA-reduced, band-selected, or any other chained variant. Every saved
model is a complete pipeline, from unprocessed spectra to a prediction in the native unit of the
indicator, so it can be applied to new spectra (see [Predict and register][ml_predict]).

## Notebook

**Path**: `xspatula_lucas/lucas/project_lucas_2009/ml_model.ipynb`

Just one step. Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the
default), then run the modeling cell.

## Machine Learning modeling with train-test

**Process file**: `project_lucas_2009/process/ml_model/regression_modeling.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_wetlab_foss-xds_10nm",
        "dataframe": "previous",
        "indicator_array": ["c-org", "sand"],
        "regressor_array": ["ols", "huber"],
        "model_parameters_fp": "default",
        "indicator_transformation": "standard",
        "indicator_standardisation": "none",
        "traintest": true,
        "test_size": 0.3,
        "kfold": false
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `dataframe` | The dataframe to model: `"raw"` for the unprocessed selection, an explicit chained name, or `"previous"` |
| `indicator_array` | The target indicators; one model per indicator and regressor |
| `regressor_array` | The regressors to fit per indicator, see the table below |
| `model_parameters_fp` | Regressor hyper-parameters: `default` (the `standard` set), `none` (scikit-learn defaults), a set name or a JSON file. See [ML settings][ml_settings] |
| `indicator_transformation` | Transformation of the indicator before fitting: `none`, `default`/`standard`, a set name or a JSON file. Default `none` |
| `indicator_standardisation` | Z-score standardisation of the indicator: same forms. Default `none` |
| `refit_preprocessing` | Refit the preprocessing chain of the dataframe inside the model on the training samples (default `true`, see below) |
| `traintest` | Split into train and test samples and evaluate on the test samples |
| `test_size` | Fraction of samples held out for testing (0.3 = 30%) |
| `kfold` | Evaluate with k-fold cross-validation (`folds` folds, default 10) |
| `model_fp` | Folder for the saved models, relative to the project root. Empty = `<project_root_fp>/models` |

Set `indicator_transformation` and `indicator_standardisation` here, deliberately. They are not
taken over from the transformation you last looked at in a plot.

## Available regressors

Hyper-parameters of the `standard` regressor set (`model_parameters_fp: "default"`):

| Key | Regressor | Hyper-parameters |
|---|---|---|
| `ols` | Ordinary least squares | `fit_intercept: true` |
| `theil_sen` | Theil-Sen estimator | `random_state: 42` |
| `huber` | Huber regressor | library defaults |
| `knn` | K-nearest neighbours | library defaults |
| `dectree` | Decision tree | `random_state: 42` |
| `svr` | Support vector regression | `kernel: linear`, `C: 1.5`, `epsilon: 0.05` |
| `randfor` | Random forest | `n_estimators: 30`, `random_state: 42` |
| `mlp` | Multi-layer perceptron | `hidden_layer_sizes: [100, 100]`, `max_iter: 200`, `tol: 0.001`, `epsilon: 1e-8`, `random_state: 42` |
| `cubist` | Cubist | `random_state: 42` |

`random_state: 42` makes the stochastic regressors give the same model every run. To change
hyper-parameters, create a custom regressor set ([ML settings][ml_settings]) rather than editing a
file.

## Train and test samples

The test samples are chosen from the `sample_id`, not from the row order: the `round(test_size × n)`
samples with the lowest hash of their `sample_id` are the test samples. The same sample is therefore
always a test sample for the same `test_size`, whatever order or subset the dataframe has.

Band selection ([permutation and univariate selection][ml_preprocess]) uses the same rule. Give the
selection and the modeling the **same `test_size`**, otherwise the model's test samples include
samples the band selection has already seen, and the test metrics are too optimistic. The model
prints a note when the two differ.

## The preprocessing is refitted inside the model

With `refit_preprocessing: true` (the default), the model does not train on the preprocessed
dataframe directly. It starts from the unprocessed source spectra (`source_parquet` in the
dataframe's companion JSON) and replays its `preprocessing_chain` as steps of a scikit-learn
pipeline, fitted on the training samples only. Mean centring, scaling, the MSC reference spectrum,
PCA, Ward clusters, variance threshold and univariate band selection therefore never see the test
samples, and their fitted values are saved with the model.

Two steps are not refitted, and the model run says so in its notes:

- **Outlier detection** (`_ol`): indicator outliers were set to missing using all samples.
- **Permutation selection**: the selected bands are taken as fixed. They are clean only if the
  selection used the same `test_size` (see above).

The indicator transformation and standardisation are part of the pipeline too, so predictions are
returned in the native unit of the indicator.

## Output

Written under `project_root_fp`, named after the input dataframe:

- **`<name>_regression.json`**: per indicator and regressor the test metrics `r2`, `rmse`, `mae`,
  `mape`, `medae`, `bias`, `rpiq`, `ccc`, with `n_train`/`n_test`, the model file and its sha256.
  `input_parquet` is the dataframe you gave, `source_parquet` the unprocessed spectra the pipeline
  starts from.
- **`models/<name>_tt/`** (or `_kf/`), per indicator and regressor:
  - `<name>_<indicator>_<regressor>_tt.joblib`: the fitted pipeline (preprocessing, regressor,
    indicator transformation);
  - `<name>_<indicator>_<regressor>_tt.manifest.json`: everything needed to trust and reuse the
    model: the sha256 of the `.joblib`, indicator and unit, regressor and hyper-parameters, the input
    bands and signal type, the preprocessing chain, the training sample ids, the metrics, and the
    provenance (process parameters, every settings set or file used with its checksum, the software
    versions and the git commit).
- **`covariate_importance/<name>_tt/`**: feature and band importance of the fitted models.
- **`plot/<name>_tt/`**: the observed-predicted and importance plots, if saved.

Example, `c-org` from a run on `lucas_2009_wetlab_foss-xds_10nm_ol_mc`:

```json
"c-org": {
  "traintest": {
    "ols":   { "r2": 0.2467,  "rmse": 11.916,  "rpiq": 1.7455, "ccc": 0.5445, "n_train": 32, "n_test": 13 },
    "huber": { "r2": -0.0707, "rmse": 14.2058, "rpiq": 1.4642, "ccc": 0.3111, "n_train": 32, "n_test": 13 }
  }
}
```

This run had 50 samples and 211 bands: more unknowns than training samples, so the metrics mean
little. Model the full campaign before drawing conclusions.

## Next step

[Predict and register][ml_predict]: apply a saved model to new spectra, and register the final
model in the database.

[ml_preprocess]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_preprocess/
[ml_settings]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_settings/
[ml_predict]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_predict/
