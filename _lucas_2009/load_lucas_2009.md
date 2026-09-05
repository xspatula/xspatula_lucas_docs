---
title: "Load LUCAS 2009"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/load_lucas_2009/
author_profile: false
---

Runs the full LUCAS 2009 campaign — sampling log, geolocations, samples, lab-measured soil
properties, and spectral observations — from the files `lucas_2009_to_xspatula.py` generated.

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/load_LUCAS_2009.ipynb`

The notebook's own first code cell, **"Manage LUCAS 2009 campaign"**, is no longer part of this
walkthrough — skip it. The campaign record is already inserted by
[Insert dataset metadata][insert_dataset_meta] before you get here, and the generator script no
longer produces the files that cell expects.

Run these eight cells, each a `job_file` pointing at one of the generated job files under
`import_data/LUCAS_2009/`, **in this fixed order** — later steps depend on earlier ones by foreign
key:

| # | Cell | Job file | Detail |
|---|---|---|---|
| 1 | Manage LUCAS 2009 sampling log | `job_LUCAS_2009_sampling_log.json` | — |
| 2 | Manage LUCAS 2009 geolocations | `job_LUCAS_2009_geolocation.json` | [Samples explained] |
| 3 | Manage LUCAS 2009 samples | `job_LUCAS_2009_sample.json` | [Samples explained] |
| 4 | Manage LUCAS 2009 laboratory observation log | `job_LUCAS_2009_observation_log_lab.json` | [Observations explained (lab)] |
| 5 | Manage LUCAS 2009 laboratory observations | `job_LUCAS_2009_observation_lab.json` | [Observations explained (lab)] |
| 6 | Manage LUCAS 2009 spectrometer | `job_LUCAS_2009_spectrometer.json` | [Observations explained (spectra)] |
| 7 | Manage LUCAS 2009 spectra observation logs | `job_LUCAS_2009_observation_log_spectra.json` | [Observations explained (spectra)] |
| 8 | Manage LUCAS 2009 spectra observations | `job_LUCAS_2009_observation_spectra.json` | [Observations explained (spectra)] |

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default), then run the
notebook top to bottom, skipping only the "Manage LUCAS 2009 campaign" markdown/code cell pair
near the top. Every step here is documented in full — job files, generated process files, and
parameter tables — in [Samples explained] and [Observations explained], which are reference
pages you don't need to read to complete the walkthrough.

## After this notebook

The LUCAS 2009 campaign is fully loaded: sampling log, one geolocation and sample per point, one
lab observation and one spectral observation per sample. See [Dataset metadata], [Samples
explained], and [Observations explained] for what the resulting tables look like in more depth.

[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[Samples explained]: /lucas_2009/samples_explained/
[Observations explained]: /lucas_2009/observations_explained/
[Observations explained (lab)]: /lucas_2009/observations_explained/#laboratory-observations
[Observations explained (spectra)]: /lucas_2009/observations_explained/#spectral-observations
[Dataset metadata]: /dataset_meta/
