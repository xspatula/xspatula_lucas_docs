---
title: "Covariates"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/covariates/
author_profile: false
excerpt: "Does knowing the region, land cover, land use or soil texture class improve predictions from spectra? Four notebooks repeat the model chain with one covariate each and compare against spectra alone."
date: 2026-11-11
last_modified_at: 2026-11-11
---

**Notebooks**: `xspatula_lucas/synthetic_dataset/project_synthetic/5_ml_model_predict_biogeo16.ipynb` … `8_ml_model_predict_texture.ipynb`

## The idea

A covariate is a class per sample: a region, a land cover, a soil texture group. It is one-hot encoded and added next to the spectra as extra model input. The question is whether it improves the predictions.

This matters in the field. A handheld spectrometer gives you the spectrum; region, land cover or a soil map class come from the location, through Earth observation and maps, not from the lab. If a covariate helps, it is information you get for free on the spot.

Each notebook repeats the chain of [Model][model] with **one** covariate, predicts the validation set with both model sets (spectra only, and spectra + covariate), and compares them on the same samples.

## How a covariate is set

**Process file**: `process/covariate_landcover/regression_modeling_landcover.json`. Only the parameters that differ from [Model][model]:

```json
"covariate_array": ["landcover"],
"class_grouping": [
  {
    "covariate": "landcover",
    "groups": [
      ["cultivated"],
      ["grass-l-no-trees", "grass-l-trees"],
      ["conif-woodl", "broadl-woodl", "mixed-woodl"]
    ],
    "other": "exclude"
  }
]
```

- `groups` pools classes into one: here all three forest types become one forest group.
- `"other": "exclude"` leaves the samples of all unlisted classes out of training and prediction. `"other": "<name>"` would instead pool them into one group with that name.
- Samples without a class, or with a class the model never saw, are not predicted.

See [Covariates in ML modeling][lucas_covariates].

## The four groupings

| Notebook | Covariate | Groups | Left out |
|---|---|---|---|
| 5 | `biogeo16` | boreal + alpine; atlantic; continental + pannonian + steppic + black-sea; mediterranean | none. Rare regions are pooled (steppic has 16 calibration samples, black-sea 3) |
| 6 | `landcover` | cultivated; grassland with and without trees; forest (coniferous, broadleaved, mixed) | agroforest, lichens, spontaneous vegetation, shrubland, swamp |
| 7 | `landuse` | agriculture; forestry | natural/semi-natural, recreation, unused, mining |
| 8 | `soil texture usda major group` | sandy, loamy, light clayey, heavy clayey (no grouping) | samples without texture (about 23%) |

## The cells, with biogeo16 as the example

Notebooks 6–8 are identical apart from the covariate.

**5.1 Train and test with biogeo16** (`process/covariate_biogeo16/regression_modeling_biogeo16.json`). Models go to `models/synthetic_calibration_10nm.olif.snv.mc_cv-biogeo16_tt/`.

![Train and test with biogeo16]({{ "/assets/media/synthetic/5.1%20Train%20and%20test%20with%20biogeo16.png" | relative_url }})

**5.2 Independent validation** (`process/covariate_biogeo16/predict_validation_biogeo16.json`) predicts with both model sets. The prediction file has columns `<indicator>__<regressor>_tt` (spectra only) and `<indicator>__<regressor>+biogeo16_tt` (with the covariate); the plot has one row per model set.

![Independent validation, spectra only and spectra + biogeo16]({{ "/assets/media/synthetic/5.2%20Independent%20validation-%20spectra%20only%20and%20spectra%20+%20biogeo16.png" | relative_url }})

**5.3 Compare** prints r² and RMSE for both model sets on the samples both could predict, with `change` = with covariate − spectra only (a negative RMSE change is better), then the RMSE per group:

```
                           r2              rmse             r2   rmse
model               +biogeo16 spectra +biogeo16 spectra change change
indicator regressor
c-org     cubist        0.845   0.843    22.139  22.289  0.002 -0.150
          plsr          0.794   0.822    25.528  23.715 -0.028  1.813
...
group                          atlantic  boreal|alpine  continental|...  mediterranean
c-org     cubist    +biogeo16    15.704         39.787           14.236          5.757
                    spectra      16.680         39.652           14.646          5.954
```

