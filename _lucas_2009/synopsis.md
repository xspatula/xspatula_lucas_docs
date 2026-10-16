---
title: "LUCAS 2009"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/
author_profile: false
---

This is the full walkthrough for first loading the LUCAS 2009 soil sampling campaign into an Xspatula database: registering and downloading the source data, generating the JSON import files, and running the notebook that inserts everything. Once that is completed, you can continue with the Machine Learning pipeline.

## Prerequisites

- A running Xspatula database — see [Setup DB][setup_db] on the core docs site if you don't have one yet.
- The `xspatula_lucas` python integrated database cloned locally, with its conda environment created — see
  the [repository README](https://github.com/xspatula/xspatula_lucas).
- A [scheme file](https://xspatula.github.io/xspatula_core_docs/framework/scheme_file/) pointing at your database, e.g. `xspatula_lucas/lucas/scheme_lucas.json`.

## The pipeline, end to end

### Data loading pipeline

```
1. Download   LUCAS.SOIL_corr.csv (+ complements) from ESDAC, LUCAS-Master-Grid.csv from Eurostat
2. Prepare    lucas_2009_to_xspatula.py  →  generates job/pilot/process files
3. Insert     insert_lucas_data.ipynb, block 1  →  general, observation, landscape utility catalogues
4. Insert     insert_lucas_data.ipynb, block 2  →  data source, person, dataset, campaign
5. Load       insert_lucas_data.ipynb, block 3  →  LUCAS 2009: sampling log, geolocation, sample,
                                                    observation logs, observations,
                                                    land cover/use, BIOGEO16
   (Load      insert_lucas_data.ipynb, block 4  →  LUCAS 2015, same sequence)
```
Step 1 is getting the required LUCAS source data and is not supported by the framework; see
[Prepare data][prepare_data] for the download list.

Step 2 prepares the downloaded LUCAS data from step 1.

Steps 3–5 are the numbered blocks of one notebook,
`xspatula_lucas/lucas/import_data/insert_lucas_data.ipynb`. Run them in this order — later blocks
have foreign-key dependencies on earlier ones (utility catalogues first, since even the dataset
metadata references them). The pages of this site name each cell by its number and title, e.g.
**3.4.1 Load LUCAS 2009 laboratory observation log**.

**Running a part of the notebook.** The notebook runs clean from top to bottom and loads the whole
database, LUCAS 2009 and 2015, in a single run. To skip a cell, remove the `#` in front of its
first line, so that it reads

```python
%%script false --no-raise-error
```

The cell is then not executed. Put the `#` back to run it again. To load only LUCAS 2009, for
example, switch off every cell in block 4.

### Machine learning pipeline

```
1. Explore & select   explore_select_data.ipynb  →  browse the loaded data, pull a working subset
                                                      to local disk, plot it
2. Preprocess          ml_preprocess.ipynb        →  clean, transform, and select bands, saving
                                                      each result as a new .parquet/.json pair
3. Model                ml_model.ipynb             →  train and evaluate regressors against any
                                                      of the datasets step 2 produced
```

Unlike the data loading pipeline above, only the three macro-steps are fixed — data must be
selected before it can be preprocessed, and preprocessed (or not — `"raw"` is also a valid input)
before it can be modeled. *Within* each notebook, the individual cells don't have to run in a
fixed order or run at all: which scaling, scatter correction, filtering, or band-selection etc. steps you chain together, and in what sequence, is a modeling choice that depends on your data
and hypothesis, not a requirement of the framework.

## Pages in this section

**Required, in order, to load the campaign:**

1. [Prepare data][prepare_data] — download the CSV, run the generator script, inspect its output
2. [Insert utility][insert_utility] — the lookup catalogues every other insert depends on
3. [Insert dataset metadata][insert_dataset_meta] — data source, person, dataset, campaign
4. [Load LUCAS 2009][load_lucas_2009] — the full LUCAS 2009 campaign: sampling log, samples, lab and spectral observations logs and observations, land cover/use and biogeographic region (BIOGEO16)

LUCAS 2015 follows the same four steps, with block 4 of the notebook — see [LUCAS 2015][lucas_2015] for what differs.

**Reference — not required, only if you want the detail:**

- [Dataset metadata explained][dataset_meta_explained] — data source, person, dataset, and
  campaign tables, job files, and parameters
- [Utility explained][utility_explained] — exactly which catalogue tables each of the four
  utility cells (1.1–1.4) inserts
- [Utility → inherit and auto][utility_inherit_auto] — the `inherit`/`auto` special values a
  couple of this project's spreadsheets use
- [Foreign key explained][foreign_key_explained] — the `xxx_id__yyy` foreign-key resolver's
  fallback mechanism, and its two uses, `unit_translate` and the land cover/use parents
- [Samples explained] — geolocation and sample tables, job files, and parameters
- [Observations explained] — spectrometer, observation log, and observation tables, job files,
  and parameters

**Machine learning — once the campaign is loaded:**

- [Explore & select data][explore_select_data] — browse and pull a working subset to local disk
- [ML preprocessing][ml_preprocess] — clean, transform, and select bands; the full catalogue of
  available algorithms and how output is chained
- [ML modeling][ml_model] — train and evaluate regressors, and the full list of available Machine Learning models

[setup_db]: https://xspatula.github.io/xspatula_core_docs/setup_db/
[prepare_data]: {{ site.baseurl }}/lucas_2009/prepare_data/
[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[insert_utility]: {{ site.baseurl }}/lucas_2009/insert_utility/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[dataset_meta_explained]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/
[utility_explained]: {{ site.baseurl }}/lucas_2009/utility_explained/
[utility_inherit_auto]: {{ site.baseurl }}/lucas_2009/utility_inherit_auto_explained/
[foreign_key_explained]: {{ site.baseurl }}/lucas_2009/foreign_key_explained/
[Samples explained]: {{ site.baseurl }}/lucas_2009/samples_explained/
[Observations explained]: {{ site.baseurl }}/lucas_2009/observations_explained/
[explore_select_data]: {{ site.baseurl }}/lucas_2009/machine_learning/explore_select_data/
[ml_preprocess]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_preprocess/
[ml_model]: {{ site.baseurl }}/lucas_2009/machine_learning/ml_model/
[lucas_2015]: {{ site.baseurl }}/lucas_2015/
