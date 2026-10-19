---
title: "Plot Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/plot_data/
author_profile: false
---

Look at the selection before preprocessing: the distribution of each indicator, the class counts of the
covariates and filters, and the spectra under different scatter corrections, scalers and filters. The
plots help decide which transformation, standardisation and spectral enhancement to apply in
[ML preprocessing][ml_preprocess].

## Which file is plotted

`project_root_fp` points at a folder, not a file. The plot reads the latest selected parquet registered in
the folder's `.previous_dataframe`, normally the selection you just ran (see
[The previous dataframe][previous_dataframe]).

## Plot indicators and covariates

`plot_indicators` reads both `indicator_array` and `covariate_array` from the selection in
`project_root_fp`.

**Process file**: `project_lucas_2009/process/plot/plot_lucas2009_indicators.json`

```json
{
  "process": [
    {
      "process": "plot_indicators",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/indicator/lucas_eu_2009",
        "indicator_array": ["cec", "clay", "silt", "sand", "c-org", "n-tot", "ph-water"],
        "targetfeaturesymbols": "default",
        "standardisation": "default",
        "transformation": "default",
        "single_canvas_plot": true,
        "individual_canvas_plot": false,
        "boxplot": true,
        "histogram": true,
        "show": true,
        "save": true
      }
    }
  ]
}
```

- Without `indicator_array`, every column except the key, `class_*` and band columns is plotted.
- Numeric indicators get a boxplot and a histogram. `single_canvas_plot` draws all indicators on one
  canvas, `individual_canvas_plot` one canvas per indicator.
- Saved to `<project_root_fp>/plot/`, e.g. `boxplot_all-indicators_z_log.png` and
  `histogram_all-indicators_z_log.png` (the suffix names the standardisation and transformation), or
  `boxplot_<indicator>.png` per indicator.

**Process file**: `project_lucas_2009/process/plot/plot_covariate.json`

```json
{
  "process": [
    {
      "process": "plot_indicators",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/covariate/lucas_eu_2009",
        "covariate_array": ["landcover", "landuse", "biogeo16"],
        "class_label": "display_name",
        "targetfeaturesymbols": "default",
        "standardisation": "default",
        "transformation": "default",
        "single_canvas_plot": true,
        "individual_canvas_plot": false,
        "boxplot": true,
        "histogram": true,
        "show": true,
        "save": true
      }
    }
  ]
}
```

**Nominal columns** (classes) can't be histogrammed or transformed. They are drawn as horizontal bars of
class counts: largest on top, at most `max_classes` (default 15), the rest summed as
`other (N classes)`. Files: `classes_<name>.png` and `classes_all-covariates.png`.

| Parameter | Description |
|---|---|
| `class_label` | `display_name` (default), `tag`, `name` or `alias`. Only changes the plot labels, not the data |
| `max_classes` | Max number of classes per covariate, default 15 |

A rank other than `lowest` is shown in the title, e.g. "Land cover (order)".

**Bar style** per covariate in `lucas/default/plot/targetfeaturesymbols.json`:

```json
"landcover": {
  "color": "olivedrab",
  "alpha": 0.6,
  "hatch": "//",
  "label": "Land cover",
  "unit": "native"
}
```

- `color`, `alpha`, `edgecolor`
- `hatch`: `/`, `\`, `|`, `-`, `+`, `x`, `o`, `O`, `.`, `*`; double a character for denser hatching
- `fill: false` draws the outline only

Defaults: landcover olivedrab `//`, landuse goldenrod `\\`, biogeo16 teal `xx`, all at alpha 0.6. The
transformation and standardisation defaults for the three covariates are `false`
(`lucas/default/transform/*.json`); non-numeric columns are skipped anyway.

`targetfeaturesymbols` is the same file used for [indicator units][indicator_units]. In the plot it only
controls colour, label and axis unit; no translation happens on the plotted values.

### Plot a filter's class distribution

To see how a [class filtered][class_filters] selection splits, put its `class_*` column in
`covariate_array`. On the cropland/grassland selection:

```json
"indicator_array": ["clay"],
"covariate_array": ["class_landcover"]
```

prints `Plotting 1 numeric: ['clay'] and 1 nominal: ['class_landcover']` and saves
`classes_class_landcover.png` and `boxplot_clay.png`. `targetfeaturesymbols.json` has no entry for
`class_*` names, so the default bar style applies.

### Coming next

Colouring the indicator plots by class, and marker shape by campaign or sampling log, are planned. The
columns needed (`class_*`, `campaign_tag`, `sampling_log_tag`) are already in every parquet. There are no
plot parameters for it yet.

## Spectra plots

The four spectra plots below read the spectra selection. All four process files set

```json
"project_root_fp": "./project_lucas_2009/data/spectra/foss-xds-rca/ds2/lucas_eu_2009"
```

They are cells in both `select_spectra_only.ipynb` and `explore_select_data.ipynb`.

### Plot spectra and scatter correction options

Overlays the raw spectra against each available scatter-correction scaler (SNV, MSC, ...) so you can
compare them before picking one for [ML preprocessing][ml_preprocess].

**Process file**: `project_lucas_2009/process/plot/plot_spectra_scattercorrection_scalers.json`

### Plot spectra and scaling method options

Same idea, for the scaling methods (meancentring, autoscaling, paretoscaling, poissonscaling).

**Process file**: `project_lucas_2009/process/plot/plot_spectra_scaling_methods.json`

### Plot spectra

A plain spectra plot of the selected dataset, no processing applied.

**Process file**: `project_lucas_2009/process/plot/plot_spectra.json`

### Plot filter spectra

Overlays the raw spectra against each available smoothing filter (moving-average, gauss, Savitzky-Golay,
lowess).

**Process file**: `project_lucas_2009/process/plot/plot_filter_spectra.json`

## Next step

Proceed to [ML preprocessing][ml_preprocess] to clean and transform the compiled dataset
(`project_root_fp: ./project_lucas_2009/data/compiled/lucas_2009_10nm`).

[ml_preprocess]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_preprocess/
[previous_dataframe]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#the-previous-dataframe-previous_dataframe
[indicator_units]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#indicator-units
[class_filters]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/#class-filters
