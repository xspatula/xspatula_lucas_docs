---
title: "LUCAS 2009"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/
author_profile: false
---

This is the full walkthrough for loading the LUCAS 2009 soil sampling campaign into an Xspatula
database: registering and downloading the source data, generating the JSON import files, and running the notebooks that insert everything.

## Prerequisites

- A running Xspatula database — see [Setup DB][setup_db] on the core docs site if you don't have one yet.
- The `xspatula_lucas` Python package cloned locally, with its conda environment created — see
  the [repository README](https://github.com/xspatula/xspatula_lucas).
- A scheme file pointing at your database, e.g. `xspatula_lucas/lucas/scheme_lucas.json`.

## The pipeline, end to end

```
1. Download   LUCAS.SOIL_corr.csv from ESDAC (registration required)
2. Prepare    lucas_2009_to_xspatula.py  →  generates job/pilot/process files
3. Insert     insert_utility.ipynb            →  general, observation, landscape utility catalogues
4. Insert     insert_lucas_dataset_meta.ipynb  →  data source, person, dataset, campaign
5. Load       load_LUCAS_2009.ipynb          →  sampling log, geolocation, sample,
                                                  observation logs, observations
```
Step 1 is getting the required LUCAS source data and is not supported by the framework.

Step 2 prepares the downloaded LUCAS data from step 1.

Steps 3–5 correspond to three separate notebooks, all under
`xspatula_lucas/lucas/import_data/`. Run them in this order — later steps have foreign-key
dependencies on earlier ones (utility catalogues first, since even the dataset metadata step references them).

## Pages in this section

**Required, in order:**

1. [Prepare data][prepare_data] — download the CSV, run the generator script, inspect its output
2. [Insert utility][insert_utility] — the lookup catalogues every other insert depends on
3. [Insert dataset metadata][insert_dataset_meta] — data source, person, dataset, campaign
4. [Load LUCAS 2009][load_lucas_2009] — the full LUCAS 2009 campaign: sampling log, samples, lab and spectral observations logs and observations

**Reference — not required, only if you want the detail:**

- [Samples explained] — geolocation and sample tables, job files, and parameters
- [Observations explained] — spectrometer, observation log, and observation tables, job files,
  and parameters

For dataset-level tables (data source, person, dataset, campaign), see [Dataset metadata] — that collection documents the schema and parameters behind step 3 above.

[setup_db]: https://xspatula.github.io/xspatula_core_docs/setup_db/
[prepare_data]: /lucas_2009/prepare_data/
[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[insert_utility]: /lucas_2009/insert_utility/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
[Samples explained]: /lucas_2009/samples_explained/
[Observations explained]: /lucas_2009/observations_explained/
[Dataset metadata]: /dataset_meta/
