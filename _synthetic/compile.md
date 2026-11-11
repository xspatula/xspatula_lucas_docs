---
title: "Compile and inspect"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/compile/
author_profile: false
excerpt: "Pull the calibration and validation sets out of the database as analysis-ready parquet files, and look at the indicators and spectra."
date: 2026-11-11
last_modified_at: 2026-11-11
---

**Notebook**: `xspatula_lucas/synthetic_dataset/project_synthetic/1_compile_dataset.ipynb`
**Scheme**: `scheme_file = '../scheme_synthetic.json'` (all notebooks in `project_synthetic/`)

From here on, process files are in `project_synthetic/process/`, data go to `project_synthetic/data/` and models to `project_synthetic/models/`. Models are kept apart from data on purpose; see [Predict and register][predict].

## 1.1 Compile the calibration set

**Process file**: `process/compile/compile_calibration.json`

```json
{
  "process": [
    {
      "process": "compile_dataset_from_db",
      "overwrite": false,
      "parameters": {
        "dataset": "synthetic",
        "sampling_log": "syn-cal_2025",
        "min_profile": 0,
        "max_profile": 20,
        "spectra_provision": "foss-xds-10nm",
        "spectra_preparation_name": "ds2",
        "begin_wavelength": 400,
        "end_wavelength": 2500,
        "output_bandwidth": 10,
        "output_spectra_signal": "absorb",
        "indicator_array": ["c-org", "n-tot", "ec", "ph-water", "clay", "silt", "sand"],
        "covariate_array": ["landcover", "landuse", "biogeo16", "soil texture usda major group"],
        "covariate_rank": "family",
        "output_root_fp": "./project_synthetic/data",
        "output_name": "synthetic_calibration_10nm"
      }
    }
  ]
}
```

| Parameter | Value | Meaning |
|---|---|---|
| `sampling_log` | `syn-cal_2025` | the calibration samples |
| `min_profile`, `max_profile` | 0, 20 | topsoil, 0–20 cm |
| `output_spectra_signal` | `absorb` | stored reflectance is converted to absorbance, log10(1/R) |
| `begin_wavelength` … `output_bandwidth` | 400, 2500, 10 | 211 bands |
| `indicator_array` | 7 lab indicators | |
| `covariate_array` | 4 nominal covariates | compiled now, used on the [Covariates][covariates] page |
| `covariate_rank` | `family` | land cover and land use are reported at family level: sunflower becomes its crop family. Fewer, larger classes |

Output: `data/compiled/synthetic_calibration_10nm/`. The next cell runs `inspect`, which prints a summary of the compiled set. The full parameter reference is [Select data][lucas_select]; the file layout is [Parquet files][lucas_parquet].

## 1.2 Compile the validation set

Same process, `process/compile/compile_validation.json`, with `sampling_log: syn-val_2026` and `output_name: synthetic_validation_10nm`.

**The validation set is not preprocessed.** A saved model carries its own preprocessing, so it takes raw absorbance spectra. That is the point of [Predict and register][predict].

## 1.3 Look at the calibration data

**Process files**: `process/plot/plot_indicators_calibration.json`, `process/plot/plot_spectra_calibration.json`

`plot_indicators` draws box plots and histograms of the 7 indicators. With `"transformation": "standard"` it shows them as the models will see them: **c-org, n-tot and ec are log-transformed**, because they are strongly right-skewed. Removing outliers does not remove the skew; the transformation does. In the figures below c-org, n-tot and ec are plotted as natural-log (ln) values.

![Box plots of the calibration indicators]({{ "/assets/media/synthetic/1.3%20Look%20at%20the%20calibration%20data%20box.png" | relative_url }})

![Histograms of the calibration indicators]({{ "/assets/media/synthetic/1.3%20Look%20at%20the%20calibration%20data.png" | relative_url }})

`plot_spectra` draws 100 spectra, raw and after the two preprocessing steps of the next page (SNV, then meancentring), so you can see what they do before you run them:

![Raw absorbance spectra]({{ "/assets/media/synthetic/1.3%20Look%20at%20the%20calibration%20data%20spectra%20raw.png" | relative_url }})

![SNV-corrected spectra]({{ "/assets/media/synthetic/1.3%20Look%20at%20the%20calibration%20data%20spectra%20snv.png" | relative_url }})

![Meancentred spectra]({{ "/assets/media/synthetic/1.3%20Look%20at%20the%20calibration%20data%20spectra%20mc.png" | relative_url }})

See [Plot data][lucas_plot] for all plot options.

## Next step

[Preprocess][preprocess].

[predict]: {{ "/synthetic/predict_register/" | relative_url }}
[covariates]: {{ "/synthetic/covariates/" | relative_url }}
[preprocess]: {{ "/synthetic/preprocess/" | relative_url }}
[lucas_select]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/select_data/
[lucas_parquet]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/parquet_files/
[lucas_plot]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/plot_data/
