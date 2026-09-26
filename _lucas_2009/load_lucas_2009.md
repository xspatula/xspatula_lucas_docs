---
title: "Load LUCAS 2009"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/load_lucas_2009/
author_profile: false
---

Loads the full LUCAS 2009 campaign — sampling log, geolocations, samples, lab-measured soil
properties, spectral observations, land cover/use and biogeographic region (BIOGEO16) — from the
files `lucas_2009_to_xspatula.py` generated.

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/load_LUCAS_2009.ipynb`

Run these fourteen cells, each a `job_file` pointing at one of the generated job files under
`import_data/LUCAS_2009/`, **in this fixed order** — later steps depend on earlier ones by foreign keys:

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
| 9 | Manage LUCAS 2009 landscape observation logs | `job_LUCAS_2009_observation_log_landscape.json` | [Observations explained (landscape)] |
| 10 | Manage LUCAS 2009 land cover | `job_LUCAS_2009_land_cover.json` | [Observations explained (landscape)] |
| 11 | Manage LUCAS 2009 land use | `job_LUCAS_2009_land_use.json` | [Observations explained (landscape)] |
| 12 | Manage EEA biogeographic regions 2016 sampling log | `job_LUCAS_2009_sampling_log_biogeo.json` | [Observations explained (BIOGEO16)] |
| 13 | Manage EEA biogeographic regions 2016 observation log | `job_LUCAS_2009_observation_log_biogeo.json` | [Observations explained (BIOGEO16)] |
| 14 | Manage EEA biogeographic regions 2016 observations | `job_LUCAS_2009_observation_biogeo.json` | [Observations explained (BIOGEO16)] |

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default), then run the
notebook top to bottom. Every step here is documented in full — job files, generated process files, and
parameter tables — in [Samples explained] and [Observations explained], which are reference
pages you don't need to read to complete the walkthrough.

### BIOGEO16 — a separate campaign

Cells 12–14 attach one of the eight EEA biogeographic regions (e.g. `continental`,
`mediterranean`, `boreal`) to each LUCAS point, taken from the `BIOGEO16` column of
`LUCAS-Master-Grid.csv` and joined by `POINT_ID`. BIOGEO16 is not part of the LUCAS 2009 campaign
— it is an EEA compilation registered as its own dataset and campaign (`biogeo16`, provision
`compilation`, fixed date 2016-03-31) in [Insert dataset metadata][insert_dataset_meta]. That is
why it gets its own sampling log (cell 12) instead of reusing `lucas_eu_2009`.

The biogeo16 sampling and observation logs are identical for LUCAS 2009 and
[LUCAS 2015][lucas_2015] — whichever campaign you load second simply finds them already in place.

Points without a match in the master grid (or with `NA`/`Outside` as value) are skipped, and the
script reports how many. Land cover/use (cells 9–11) and BIOGEO16 are only generated for points
from `LUCAS.SOIL_corr.csv` — the `SoilAttr_*.dbf` complement files carry neither.

## After this notebook

The LUCAS 2009 campaign is fully loaded: sampling log, one geolocation and sample per point, one
lab observation and one spectral observation per sample, plus land cover, land use and
biogeographic region per point. See [Dataset metadata explained], [Samples
explained], and [Observations explained] for what the resulting tables look like in more depth.

You can also continue directly with the [Machine Learning - Explore & Select Data][machine_learning]
section, or load the next campaign, [LUCAS 2015][lucas_2015].

[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[Samples explained]: /lucas_2009/samples_explained/
[Observations explained]: /lucas_2009/observations_explained/
[Observations explained (lab)]: /lucas_2009/observations_explained/#laboratory-observations
[Observations explained (spectra)]: /lucas_2009/observations_explained/#spectral-observations
[Observations explained (landscape)]: /lucas_2009/observations_explained/#landscape-observations
[Observations explained (BIOGEO16)]: /lucas_2009/observations_explained/#biogeographic-region-biogeo16-observations
[Dataset metadata explained]: /lucas_2009/dataset_meta_explained/
[lucas_2015]: /lucas_2015/
[machine_learning]: /lucas_2009/machine_learning/explore_select_data/
