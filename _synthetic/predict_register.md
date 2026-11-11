---
title: "Predict and register"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/predict_register/
author_profile: false
excerpt: "Validate all 16 models on 300 independent samples, register the chosen one in the database, clean up the intermediate files, and predict by the model's registered name."
date: 2026-11-11
last_modified_at: 2026-11-11
---

**Notebook**: `xspatula_lucas/synthetic_dataset/project_synthetic/4_ml_predict.ipynb`

## 4.1 Independent validation, all models

**Process file**: `process/predict/predict_validation.json`

```json
{
  "process": [
    {
      "process": "predict_independent",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_synthetic/data/compiled/synthetic_validation_10nm",
        "dataframe": "synthetic_validation_10nm",
        "model_array": [
          "./project_synthetic/models/synthetic_calibration_10nm.olif.snv.mc_tt/synthetic_calibration_10nm.olif.snv.mc_c-org_plsr_tt.joblib",
          "... 15 more ..."
        ],
        "dst_fp": "./project_synthetic/data/predictions/synthetic_validation_10nm"
      }
    }
  ]
}
```

All 16 models predict the 300 **unprocessed** validation spectra. The validation set has lab values, so the run also reports independent metrics:

| Indicator | n | PLSR r² | SVR r² | Random forest r² | Cubist r² |
|---|---|---|---|---|---|
| c-org | 300 | 0.822 | 0.774 | 0.694 | 0.843 |
| n-tot | 300 | 0.674 | 0.668 | 0.600 | 0.786 |
| ph-water | 300 | 0.745 | 0.740 | 0.644 | 0.872 |
| clay | 230 | 0.593 | 0.555 | 0.595 | 0.745 |

Clay has 230 samples because texture is missing for the rest.

![Independent validation, all models]({{ "/assets/media/synthetic/4.1%20Independent%20validation,%20all%20models.png" | relative_url }})

Output in `data/predictions/synthetic_validation_10nm/`: `_pred.parquet` (one column `<indicator>__<regressor>_tt` per model) and `_pred.json` (models, sha256, metrics). See [Predict and register][lucas_predict].

## 4.2 Register the chosen model

**Process file**: `process/ml_model/register_model.json`

```json
{
  "process": [
    {
      "process": "register_model",
      "parameters": {
        "model_array": [
          "./project_synthetic/models/synthetic_calibration_10nm.olif.snv.mc_tt/synthetic_calibration_10nm.olif.snv.mc_c-org_plsr_tt.joblib"
        ],
        "name": "synthetic_c-org_plsr",
        "abstract": "Organic carbon from synthetic FOSS XDS 10 nm spectra (calibration set, 2000 composites of LUCAS 2015 points); SNV + meancentring, PLSR, standard indicator transformation. Example model, not for real predictions."
      }
    }
  ]
}
```

This registers c-org PLSR as `synthetic_c-org_plsr`. In real work you register only after testing preprocessing chains and hyperparameters; it is done here to show how. To register another model, pick it from the 4.1 metrics and change `model_array` and `name`.

```
✅ registered "synthetic_c-org_plsr" (id 1): c-org plsr, traintest; wl_nm 400–2500, step 10; 3 step(s);
   parameter sets frozen: plot_layout/scatter-4@1, plot_symbol/regressionmodelsymbols@1,
   plot_symbol/targetfeaturesymbols@1, regressor/standard@2, spectra_plot/standard@1, transformation/standard@1
```

Registration:

- stores the path, sha256, manifest, preprocessing steps, metrics and spectral range in the schema `ml_model`;
- **freezes** the parameter sets the model used: they can no longer be edited or deleted;
- is permanent: a model can be retired, but not deleted or registered twice.

### What a registered model needs

With a registered model you only need the independent dataframe to rerun the predictions, with two conditions:

1. **The model files must stay.** The registry holds the path and sha256 of the `.joblib`, not the file. Prediction needs the `.joblib` and its `.manifest.json` in `models/`. It does not need the calibration data or the preprocessed dataframes. Deleting `data/` is safe; deleting `models/` breaks the registered model.
2. **The path is absolute.** Moving or renaming the repository folder also breaks the registered model.

