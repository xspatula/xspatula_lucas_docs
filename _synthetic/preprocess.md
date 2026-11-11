---
title: "Preprocess"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/preprocess/
author_profile: false
excerpt: "Remove indicator outliers, correct scatter with SNV and meancentre the spectra. Three steps, each saved as a new dataframe that records its history in its name."
date: 2026-11-11
last_modified_at: 2026-11-11
---

**Notebook**: `xspatula_lucas/synthetic_dataset/project_synthetic/2_ml_preprocess.ipynb`

Only the calibration set is preprocessed. Each step reads the output of the step before it (`"dataframe": "previous"`) and writes a new dataframe.

| Cell | Process | Output dataframe |
|---|---|---|
| 2.1 | `detect_outliers`, isolation forest on c-org, n-tot and ec, threshold 0.05 | `synthetic_calibration_10nm.olif` |
| 2.2 | `spectra_scatter_correction`, SNV | `synthetic_calibration_10nm.olif.snv` |
| 2.3 | `spectra_scaling`, meancentring | `synthetic_calibration_10nm.olif.snv.mc` |

<div class="notice--info" markdown="1">
**The cells wait for your answers.** In VS Code the input box appears at the **top of the window**, not in the cell, and the cell looks hung until you answer. In order:

1. 2.1: the outlier fraction. Enter keeps 0.05; the example below used `0.025`.
2. 2.1: which indicators' outliers to remove: `all`, or indicator names.
3. 2.2: save the SNV result? `y`
4. 2.3: save the meancentred result? `y`
</div>

## 2.1 Indicator outliers

**Process file**: `process/ml_preprocess/1_detect_outliers.json`

```json
{
  "process": [
    {
      "process": "detect_outliers",
      "parameters": {
        "project_root_fp": "./project_synthetic/data/compiled/synthetic_calibration_10nm",
        "dataframe": "raw",
        "indicator_array": ["c-org", "n-tot", "ec"],
        "detector": "iforest",
        "threshold": 0.05
      }
    }
  ]
}
```

The run reports how many values each threshold flags, then asks which fraction to use:

```
Outliers detected (threshold=0.050) — c-org: 100  n-tot: 100  ec: 99
Outliers detected (threshold=0.025) — c-org: 50  n-tot: 50  ec: 49
Removed 149 outlier values across 3 indicators.
```

**An outlier is set to missing for that indicator only.** The sample stays, with its spectrum and its other indicators.

![Indicator outliers at threshold 0.025]({{ "/assets/media/synthetic/2.1%20Indicator%20outliers%200.025.png" | relative_url }})

For comparison, the default threshold 0.05 flags twice as many:

![Indicator outliers at threshold 0.05]({{ "/assets/media/synthetic/2.1%20Indicator%20outliers%200.05.png" | relative_url }})

## 2.2 SNV scatter correction

**Process file**: `process/ml_preprocess/2_scatter_correction_snv.json`, `"scaler": ["snv"]`. Standard normal variate removes differences in overall level and slope between spectra caused by particle size and packing, not by chemistry.

![SNV scatter correction]({{ "/assets/media/synthetic/2.2%20SNV%20scatter%20correction.png" | relative_url }})

## 2.3 Meancentring

**Process file**: `process/ml_preprocess/3_meancentring.json`, `"method": "meancentring"`. Subtracts the mean spectrum, so the models see the differences between samples.

![Meancentring]({{ "/assets/media/synthetic/2.3%20Meancentring.png" | relative_url }})

If you answer `n` at 2.2, meancentring runs on the uncorrected spectra, and the plot shows it.

## The dataframe name is its history

The final dataframe is `synthetic_calibration_10nm.olif.snv.mc` (a parquet file plus a JSON file describing the whole chain). The dot suffixes record the steps: `olif` = indicator outliers by isolation forest, `snv` = scatter correction, `mc` = meancentring. The next notebook expects exactly this name.

How the chain is saved, and every other preprocessing method, is in [ML preprocessing][lucas_preprocess].

## Next step

[Model][model].

[model]: {{ "/synthetic/model/" | relative_url }}
[lucas_preprocess]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_preprocess/
