---
title: "ML Modeling"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/ml_model/
author_profile: false
---

Trains and evaluates regressors against any of the dataframes produced in
[ML preprocessing][ml_preprocess] — raw, meancentred, derivative, PCA-reduced, band-selected, or
any other chained variant.

## Notebook

**Path**: `xspatula_lucas/lucas/project_lucas_2009/ml_model.ipynb`

Just one step. Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the
default), then run the modeling cell.

## Machine Learning modeling with train-test

**Process file**: `project_lucas_2009/process/regression_modeling.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/lucas_400-2500_10",
        "dataframe": "raw",
        "indicator_array": ["c-org", "sand"],
        "regressor_array": ["ols", "huber"],
        "model_parameters_fp": "./project_lucas_2009/regressors/regression_model_parameters.json",
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
| `dataframe` | Which preprocessed dataframe to train against — `"raw"` for the unprocessed selection, an explicit chained name, or `"previous"` |
| `indicator_array` | Which target indicators to fit a separate model for, one at a time |
| `regressor_array` | Which regressors to fit per indicator — see the table below |
| `model_parameters_fp` | Path to the hyperparameter file for every regressor, or `"default"` |
| `traintest` | Split into train/test and evaluate on the held-out set |
| `test_size` | Fraction held out for testing (0.3 = 30%) |
| `kfold` | Use k-fold cross-validation instead of/alongside a single train-test split |

## Available regressors

From `xspatula_lucas/lucas/project_lucas_2009/regressors/regression_model_parameters.json`,
checked against `src/ai4sh/machine_learning_model.py`:

| Key | Regressor | Hyperparameters shipped |
|---|---|---|
| `ols` | Ordinary least squares | `fit_intercept: false` |
| `theil_sen` | Theil-Sen estimator | none set (library defaults) |
| `huber` | Huber regressor | none set |
| `knn` | K-nearest neighbours | none set |
| `dectree` | Decision tree | none set |
| `svr` | Support vector regression | `kernel: linear`, `C: 1.5`, `epsilon: 0.05` |
| `randfor` | Random forest | `n_estimators: 30` |
| `mlp` | Multi-layer perceptron | `hidden_layer_sizes: [100, 100]`, `max_iter: 200`, `tol: 0.001`, `epsilon: 1e-8` |
| `cubist` | Cubist | none set |

Any subset of these keys can go in `regressor_array` — `regression_modeling.json` above only
fits `ols` and `huber`, for example. Edit `regression_model_parameters.json` directly to change
hyperparameters for any of them.

## Output

Three things get written under `project_root_fp`, all keyed by the source dataframe's name:

- **`data-<name>_regression.json`** — one entry per indicator, each with one entry per regressor
  fit, holding the evaluation metrics: `r2`, `rmse`, `mae`, `mape`, `medae`, `bias`, `rpiq`, `ccc`,
  plus `n_train`/`n_test` and a `model_fp` pointing at the saved model.
- **`models/<name>_tt/`** — the fitted models themselves, one `.joblib` file per
  indicator/regressor combination (`<name>_<indicator>_<regressor>_tt.joblib`).
- **`covariate_importance/<name>_tt/`** — feature/band importance output for the fitted models.

Example, from a real run against `c-org` and `sand` with `ols` and `huber`:

```json
{
  "source_parquet": "data-foss xds rapid content analyzer_400-2500_10.parquet",
  "results": {
    "c-org": {
      "traintest": {
        "ols":   { "r2": 0.8108, "rmse": 20.313, "rpiq": 2.0418, "ccc": 0.8701, "n_train": 17, "n_test": 8 },
        "huber": { "r2": 0.6308, "rmse": 28.373, "rpiq": 1.4618, "ccc": 0.7080, "n_train": 17, "n_test": 8 }
      }
    },
    "sand": {
      "traintest": {
        "ols":   { "r2": -0.8317, "rmse": 27.868, "rpiq": 1.4533, "ccc": 0.1763, "n_train": 17, "n_test": 8 },
        "huber": { "r2": 0.8454,  "rmse": 8.095,  "rpiq": 5.0031, "ccc": 0.9167, "n_train": 17, "n_test": 8 }
      }
    }
  }
}
```

Note how the two regressors disagree sharply on `sand` (`ols` r² of -0.83 vs. `huber`'s 0.85) —
with only 25 test records in this example (`n_train: 17`, `n_test: 8`), that's expected noise, not
a signal to read into. Re-run against the full campaign (`RECORDS = 0` in
[Prepare data][prepare_data]) before drawing conclusions from any single regressor's metrics.

[ml_preprocess]: /lucas_2009/machine_learning/ml_preprocess/
[prepare_data]: /lucas_2009/prepare_data/
