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

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_data.ipynb`, block **3. LUCAS 2009 data**

Blocks 1 (utilities) and 2 (dataset metadata) must have run first. Block 3 has fourteen cells,
each a `job_file` pointing at one of the generated job files under `import_data/LUCAS_2009/`. Run
them **in this fixed order** — later cells depend on earlier ones by foreign keys:

| Cell | Job file | Detail |
|---|---|---|
| 3.1 Load LUCAS 2009 sampling log | `job_LUCAS_2009_sampling_log.json` | [Auto naming][auto_naming] |
| 3.2 Load LUCAS 2009 geolocations | `job_LUCAS_2009_geolocation.json` | [Samples explained (geolocation)] |
| 3.3 Load LUCAS 2009 samples | `job_LUCAS_2009_sample.json` | [Samples explained (sample)] |
| 3.4.1 Load LUCAS 2009 laboratory observation log | `job_LUCAS_2009_observation_log_lab.json` | [Observations explained (lab)] |
| 3.4.2 Load LUCAS 2009 laboratory observations | `job_LUCAS_2009_observation_lab.json` | [Observations explained (lab)] |
| 3.5.1 Load LUCAS 2009 spectrometer | `job_LUCAS_2009_spectrometer.json` | [Observations explained (spectra)] |
| 3.5.2 Load LUCAS 2009 spectra observation log | `job_LUCAS_2009_observation_log_spectra.json` | [Observations explained (spectra)] |
| 3.5.3 Load LUCAS 2009 spectra observations | `job_LUCAS_2009_observation_spectra.json` | [Observations explained (spectra)] |
| 3.6.1 Load LUCAS 2009 landscape observation logs | `job_LUCAS_2009_observation_log_landscape.json` | [Observations explained (landscape)] |
| 3.6.2 Load LUCAS 2009 land cover | `job_LUCAS_2009_land_cover.json` | [Observations explained (land cover/use)] |
| 3.6.3 Load LUCAS 2009 land use | `job_LUCAS_2009_land_use.json` | [Observations explained (land cover/use)] |
| 3.7.1 Load LUCAS 2009 BIOGEO16 sampling log | `job_LUCAS_2009_sampling_log_biogeo.json` | [BIOGEO16 sampling log] |
| 3.7.2 Load BIOGEO16 observation log | `job_LUCAS_2009_observation_log_biogeo.json` | [BIOGEO16 observation log] |
| 3.7.3 Load LUCAS 2009 biogeographic regions | `job_LUCAS_2009_observation_biogeo.json` | [BIOGEO16 observations] |

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default). Every step
here is documented in full — job files, generated process files, and parameter tables — in
[Samples explained] and [Observations explained], which are reference pages you don't need to
read to complete the walkthrough.

To load only LUCAS 2009, switch off the cells of block 4 (LUCAS 2015) with
`%%script false --no-raise-error`; see [the synopsis][synopsis_partial].

### BIOGEO16 — a separate campaign

Cells 3.7.1–3.7.3 attach one of the eight EEA biogeographic regions (e.g. `continental`,
`mediterranean`, `boreal`) to each LUCAS point, taken from the `BIOGEO16` column of
`LUCAS-Master-Grid.csv` and joined by `POINT_ID`. BIOGEO16 is not part of the LUCAS 2009 campaign
— it is an EEA compilation registered as its own dataset and campaign (`biogeo16 european union`,
provision `compilation`, fixed date 2016-03-31) in [Insert dataset metadata][insert_dataset_meta].
That is why it gets its own sampling log, `biogeo16 european union 2016` / `biogeo16_eu_2016`
(cell 3.7.1), instead of reusing `lucas european union 2009`. See
[Why BIOGEO16 has its own sampling log][biogeo_why].

The biogeo16 sampling and observation logs are identical for LUCAS 2009 and
[LUCAS 2015][lucas_2015] — whichever campaign you load second simply finds them already in place.

Points without a match in the master grid (or with `NA`/`Outside` as value) are skipped, and the
script reports how many. Land cover/use (cells 3.6.1–3.6.3) and BIOGEO16 are only generated for points
from `LUCAS.SOIL_corr.csv` — the `SoilAttr_*.dbf` complement files carry neither.

## After block 3

The LUCAS 2009 campaign is fully loaded: sampling log, one geolocation and sample per point, one
lab observation and one spectral observation per sample, plus land cover, land use and
biogeographic region per point. See [Dataset metadata explained], [Samples
explained], and [Observations explained] for what the resulting tables look like in more depth.

If you also load LUCAS 2015, do that first. Then [dump the database][synopsis_dump]: restoring a
dump takes minutes, the import takes hours. The dump is for your own use only; the
[LUCAS licence][synopsis_license] does not permit sharing it.

You can also continue directly with the [Machine Learning - Explore & Select Data][machine_learning]
section, or load the next campaign, [LUCAS 2015][lucas_2015].

[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[Samples explained]: {{ site.baseurl }}/lucas_2009/samples_explained/
[Samples explained (geolocation)]: {{ site.baseurl }}/lucas_2009/samples_explained/#geolocation
[Samples explained (sample)]: {{ site.baseurl }}/lucas_2009/samples_explained/#sample
[Observations explained (land cover/use)]: {{ site.baseurl }}/lucas_2009/observations_explained/#land-cover-and-land-use
[BIOGEO16 sampling log]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeo16-sampling-log
[BIOGEO16 observation log]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeo16-observation-log
[BIOGEO16 observations]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeo16-observations
[biogeo_why]: {{ site.baseurl }}/lucas_2009/observations_explained/#why-biogeo16-has-its-own-sampling-log
[auto_naming]: {{ site.baseurl }}/lucas_2009/auto_naming_explained/#a-record-without-a-name
[synopsis_partial]: {{ site.baseurl }}/lucas_2009/#data-loading-pipeline
[Observations explained]: {{ site.baseurl }}/lucas_2009/observations_explained/
[Observations explained (lab)]: {{ site.baseurl }}/lucas_2009/observations_explained/#laboratory-observations
[Observations explained (spectra)]: {{ site.baseurl }}/lucas_2009/observations_explained/#spectral-observations
[Observations explained (landscape)]: {{ site.baseurl }}/lucas_2009/observations_explained/#landscape-observations
[Observations explained (BIOGEO16)]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeographic-region-biogeo16-observations
[Dataset metadata explained]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/
[lucas_2015]: {{ site.baseurl }}/lucas_2015/
[machine_learning]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/
[synopsis_dump]: {{ site.baseurl }}/lucas_2009/#dump-once-restore-in-minutes
[synopsis_license]: {{ site.baseurl }}/lucas_2009/#lucas-license-the-data-may-not-be-shared
