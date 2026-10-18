---
title: "Explore & Select Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/explore_select_data/
author_profile: false
---

The first step of the machine learning pipeline: pull the data you need out of the database and
onto local disk, compile it into one project dataset, and explore it to decide if and what
preprocessing steps to apply.

Spectra, indicators (the targets you want to predict) and covariates (e.g. land cover, land use,
biogeographic region) are selected **separately**, each into its own tag-named Parquet file. A
compile step then joins them on `sample_id` into the dataset that every later ML step works on.

The extracted files contain LUCAS data and fall under the
[LUCAS_SOIL licence agreement][lucas_license]: keep them within your own organisation. Published
results must be aggregated so that no individual point can be identified, and maps may not show
detectable sample locations (art. 5).

## Notebooks

All notebooks are under `xspatula_lucas/lucas/project_lucas_2009/`. Set
`scheme_file = '../scheme_lucas.json'` in the setup cell (already the default).

| Notebook | Cells (process files under `project_lucas_2009/process/`) |
|---|---|
| `select_spectra_only.ipynb` | `select_lucas_spectra_2009_10nm.json`, then the four spectra plots |
| `select_indicator_only.ipynb` | `select/select_list_lucas2009_indicator.json`, `select/select_save_lucas2009_indicator.json`, `plot/plot_indicator.json` |
| `select_landscape_covariate_only.ipynb` | `select/select_list_lucas2009_indicator.json`, `select/select_save_lucas2009_covariate.json`, `plot/plot_covariate.json` |
| `explore_select_data.ipynb` | the indicator list, the all-in-one `select_lucas_2009_10nm.json`, and all plots |