If you deleted `data/`, re-create the validation dataframe by re-running cell 1.2 of [Compile and inspect][compile].

## 4.3 Clean up the intermediate dataframes

**Process file**: `process/cleanup/delete_dataframes_calibration.json`

```json
{
  "process": [
    {
      "process": "delete_dataframes",
      "parameters": {
        "project_root_fp": "./project_synthetic/data/compiled/synthetic_calibration_10nm",
        "keep_array": ["raw", "synthetic_calibration_10nm.olif.snv.mc"]
      }
    }
  ]
}
```

It lists what it keeps and what it deletes, then asks `y/n` (one more prompt at the top of the VS Code window). Answer `y`.

- **Deleted**: `synthetic_calibration_10nm.olif` and `.olif.snv` (parquet + companion JSON, about 8 MB), and their entries in `.previous_dataframe`.
- **Kept**: the raw compiled set (selected datasets are never deleted) and `.olif.snv.mc`.

`keep_array` accepts `raw`, `previous` (the last dataframe written) and dataframe names.

The lesson: a model holds its own preprocessing, so the intermediate steps are not needed once the chain is final. Their history stays in the companion JSON of `.olif.snv.mc` and in each model's manifest. `.olif.snv.mc` is kept because it is the modelling input: it carries the indicator values with outliers set to missing, and the [Covariates][covariates] notebooks train on it again. A new `regression_modeling` run needs it plus the raw set, nothing else.

## 4.4 Predict with the registered model

**Process file**: `process/predict/predict_validation_registered.json`

```json
{
  "process": [
    {
      "process": "predict_independent",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_synthetic/data/compiled/synthetic_validation_10nm",
        "dataframe": "synthetic_validation_10nm",
        "model_array": ["synthetic_c-org_plsr"],
        "dst_fp": "./project_synthetic/data/predictions/synthetic_validation_10nm_registered"
      }
    }
  ]
}
```

The model is named by its registry name instead of a file path. It runs after the cleanup, which shows that the model does not need the calibration data. The file is checked against the sha256 recorded at registration, and the independent metrics are written to the registry (dataset `independent:synthetic_validation_10nm`; a rerun replaces them).

```
independent metrics added to the registry for "synthetic_c-org_plsr" (independent:synthetic_validation_10nm)
c-org__plsr_tt  independent: n=300  r2=0.822  rmse=23.715  rpiq=1.027
```

![Prediction with the registered model]({{ "/assets/media/synthetic/4.4%20Predict%20with%20the%20registered%20model.png" | relative_url }})

Output in `data/predictions/synthetic_validation_10nm_registered/`.

### What you can do now

The registry is a set of database tables, so you can search it. For example, every registered organic carbon model that needs nothing outside 400–2500 nm at 10 nm or finer (the new model is one of them):

```sql
SELECT m.name, m.regressor, m.band_min, m.band_max, m.band_step
FROM ml_model.model m
JOIN observation_utility.indicator i ON i.id = m.indicator_id
WHERE i.alias = 'c-org' AND m.status = 'registered'
  AND m.spectral_unit = 'nm' AND m.band_min >= 400 AND m.band_max <= 2500 AND m.band_step <= 10;
```

For a field spectrometer covering only 1000–2500 nm, use `m.band_min >= 1000`: this model would then not be listed, because it needs the visible range. Filtering on preprocessing steps and ranking by independent metrics are covered in [Predict and register][lucas_predict].

## 4.5 Retire the model (disabled)

The cell starts with `%%script false`, so it does nothing. Retiring (`process/ml_model/retire_model.json`, `"model_array": ["synthetic_c-org_plsr"]`) means the name can no longer be used for predictions; the record, its metrics and the frozen parameter sets stay. To enable the cell, remove its first line.

## Next step

[Covariates][covariates].

[compile]: {{ "/synthetic/compile/" | relative_url }}
[covariates]: {{ "/synthetic/covariates/" | relative_url }}
[lucas_predict]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_predict/
