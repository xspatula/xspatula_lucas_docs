---
title: "LUCAS 2009 Campaign"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/
author_profile: false
---

This is the full walkthrough for loading the LUCAS 2009 soil sampling campaign into an Xspatula
database: registering and downloading the source data, generating the JSON import files, and
running the notebooks that insert everything.

## Prerequisites

- A running Xspatula database — see [Setup DB][setup_db] on the core docs site if you don't have
  one yet.
- The `xspatula_lucas` Python package cloned locally, with its conda environment created — see
  the [repository README](https://github.com/xspatula/xspatula_lucas).
- A scheme file pointing at your database, e.g. `xspatula_lucas/lucas/scheme_lucas.json`.

## The pipeline, end to end

```
1. Download   LUCAS.SOIL_corr.csv from ESDAC (registration required)
2. Prepare    lucas_2009_to_xspatula.py  →  generates job/pilot/process files
3. Insert     insert_lucas_dataset_meta.ipynb  →  data source, person, dataset, campaign
4. Insert     insert_utility.ipynb            →  general, observation, landscape utility catalogues
5. Load       load_LUCAS.ipynb               →  campaign, sampling log, geolocation, sample,
                                                  lab observations, spectrometer, spectral observations
```

Steps 3–5 correspond to three separate notebooks, all under
`xspatula_lucas/lucas/import_data/`. Run them in this order — later steps have foreign-key
dependencies on earlier ones.

## Pages in this section

1. [Prepare data][prepare_data] — download the CSV, run the generator script, inspect its output
2. [Insert dataset metadata][insert_dataset_meta] — data source, person, dataset, campaign
3. [Insert utility][insert_utility] — the lookup catalogues every other insert depends on
4. [Load campaign][load_campaign] — the full campaign: sampling, samples, lab and spectral
   observations

For what the resulting database tables look like once loaded, see [Dataset metadata],
[Sample], and [Spectra] — those pages document the schema and parameters in more depth than the
step-by-step notebook walkthrough here.

## Scope note

`xspatula_ai4sh/lucas/prepare_lucas_data/` also ships `build_lucas_labdata_2009.py`, a separate
script. It is **not** part of this pipeline — ignore it.

[setup_db]: https://xspatula.github.io/xspatula_core_docs/setup_db/
[prepare_data]: /lucas_2009/prepare_data/
[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[insert_utility]: /lucas_2009/insert_utility/
[load_campaign]: /lucas_2009/load_campaign/
[Dataset metadata]: /dataset_meta/
[Sample]: /sample/
[Spectra]: /spectra/
