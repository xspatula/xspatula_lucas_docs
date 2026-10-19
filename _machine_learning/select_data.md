---
title: "Select Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/machine_learning/select_data/
author_profile: false
---

The first step of the machine learning pipeline: decide which samples and which data you need, check
that there are enough of them, and pull them out of the database onto local disk. This page covers the
selection. [Parquet files][parquet_files] covers what is saved and how to inspect and compile it, and
[Plot data][plot_data] how to look at it.

Spectra, indicators (the targets you want to predict) and covariates (e.g. land cover, land use,
biogeographic region) are selected **separately**, each into its own tag-named Parquet file. A compile
step then joins them on `sample_id` into the dataset that every later ML step works on.

The extracted files contain LUCAS data and fall under the
[LUCAS_SOIL licence agreement][lucas_license]: keep them within your own organisation. Published
results must be aggregated so that no individual point can be identified, and maps may not show
detectable sample locations (art. 5).

## Workflow

The notebook `select_indicator_only.ipynb` runs the canonical sequence, one cell per step (process files
under `project_lucas_2009/process/`):

1. **Identify** (`select/select_list_lucas_indicator.json`, process `identify_indicator`): how many
   samples there are, per class filter entry, spectra sensor and indicator. Nothing is saved. See
   [Identify indicators and sample counts](#identify-indicators-and-sample-counts).
2. **Select** (`select/select_save_lucas2009_indicator.json`, process `select_indicator_only`): saves a
   `.parquet` + `.json`. See [Parquet files][parquet_files].
3. **Inspect** (`inspect/inspect_latest_indicator_df.json`): shows what was just saved. See
   [Inspect the dataset][inspect].
4. **Plot** (`plot/plot_lucas2009_indicators.json`). See [Plot data][plot_data].

Spectra and covariates follow the same pattern.

## Notebooks

All notebooks are under `xspatula_lucas/lucas/project_lucas_2009/`. Set
`scheme_file = '../scheme_lucas.json'` in the setup cell (already the default).

| Notebook | Cells (process files under `project_lucas_2009/process/`) |
|---|---|
| `select_indicator_only.ipynb` | `select/select_list_lucas_indicator.json`, `select/select_save_lucas2009_indicator.json`, `inspect/inspect_latest_indicator_df.json`, `plot/plot_lucas2009_indicators.json` |
| `select_spectra_only.ipynb` | `select_lucas_spectra_2009_10nm.json`, then the four spectra plots |
| `select_landscape_covariate_only.ipynb` | `select/select_list_lucas_indicator.json`, `select/select_save_lucas2009_covariate.json`, `plot/plot_covariate.json` |
| `explore_select_data.ipynb` | the identify cell, the all-in-one `select_lucas_2009_10nm.json`, and all plots |

Run the three `*_only` notebooks, then [compile][compile] the selections. `explore_select_data.ipynb` is
the older all-in-one route (see [All in one: select_spectra](#all-in-one-select_spectra)).

## The select processes

One function does all selections; the process sets the **role** of the output (`SELECT_ROLE_D` in
`src/ai4sh/select.py`):

| Process | Role | Saves |
|---|---|---|
| `identify_indicator` | — | prints sample counts per indicator, spectra sensor and filter class; saves nothing |
| `select_spectra_only` | `spectra` | spectra only |
| `select_indicator_only` | `indicator` | numeric (and nominal) indicators |
| `select_covariate_only` | `covariate` | covariates, e.g. land cover, land use, biogeo16 |
| `select_spectra` | `all` | spectra + indicators (+ covariates) in one file |
| `compile_dataset` | — | joins saved selections on `sample_id`, see [Compile][compile] |

The process decides the role, not the data: pH can be an indicator in one file and a covariate in
another. Whether a name is a numeric measurement, a nominal classification or a landscape class is
looked up in the database.

The old process `select_indicator` and its parameters `dataset_name`/`campaign_name` no longer exist.

### Scope parameters

All select processes take the same scope parameters:

| Parameter | Description |
|---|---|
| `dataset` | Dataset name, tag or alias, e.g. `lucas` |
| `campaign` | Optional: one campaign (the place), e.g. `lucas_eu` |
| `sampling_log` | Optional: one sampling log (the survey round), e.g. `lucas_eu_2009` |
| `sample_array` | Optional: a list of sample names; the output sub path then ends with `sample_select` |
| `min_profile` / `max_profile` | Depth profile filter, cm |

Each can be given as name, tag or alias. If a value matches more than one record, the process stops and
asks for the tag, or the parent (dataset/campaign). See
[Auto naming and tags explained][auto_naming_explained] for the LUCAS names and tags.

Each process skips silently if its output already exists, unless `overwrite: true`.

## Identify indicators and sample counts

Before saving anything, check how many samples you would get.

**Process file**: `project_lucas_2009/process/select/select_list_lucas_indicator.json`

```json
{
  "process": [
    {
      "process": "identify_indicator",
      "overwrite": false,
      "parameters": {
        "dataset": "lucas",
        "campaign": "",
        "sampling_log": "",
        "min_profile": 0,
        "max_profile": 20,
        "texture_class_array": [],
        "texture_classification": "soil texture usda",
        "landuse_array": [],
        "landcover_array": [],
        "biogeo16_array": [],
        "juxtaposition_array": [],
        "composition_array": [],
        "proximity_array": [],
        "sample_source": "",
        "min_samples": 30
      }
    }
  ]
}
```

Parameters: the scope parameters, every [class filter](#class-filters), `sample_source`
([below](#reuse-a-selections-samples-sample_source)) and `min_samples` (default 30; counts below it are
flagged ⚠️).

Output, LUCAS 2009 unfiltered (dataset `lucas`, all sampling logs; list shortened):

```
    Querying available data for dataset "land use and coverage area frame survey (lucas) topsoil data"

    Samples in scope: 399

    Spectral sensors (provisions):
        - foss xds rapid content analyzer: 150 samples

    Indicators (samples with a value):
        - carbonate (caco3) [g*kg^-1]: 399
        - cation exchange capacity (cec) [cmolc*kg^-1]: 149
        - clay (<0.002 mm) (clay) [weight percent]: 382
        ...
        - soil texture class usda (soil texture usda) [class]: 382
        - total nitrogen (n-tot) [g*kg^-1]: 392
```

- Counts are samples with a value. Missing-value codes (-9999, -999, -888) are not counted. The range
  screening ([Invalid values][invalid_values]) can lower the saved count further.
- Nominal indicators get the unit `class`.
- The depth filter (`min_profile`/`max_profile`) applies here too.

### When the selection is too narrow

This is why the counts exist. LUCAS 2009, texture + land cover:

```
    Querying available data for dataset "land use and coverage area frame survey (lucas) topsoil data" / sampling_log "lucas european union 2009"
    Filter texture: ['loam', 'silt loam', 'clay']
    Filter texture_classification: soil texture class usda
    Filter landcover: ['cropland', 'grassland']

    Samples in scope: 13  ⚠️  < 30

    Samples per class_texture:
        - loam: 6  ⚠️  < 30
        - silt loam: 5  ⚠️  < 30
        - clay: 2  ⚠️  < 30

    Samples per class_landcover:
        - cropland: 8  ⚠️  < 30
        - grassland: 5  ⚠️  < 30

    Spectral sensors (provisions):
        - foss xds rapid content analyzer: 13 samples  ⚠️  < 30
```

13 samples is too few for a model, let alone one per class. Broaden the selection: drop a filter, add
classes, go up a rank (`cropland` instead of `wheat`), or widen the scope. The same filters over the whole
`lucas` dataset (both sampling logs) give 105 samples: silt loam 52, loam 51, clay 2; grassland 78,
cropland 27. Clay is still far too thin, so drop it.

## Class filters

A model can be general (all samples) or specific (one model per class, e.g. per texture class or land
cover). Splitting at selection means each class can get its own preprocessing (indicator transformation,
spectral enhancement), which is lost if the split happens only at the modelling stage. The parquet keeps
the class ([Columns][columns]), so a general model can still be coloured or split by class later.

| Parameter | Filters on | Column in parquet | Name part |
|---|---|---|---|
| `texture_class_array` (+ `texture_classification`) | soil texture class of the sample (observation) | `class_texture` | `tx-` |
| `landcover_array` | land cover at the sample position | `class_landcover` | `lc-` |
| `landuse_array` | land use at the sample position | `class_landuse` | `lu-` |
| `biogeo16_array` | biogeographic region at the sample position | `class_biogeo16` | `bg-` |
| `juxtaposition_array`, `composition_array`, `proximity_array` | sample setting | `class_juxtaposition`, `class_composition`, `class_proximity` | `jx-`, `cp-`, `px-` |

Common rules:

- Each filter is a list. A sample passes if its class is any entry in the list (OR within a filter).
  Several filters combine with AND.
- An entry is a class name, tag, alias or display name, case-insensitive.
- An unknown entry stops the process and lists the valid classes. Two entries naming the same class stop
  it too.
- Samples without a recorded class are excluded by that filter. Coverage depends on what has been
  imported. Current DB: land cover and land use for 50 of 199 samples in 2009 and 200 of 200 in 2015;
  biogeo16 for 50 (2009) and 51 (2015); texture for 186 (2009) and 196 (2015). Check with
  `identify_indicator`.
- Filters restrict **every** query (keys, spectra, indicators, covariates): they are applied once, in the
  scope of the selection. A filtered spectra selection and a filtered indicator selection therefore start
  from the same samples (before each drops samples without data).

### Soil texture

`texture_class_array` keeps samples of the listed texture classes, under the classification given in
`texture_classification`:

- LUCAS records two texture classifications as nominal indicators: `soil texture class usda` (alias
  `soil texture usda`, the default) and `soil texture class iuss` (alias `soil texture iuss`).
  `texture_classification` takes the name or alias.
- Texture is an observation of the sample, unlike the three landscape classifications below.

USDA classes, LUCAS samples (all sampling logs, current DB):

| Class | Samples | Class | Samples |
|---|---|---|---|
| clay | 124 | sandy clay | 0 |
| clay loam | 64 | sandy clay loam | 4 |
| loam | 121 | sandy loam | 66 |
| loamy sand | 23 | silt | 3 |
| sand | 6 | silt loam | 173 |
| silty clay | 14 | silty clay loam | 56 |

The thin classes (sand, silt, sandy clay loam) show that one model per USDA class is rarely possible.
Group them instead, e.g. coarse / medium / fine. Grouping in one run is a planned feature. Today, run one
selection per group, with the group's classes in the list.

IUSS classes: clay, clay loam, light clay, loam, loamy sand, sand, sandy clay, sandy clay loam, sandy
loam (silt loam, silty clay and silty clay loam exist in the lookup with 0 samples).

The "major group" classifications (`... major group`: heavy clayey / light clayey / loamy / sandy soils)
exist as indicators but have no LUCAS data. Once filled, they would be the natural coarse grouping.

### Land cover and land use

`landcover_array` and `landuse_array` are hierarchical: order > family > genus (see
[Utility explained][utility_explained] and [Covariates and ranks](#covariates-and-ranks)). Land cover has
10 orders, 26 families and 73 genera; land use 4 orders, 17 families and 26 genera (current
`landscape_utility`).

- **No rank parameter.** An entry can be at any rank and includes everything below it: `cropland`
  (order) = `cultivated` + `agroforest` (families) + all their genera. `["wheat", "barley"]` selects
  just those two genera. The rank is whatever you name.
- Entries in one filter must not overlap. `["primary production", "agriculture"]` is an error, as
  agriculture lies within primary production:
  `❌ ERROR: landuse_array entries overlap (one lies within the other): agriculture and primary production`
- `class_landcover` holds **the filter entry the sample matched**, not the recorded class. A wheat sample
  under the filter `["cropland", "grassland"]` gets `cropland`. That is the value to split or colour by.
- Check: `landuse_array: ["primary production"]` gives 220 samples, and its five families together give
  the same 220.
- Land cover and land use describe the position, not the sample. They are joined by the sample's geotag.
  If a position has several observations, the one from the sample's own sampling log wins, then the most
  recent. This is the same rule as for the covariate selection.

Orders, as examples (full lists in `landcover.xlsx` and `landuse.xlsx`):

| Classification | Orders (tags) |
|---|---|
| Land cover | `artificial`, `cropland`, `woodland`, `shrubland`, `grassland`, `bare-land`, `water`, `wetland`, `ice-snow`, `unclassified` |
| Land use | primary production (`prim-prod`), industrial production (`industry`), infrastructure services (`infrastructure`), unused |

Families of grassland: `grass-l-trees`, `grass-l-no-trees`, `spontan`.

**Filter or covariate?** A filter decides *which samples* are selected and which group each belongs to
(`class_landcover`). A covariate (`covariate_array` with `covariate_rank`) adds the class as a *column*,
rolled to a rank, as a model input or for plotting. Use the filter to split, the covariate to describe.

### Biogeographic region

`biogeo16_array` is flat (no ranks). Tags: `alpine`, `anatolian`, `arctic`, `atlantic`, `black-sea`,
`boreal`, `continental`, `macaronesia`, `mediterranean`, `outside`, `pannonian`, `steppic`, `na`. In the
current DB only continental (53), alpine (43) and pannonian (5) occur, so a biogeo16 filter on the LUCAS
subset keeps at most about a quarter of the samples.

### Sample setting

`juxtaposition_array`, `composition_array` and `proximity_array` filter on how the sample was taken.
Every LUCAS sample has the same values (no juxtaposition, composite, general), so these filters do
nothing for LUCAS. They matter for AI4SoilHealth: see [AI4SH Select data][ai4sh_select]. A juxtaposition
name that exists under several setting systems matches all of them.

## Reuse a selection's samples (`sample_source`)

How to get the spectra for exactly the samples of a filtered indicator selection:

- Set `sample_source` to a saved parquet (relative to the project root, `.parquet` optional).
- Its `sample_id`s restrict the scope. Its `class_*` columns and its `sample_filter` are inherited, so
  the spectra file gets the same filter part in its name.
- Don't repeat the filter instead. The indicator selection may already have dropped samples (no value,
  `data_range`). `sample_source` gives exactly the same samples, and `sample_id` is the database key,
  unique across datasets.

**Process file**: `project_lucas_2009/process/select/select_lucas_spectra_2009_10nm_landcover.json`

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
        "output_root_fp": "./project_lucas_2009/data",
        "sample_source": "./project_lucas_2009/data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20__lc-cropland-grassland"
      }
    }
  ]
}
```

The spectra are saved as `400-2500_10__lc-cropland-grassland.parquet`, with the same samples as the
indicator file. In a test run, 37 indicator samples gave 37 spectra.

## Select spectra

Queries the database, resamples the spectra to a target wavelength grid, optionally converts the signal,
and saves the result.

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
| `begin_wavelength` / `end_wavelength` / `output_bandwidth` | Target wavelength grid, resampled from the spectrometer's native `wavelength_array` (see [Observations explained][observations_explained]), not assumed to match it |
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
| `indicator_array` | The lab indicators to select. Each entry is resolved by indicator name or alias: `ph-water` here and `ph-h2o` in the observation records (see [Observations explained][observations_explained]) are the same indicator |
| `data_range` | Optional JSON `{indicator: {min, max}}` to filter values; empty = no filter |
| `targetfeaturesymbols` | Which `unit` to save each indicator in, see [Indicator units][indicator_units]. Optional, defaults to `"default"` |

If a requested indicator has no data in the database, the process lists what is available and asks
whether to continue without it.

**Output**: `data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet`
(+ `.json`).

The filtered variant, `select/select_save_lucas2009_indicator_landcover.json`, adds every class filter
key (all empty except `"landcover_array": ["cropland", "grassland"]`) and `output_suffix`. Its output is
`cec-clay-silt-sand-c-org-n-tot-ph-water_0-20__lc-cropland-grassland.parquet` (see
[Output names][output_names]).

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

Land cover, land use and biogeo16 describe the geolocation, not the sample; they are joined to the
samples by position. `covariate_rank` is explained in [Covariates and ranks](#covariates-and-ranks).

**Output**: `data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet`
(+ `.json`).

## All in one: select_spectra

`select_spectra` (role `all`) saves spectra, indicators and covariates in one file, without a compile
step. Used by `explore_select_data.ipynb`.

**Process file**: `project_lucas_2009/process/select_lucas_2009_10nm.json`: the parameters of
`select_lucas_spectra_2009_10nm.json` plus `indicator_array` and `data_range`.

Default output name: `{begin}-{end}_{bandwidth}+{indicator aliases}[+{covariate aliases}]_{min}-{max}`
under `data/all/foss-xds-rca/ds2/lucas_eu_2009/`.

## Covariates and ranks

Land cover and land use are ranked order > family > genus (see [Utility explained][utility_explained]).
`covariate_rank` sets the rank of the selected classes:

- `order`, `family`, `genus`, or `lowest` (the rank recorded, the default).
- One rank for all covariates (`"order"`), or JSON per covariate:
  `{"landcover": "family", "landuse": "order"}`. Covariates not listed keep `lowest`.

A recorded class is rolled up through its parents to the requested rank. A class recorded at a coarser
rank than requested is kept as recorded, with a warning giving the count:

```
⚠️  landuse: 3 of 50 samples recorded at a coarser rank than family, kept as recorded
```

The parquet stores the class **tag**; the companion JSON gets `covariate_rank` and `class_labels` (see
[The companion JSON][companion_json]). `biogeo16` is not ranked, so its rank is always `lowest`.

A covariate rank is not a filter: see [Filter or covariate?](#land-cover-and-land-use) above.

## Next step

[Parquet files][parquet_files]: what the selection saved, how it is named, and how to inspect and
compile it.

[parquet_files]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/
[plot_data]: {{ site.baseurl }}/lucas_2009/machine_learning/plot_data/
[inspect]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#inspect-the-dataset
[compile]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#compile-the-project-dataset
[columns]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#columns
[output_names]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#output-names
[companion_json]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#the-companion-json
[indicator_units]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#indicator-units
[invalid_values]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#invalid-values
[observations_explained]: {{ site.baseurl }}/lucas_2009/observations_explained/
[auto_naming_explained]: {{ site.baseurl }}/lucas_2009/auto_naming_explained/
[utility_explained]: {{ site.baseurl }}/lucas_2009/utility_explained/
[ai4sh_select]: https://xspatula.github.io/xspatula_ai4sh_docs/machine_learning/select_data/
[lucas_license]: https://esdac.jrc.ec.europa.eu/projects/LUCAS/Documents/LUCAS_SOIL_LIC_AGR_final_for_web.pdf