Run the three `*_only` notebooks, then [compile](#compile-the-project-dataset) the selections.
`explore_select_data.ipynb` is the older all-in-one route (see
[All in one: select_spectra](#all-in-one-select_spectra)); its plot cells read the separate
selections, so they work once the `*_only` selections exist.

## The select processes

One function does all selections; the process sets the **role** of the output (`SELECT_ROLE_D` in
`src/ai4sh/select.py`):

| Process | Role | Saves |
|---|---|---|
| `select_spectra_only` | `spectra` | spectra only |
| `select_indicator_only` | `indicator` | numeric (and nominal) indicators |
| `select_covariate_only` | `covariate` | covariates, e.g. land cover, land use, biogeo16 |
| `select_spectra` | `all` | spectra + indicators (+ covariates) in one file |
| `compile_dataset` | — | joins saved selections on `sample_id` |
| `select_indicator` | — | prints what is available, saves nothing |

The process decides the role, not the data: pH can be an indicator in one file and a covariate in
another. Whether a name is a numeric measurement, a nominal classification or a landscape class
is looked up in the database.

### Scope parameters

All select processes take the same scope parameters:

| Parameter | Description |
|---|---|
| `dataset` | Dataset name, tag or alias, e.g. `lucas` |
| `campaign` | Optional: one campaign (the place), e.g. `lucas_eu` |
| `sampling_log` | Optional: one sampling log (the survey round), e.g. `lucas_eu_2009` |
| `sample_array` | Optional: a list of sample names; the output sub path then ends with `sample_select` |
| `min_profile` / `max_profile` | Depth profile filter, cm |

Each can be given as name, tag or alias. If a value matches more than one record, the process
stops and asks for the tag, or the parent (dataset/campaign). See
[Auto naming and tags explained][auto_naming_explained] for the LUCAS names and tags.

Each process skips silently if its output already exists, unless `overwrite: true`.

## Find the available indicators

Prints the indicators available for a dataset/campaign — a discovery step, nothing is saved.

**Process file**: `project_lucas_2009/process/select/select_list_lucas2009_indicator.json`

```json
{
  "process": [
    {
      "process": "select_indicator",
      "overwrite": false,
      "parameters": {
        "dataset_name": "lucas",
        "campaign_name": ""
      }
    }
  ]
}
```

`select_indicator` still uses the old parameter names `dataset_name`/`campaign_name`. Leave
`campaign_name` empty to list everything under the `lucas` dataset, or set it to one campaign
(e.g. `lucas_eu`).

## Select spectra

Queries the database, resamples the spectra to a target wavelength grid, optionally converts the
signal, and saves the result as a `.parquet` file plus a companion `.json`.

**Process file**: `project_lucas_2009/process/select_lucas_spectra_2009_10nm.json`

```json
{
  "process": [
    {
      "process": "select_spectra_only",
      "overwrite": false,
      "parameters": {
        "dataset": "lucas",
        "campaign": "",
        "sampling_log": "lucas_eu_2009",
        "min_profile": 0,
        "max_profile": 20,
        "spectra_provision": "foss-xds-rca",
        "spectra_preparation_name": "ds2",
        "begin_wavelength": 400,
        "end_wavelength": 2500,
        "output_bandwidth": 10,
        "output_spectra_signal": "absorb",
        "output_root_fp": "./project_lucas_2009/data"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `spectra_provision` | Spectrometer provision (name, tag or alias), here the tag `foss-xds-rca` |
| `spectra_preparation_name` | Sample preparation (`ds2`, matching the observation log) |
| `begin_wavelength` / `end_wavelength` / `output_bandwidth` | Target wavelength grid — resampled from the spectrometer's native `wavelength_array` (see [Observations explained][observations_explained]), not assumed to match it |
| `output_spectra_signal` | `absorb` converts the stored diffuse reflectance to absorbance on the way out |
| `output_root_fp` | Root directory of all selections |

**Output**: `data/spectra/foss-xds-rca/ds2/lucas_eu_2009/400-2500_10.parquet` (+ `.json`).

## Select indicators

**Process file**: `project_lucas_2009/process/select/select_save_lucas2009_indicator.json`

```json
{
  "process": [
    {
      "process": "select_indicator_only",
      "overwrite": false,
      "parameters": {
        "dataset": "lucas",
        "campaign": "",
        "sampling_log": "lucas_eu_2009",
        "min_profile": 0,
        "max_profile": 20,
        "indicator_array": ["cec", "clay", "silt", "sand", "c-org", "n-tot", "ph-water"],
        "data_range": "",
        "output_root_fp": "./project_lucas_2009/data"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `indicator_array` | The lab indicators to select. Each entry is resolved by indicator name or alias — `ph-water` here and `ph-h2o` in the observation records (see [Observations explained][observations_explained]) are the same indicator |
| `data_range` | Optional JSON `{indicator: {min, max}}` to filter values; empty = no filter |
| `targetfeaturesymbols` | Which `unit` to save each indicator in — see [Indicator units](#indicator-units). Optional, defaults to `"default"` |

If a requested indicator has no data in the database, the process lists what is available and
asks whether to continue without it.

**Output**: `data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet`
(+ `.json`).

## Select covariates

**Process file**: `project_lucas_2009/process/select/select_save_lucas2009_covariate.json`

```json
{
  "process": [
    {
      "process": "select_covariate_only",
      "overwrite": false,
      "parameters": {
        "dataset": "lucas",
        "campaign": "",
        "sampling_log": "lucas_eu_2009",
        "min_profile": 0,
        "max_profile": 20,
        "covariate_array": ["landcover", "landuse", "biogeo16"],
        "covariate_rank": "order",
        "output_root_fp": "./project_lucas_2009/data"
      }
    }
  ]
}
```

Land cover, land use and biogeo16 describe the geolocation, not the sample; they are joined to
the samples by position. `covariate_rank` is explained in
[Covariates and ranks](#covariates-and-ranks) below.

**Output**: `data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet`
(+ `.json`).

## Output names

Output paths are built from tags (`_Ml_output_paths` in `select.py`). The defaults:

```
{output_root_fp}/{role}/{provision tag}/{preparation alias}/{scope tag}/{begin}-{end}_{bandwidth}.parquet   spectra
{output_root_fp}/{role}/{scope tag}/{aliases}_{min}-{max}.parquet                                        indicator, covariate
{output_root_fp}/compiled/{output_name}/{output_name}.parquet                                             compile
```

The **scope tag** is the most specific one given: sampling_log > campaign > dataset. The LUCAS
2009 files:

```
data/spectra/foss-xds-rca/ds2/lucas_eu_2009/400-2500_10.parquet
data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet
data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet
data/compiled/lucas_2009_10nm/lucas_2009_10nm.parquet
```

- Each `.parquet` has a companion `.json` with the same name (no `data-`/`params-` prefix).
- An indicator or covariate list longer than 60 characters is cut and given an 8-character hash
  of the full list.
- A covariate rolled up to a rank gets the rank in the name (`landcover.order`).
- Set `output_sub_fp` and/or `output_name` to override the defaults.

**Key columns** in every output: `sample_id` (the join key — the same LUCAS point exists in 2009
and 2015, so sample names are not unique across sampling logs), `sample_name`, `sampling_log_tag`,
`latitude_dd`, `longitude_dd`, `profile_min`, `profile_max`.

## Compile the project dataset

Joins the saved selections into one dataset. The compiled folder is the `project_root_fp` of all
later ML steps.

**Process file**: `project_lucas_2009/process/compile_lucas_2009_10nm.json`

```json
{
  "process": [
    {
      "process": "compile_dataset",
      "overwrite": false,
      "parameters": {
        "input_array": [
          "./project_lucas_2009/data/spectra/foss-xds-rca/ds2/lucas_eu_2009/400-2500_10.parquet",
          "./project_lucas_2009/data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet"
        ],
        "output_root_fp": "./project_lucas_2009/data",
        "output_name": "lucas_2009_10nm"
      }
    }
  ]
}
```

- `input_array` needs at least two files; `output_name` is required.
- The inputs are **inner joined** on `sample_id`: only samples present in all inputs are kept.
- Key columns are kept once; a feature column found in two inputs is an error.
- Units, `covariate_rank` and `class_labels` are merged, and the companion JSON lists each input
  with its role and sample count.

To add the covariates, append
`./project_lucas_2009/data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet`
to `input_array`.

**Output**: `data/compiled/lucas_2009_10nm/lucas_2009_10nm.parquet` (+ `.json`).

If a folder used as `project_root_fp` holds several candidate selections, the newest is used and
the others are listed in a warning (`Find_selected_parquet`). Derived files (with step suffixes
like `_ol`, `_mc`) are not candidates. Delete stale files, e.g. old `data-*.parquet` files from
before the tag-based names, to silence the warning.

## All in one: select_spectra

`select_spectra` (role `all`) saves spectra, indicators and covariates in one file, without a
compile step. Used by `explore_select_data.ipynb`.

**Process file**: `project_lucas_2009/process/select_lucas_2009_10nm.json` — the parameters of
`select_lucas_spectra_2009_10nm.json` plus `indicator_array` and `data_range`.

Default output name: `{begin}-{end}_{bandwidth}+{indicator aliases}[+{covariate aliases}]_{min}-{max}`
under `data/all/foss-xds-rca/ds2/lucas_eu_2009/`.

Old parameter names, if you have your own process files:

| Old | New |
|---|---|
| `dataset_name` | `dataset` |
| `campaign_name` | `campaign` |
| — | `sampling_log` |
| `provision_name` | `spectra_provision` |
| `preparation_name` | `spectra_preparation_name` |
| `as_absorbance: true` | `output_spectra_signal: "absorb"` |

## Indicator units

Different provisions and campaigns can record the same indicator in different units — the select
processes harmonise them at extraction time, so every value in the saved `.parquet` ends up in one
consistent unit per indicator.

**Where the target unit comes from**: `lucas/default/plot/targetfeaturesymbols.json`, keyed by
indicator name, e.g.:

```json
"c-org": {
  "color": "dimgray",
  "alpha": 0.2,
  "label": "Organic carbon (C)",
  "unit": "native"
}
```

All indicators in the default file use `"unit": "native"`. Point `targetfeaturesymbols` at a
different file (a path, instead of `"default"`) to use your own per-indicator unit settings — the
same file also drives the axis units and colors of the [plots](#plot-indicators-and-covariates).

**Two ways to set an indicator's `unit`:**

- `"native"` — keep whatever unit the DB recorded for that indicator, no translation. Simplest
  option; use it when you don't need to combine this extraction with data recorded in a different
  unit. If the DB has mixed native units for one indicator across records, the process prints a
  warning and uses the first one it finds as the saved unit label — the values themselves are left
  untouched, so check this isn't silently mixing units before trusting the result.
- A specific unit name (e.g. `"percent"`) — every record gets translated into that unit via
  `observation_utility.unit_translate`. That translation must already exist as a row in the table —
  see [Adding a new unit translation][foreign_key_explained] if it doesn't. If it's missing, the
  process raises an error naming the exact `unit_translate.xlsx` row to add (source unit,
  destination unit). Add the row, then re-run cell 1.2 (Insert observation utilities) of
  `insert_lucas_data.ipynb`.

**Combining datasets later**: if you plan to compile selections from different provisions or
sampling logs (e.g. 2009 and 2015), set the *same* target unit for every indicator they share —
g/kg from one and percent from the other won't merge into anything usable.

**Where it ends up**: the unit applied to each column is written to the output `.json`'s
`indicator_units` field, and embedded directly in the `.parquet` file as column metadata — see
[Inspect dataset][inspect_dataset] for how to view it (`columns_units_data` / `columns_units_only`).
Nominal columns (classes) get the unit `class`.

## Invalid values

Two safety nets keep invalid values out of the models. They apply to all datasets, and come on
top of the [LUCAS 2009 import cleaning][data_cleaning].

**Selection** (`_Screen_values` in `select.py`): every numeric indicator or covariate is checked
after unit translation. A value is set to missing if it is

- a missing-value code: `-9999`, `-999`, `-888` (`MISSING_VALUE_CODES`), or
- outside the physical range of its unit (`Valid_unit_range`): percent 0–100, permill 0–1000,
  `g*kg^-1` 0–1000, `mg*kg^-1` 0–1e6, pH 0–14, and no negative contents, concentrations,
  conductivities, lengths or areas.

Each removal is printed per indicator, e.g.
`⚠️  clay: 3 value(s) are missing codes or outside the valid range for weight percent, set to missing: [-999.0]`.

**Transformation** (`chemometrics.py`): log of values ≤ 0, square root of values < 0 and the
reciprocal of 0 give a missing value with a warning, instead of ±inf that crashes the plots.

## Covariates and ranks

Land cover and land use are ranked order > family > genus (see
[Utility explained][utility_explained]). `covariate_rank` sets the rank of the selected classes:

- `order`, `family`, `genus`, or `lowest` (the rank recorded, the default).
- One rank for all covariates (`"order"`), or JSON per covariate:
  `{"landcover": "family", "landuse": "order"}`. Covariates not listed keep `lowest`.

A recorded class is rolled up through its parents to the requested rank. A class recorded at a
coarser rank than requested is kept as recorded, with a warning giving the count:

```
⚠️  landuse: 3 of 50 samples recorded at a coarser rank than family, kept as recorded
```

The parquet stores the class **tag**. The companion JSON gets `covariate_rank` and `class_labels`
(tag → `name`, `display_name`, `alias`, `rank`); compile keeps both. From
`data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.json`:

```json
"covariate_rank": {
  "landcover": "order",
  "landuse": "order",
  "biogeo16": "lowest"
},
"class_labels": {
  "landcover": {
    "cropland": {
      "name": "cropland",
      "display_name": "Cropland",
      "alias": "b",
      "rank": "order"
    },
    ...
  },
  "landuse": {
    "prim-prod": {
      "name": "primary production",
      "display_name": "Primary production",
      "alias": "u100",
      "rank": "order"
    },
    ...
  },
  "biogeo16": {
    "continental": {
      "name": "continental biogeographic region",
      "display_name": "Continental",
      "alias": "continental",
      "rank": null
    },
    ...
  }
},
"indicator_units": {
  "landcover": "class",
  "landuse": "class",
  "biogeo16": "class"
}
```

`biogeo16` is not ranked, so its rank is always `lowest`.

## Inspect the dataset

Any `.parquet` file produced along this pipeline — the selections, the compiled dataset, or any
dataframe from [ML preprocessing][ml_preprocess] — can be inspected with the
`inspect_pandas_dataset` process: column names and units, a sample of the data, and the
dataframe's full size.

**Process file**: `project_lucas_2009/process/pandas/inspect_pandas_df.json`

```json
{
  "process": [
    {
      "process": "inspect_pandas_dataset",
      "parameters": {
        "project_root_fp": "./project_lucas_2009/data/compiled/lucas_2009_10nm",
        "parquet_file": "lucas_2009_10nm.parquet",
        "column_array": "",
        "max_rows": 10
      }
    }
  ]
}
```

See [Inspect dataset][inspect_dataset] for the full parameter reference and an example run.

## Plot indicators and covariates

`plot_indicators` reads both `indicator_array` and `covariate_array` from the selection in
`project_root_fp`.

**Process file**: `project_lucas_2009/process/plot/plot_indicator.json`

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

Numeric indicators get a boxplot and a histogram. `single_canvas_plot` draws all indicators on one
canvas, `individual_canvas_plot` one canvas per indicator. Saved to `<project_root_fp>/plot/`, e.g.
`boxplot_all-indicators_z_log.png` and `histogram_all-indicators_z_log.png` (the suffix names the
standardisation and transformation), or `boxplot_<indicator>.png` per indicator.

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

**Nominal columns** (classes) can't be histogrammed or transformed. They are drawn as horizontal
bars of class counts: largest on top, at most `max_classes` (default 15), the rest summed as
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
- `hatch`: `/`, `\`, `|`, `-`, `+`, `x`, `o`, `O`, `.`, `*`; double a character for denser
  hatching
- `fill: false` draws the outline only

Defaults: landcover olivedrab `//`, landuse goldenrod `\\`, biogeo16 teal `xx`, all at alpha 0.6.
The transformation and standardisation defaults for the three covariates are `false`
(`lucas/default/transform/*.json`); non-numeric columns are skipped anyway.

`targetfeaturesymbols` is the same file used for [indicator units](#indicator-units) — in the plot
it only controls color, label and axis unit; no translation happens on the plotted values.

## Spectra plots

The four spectra plots below read the spectra selection: all four process files set

```json
"project_root_fp": "./project_lucas_2009/data/spectra/foss-xds-rca/ds2/lucas_eu_2009"
```

They are cells in both `select_spectra_only.ipynb` and `explore_select_data.ipynb`.

## Plot spectra and scatter correction options

Overlays the raw spectra against each available scatter-correction scaler (SNV, MSC, ...) so you
can visually compare them before picking one for [ML preprocessing][ml_preprocess].

**Process file**: `project_lucas_2009/process/plot/plot_spectra_scattercorrection_scalers.json`

## Plot spectra and scaling method options

Same idea, for the scaling methods (meancentring, autoscaling, paretoscaling, poissonscaling).

**Process file**: `project_lucas_2009/process/plot/plot_spectra_scaling_methods.json`

## Plot spectra

A plain spectra plot of the selected dataset, no processing applied.

**Process file**: `project_lucas_2009/process/plot/plot_spectra.json`

## Plot filter spectra

Overlays the raw spectra against each available smoothing filter (moving-average, gauss,
Savitzky-Golay, lowess).

**Process file**: `project_lucas_2009/process/plot/plot_filter_spectra.json`

## Next step

Proceed to [ML preprocessing][ml_preprocess] to clean and transform the compiled dataset
(`project_root_fp: ./project_lucas_2009/data/compiled/lucas_2009_10nm`).

[ml_preprocess]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_preprocess/
[observations_explained]: {{ site.baseurl }}/lucas_2009/observations_explained/
[inspect_dataset]: {{ site.baseurl }}/lucas_2009/machine_learning/inspect_dataset/
[foreign_key_explained]: {{ site.baseurl }}/lucas_2009/foreign_key_explained/#adding-a-new-unit-translation
[auto_naming_explained]: {{ site.baseurl }}/lucas_2009/auto_naming_explained/
[utility_explained]: {{ site.baseurl }}/lucas_2009/utility_explained/
[data_cleaning]: {{ site.baseurl }}/lucas_2009/prepare_data/#data-cleaning
[lucas_license]: https://esdac.jrc.ec.europa.eu/projects/LUCAS/Documents/LUCAS_SOIL_LIC_AGR_final_for_web.pdf
