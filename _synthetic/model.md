---
title: "Model"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/model/
author_profile: false
excerpt: "Train 16 models (4 indicators × 4 regressors) on the preprocessed calibration set. Each saved model is a complete pipeline: raw spectra in, indicator values out."
date: 2026-11-11
last_modified_at: 2026-11-11
---

**Notebook**: `xspatula_lucas/synthetic_dataset/project_synthetic/3_ml_model.ipynb`

## 3.1 Parameter sets available (optional)

**Process file**: `process/ml_setting/list_parameter_sets.json`

```json
{ "process": [ { "process": "list_parameter_sets", "parameters": { "family": "" } } ] }
```

Lists the parameter sets stored in the database (schema `ml_setting`): family (regressor hyperparameters, indicator transformation, plot symbols …), name, version and category (`raw`, `preset` or `custom`). `"family": ""` lists all families.

```
family           name                       v  category frozen creator        abstract
filter           gauss                      1  preset         setup          Standard gauss filter.
filter           lowess                     1  preset         setup          Standard lowess filter.
...
```

The names in the list are the values you can give in 3.2: `"default"` means the preset `standard`, `name@version` a specific version. The cell only reads; nothing changes. See [ML settings][lucas_settings].

## 3.2 Train and test

**Process file**: `process/ml_model/regression_modeling.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_synthetic/data/compiled/synthetic_calibration_10nm",
        "dataframe": "synthetic_calibration_10nm.olif.snv.mc",
        "indicator_array": ["c-org", "n-tot", "ph-water", "clay"],
        "regressor_array": ["plsr", "svr", "randfor", "cubist"],
        "model_parameters_fp": "default",
        "indicator_transformation": "standard",
        "indicator_standardisation": "none",
        "refit_preprocessing": true,
        "traintest": true,
        "test_size": 0.3,
        "kfold": false,
        "model_fp": "./project_synthetic/models",
        "show_observed_predicted": true,
        "save_observed_predicted": true
      }
    }
  ]
}
```

| Parameter | Value | Meaning |
|---|---|---|
| `dataframe` | `synthetic_calibration_10nm.olif.snv.mc` | the output of [Preprocess][preprocess] |
| `indicator_array` | c-org, n-tot, ph-water, clay | |
| `regressor_array` | plsr, svr, randfor, cubist | 4 × 4 = 16 models |
| `model_parameters_fp` | `default` | hyperparameters: the preset `standard` |
| `indicator_transformation` | `standard` | log for c-org, n-tot and ec; predictions are back-transformed to the original unit |
| `refit_preprocessing` | `true` | SNV and meancentring are refitted on the training samples and stored in the model |
| `test_size` | 0.3 | holdout chosen by a hash of the sample id: the same samples every run |
| `traintest` / `kfold` | `true` / `false` | k-fold is slower; set `"kfold": true` to add it |
| `model_fp` | `./project_synthetic/models` | models are kept outside `data/` |

**A saved model is a complete pipeline**: raw absorbance spectra in, indicator values out, with the preprocessing inside. That is why the validation set was compiled but not preprocessed.

Output, per model, in `models/synthetic_calibration_10nm.olif.snv.mc_tt/`: a `.joblib` file (the fitted pipeline) and a `.manifest.json` (what went into it), plus observed-versus-predicted plots of the test samples.

![Observed versus predicted, test samples, 16 models]({{ "/assets/media/synthetic/3.2%20Train%20and%20test.png" | relative_url }})

Regressors, the train/test split and the refitted preprocessing are explained in [ML modeling][lucas_model].

## Next step

[Predict and register][predict].

[preprocess]: {{ "/synthetic/preprocess/" | relative_url }}
[predict]: {{ "/synthetic/predict_register/" | relative_url }}
[lucas_settings]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_settings/
[lucas_model]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_model/