| Notebook | Process files | Model folder suffix |
|---|---|---|
| `6_ml_model_predict_landcover.ipynb` | `process/covariate_landcover/` | `_cv-landcover_tt` |
| `7_ml_model_predict_landuse.ipynb` | `process/covariate_landuse/` | `_cv-landuse_tt` |
| `8_ml_model_predict_texture.ipynb` | `process/covariate_texture/` | `_cv-soiltextureusdamajorgroup_tt` (spaces dropped) |

## Results

Change in independent r² when the covariate is added (spectra + covariate − spectra only), from the run of 2026-11-10. Each covariate is compared on its own sample set (samples in left-out classes are not predicted), so the spectra-only baselines differ slightly between columns.

| Indicator | Regressor | biogeo16 | landcover | landuse | texture |
|---|---|---|---|---|---|
| c-org | plsr | −0.028 | −0.012 | −0.002 | −0.009 |
| | svr | +0.005 | +0.058 | +0.023 | +0.005 |
| | randfor | −0.004 | +0.023 | +0.008 | −0.002 |
| | cubist | +0.002 | +0.015 | −0.001 | +0.030 |
| n-tot | plsr | −0.013 | +0.030 | +0.002 | +0.040 |
| | svr | +0.002 | +0.023 | +0.008 | +0.076 |
| | randfor | −0.000 | +0.010 | +0.007 | +0.040 |
| | cubist | +0.041 | −0.013 | +0.012 | +0.178 |
| ph-water | plsr | −0.017 | +0.015 | −0.007 | −0.000 |
| | svr | −0.005 | +0.013 | −0.008 | +0.020 |
| | randfor | +0.009 | +0.054 | +0.042 | −0.030 |
| | cubist | +0.011 | +0.014 | +0.025 | +0.010 |
| clay | plsr | +0.017 | −0.039 | −0.006 | **+0.226** |
| | svr | +0.011 | +0.002 | +0.004 | **+0.251** |
| | randfor | +0.004 | −0.006 | −0.042 | **+0.240** |
| | cubist | −0.005 | −0.071 | −0.024 | **+0.099** |

![Independent validation, spectra only and spectra + texture]({{ "/assets/media/synthetic/8.2%20Independent%20validation-%20spectra%20only%20and%20spectra%20+%20texture.png" | relative_url }})

Plots for the other covariates:
[6.1 landcover, train/test][f61] · [6.2 landcover, validation][f62] ·
[7.1 landuse, train/test][f71] · [7.2 landuse, validation][f72] ·
[8.1 texture, train/test][f81]

## What it shows

- **Region, land cover and land use add little.** Most changes are within ±0.03 r², in both directions, and no covariate helps all regressors for any indicator. Run-to-run differences of about ±0.02 r² are noise; treat smaller changes as no change.
- **The per-group RMSE is the more useful output.** It shows where a spectra-only model is weak. For c-org the boreal + alpine group has an RMSE of about 40 g/kg against about 6 g/kg in the mediterranean group: a few organic-rich northern soils dominate the overall error.
- **Texture improves clay by up to 0.25 r², but that is leakage.** The texture class was computed from the measured clay, silt and sand, so the model is partly given the answer. In the field the class would come from a soil map, and the gain would be much smaller. The teaching point: **check where a covariate comes from before you trust a gain.** The n-tot gain with texture has the same origin in part.
- The data are synthetic composites. Real LUCAS covariate effects may differ.

[model]: {{ "/synthetic/model/" | relative_url }}
[lucas_covariates]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_model/#covariates
[f61]: {{ "/assets/media/synthetic/6.1%20Train%20and%20test%20with%20landcover.png" | relative_url }}
[f62]: {{ "/assets/media/synthetic/6.2%20Independent%20validation-%20spectra%20only%20and%20spectra%20+%20landcover.png" | relative_url }}
[f71]: {{ "/assets/media/synthetic/7.1%20Train%20and%20test%20with%20landuse.png" | relative_url }}
[f72]: {{ "/assets/media/synthetic/7.2%20Independent%20validation-%20spectra%20only%20and%20spectra%20+%20landuse.png" | relative_url }}
[f81]: {{ "/assets/media/synthetic/8.1%20Train%20and%20test%20with%20texture.png" | relative_url }}
