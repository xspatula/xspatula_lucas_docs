---
title: "LUCAS 2015"
layout: single
sidebar:
  nav: "lucas_2015"
permalink: /lucas_2015/
author_profile: false
---

Loading the LUCAS 2015 soil sampling campaign follows the **same sequence as [LUCAS 2009][synopsis]**:
prepare → insert utility → insert dataset metadata → load. Only the source files and the script
that translates them differ. This page covers those differences and lists the load notebook's
cells. For the detail behind each step, follow the links back to the LUCAS 2009 pages.

## Prerequisites

- A running Xspatula database and the `xspatula_lucas` repository, as for
  [LUCAS 2009][synopsis].
- [Insert utility][insert_utility] and [Insert dataset metadata][insert_dataset_meta] completed.
  `campaign.xlsx` already registers `lucas_eu_2015` and `biogeo16` (see
  [Dataset metadata explained][dataset_meta_campaign]), and the utility spreadsheets already hold
  the `lucas-wetlab-2015` provision and the `@ec` indicator. If you ran those two notebooks before
  these rows existed, run them again.
- LUCAS 2009 does **not** have to be loaded first. But you need `LUCAS.SOIL_corr.csv` on disk
  for the texture backfill (below).

## 1. Download the source files

Register at [esdac.jrc.ec.europa.eu/projects/lucas](https://esdac.jrc.ec.europa.eu/projects/lucas)
and download the LUCAS 2015 topsoil data. LUCAS 2009 comes as one file with lab data, coordinates,
dates and spectra together. LUCAS 2015 splits them up. Place everything directly under one
directory (`CSV_PATH`):

| File | Content |
|---|---|
| `LUCAS_Topsoil_2015_20200323.csv` | Main file: one row per point with lab data, `LC1`/`LU1` and `Revisited_point`. **No coordinates, no dates** |
| `LUCAS_Topsoil_2015_20200323.dbf` | `Point_ID` → Long/Lat lookup, from the `LUCAS2015_topsoildata_20200323` package |
| `spectra/spectra_ <CC> .csv` | Spectra, one file per country, as downloaded. Usually **two scans per point** |
| `LUCAS-Master-Grid.csv` | The LUCAS master grid with its `BIOGEO16` column. It is the same file LUCAS 2009 uses |

## 2. Configure and run the script

**Path**: `xspatula_lucas/lucas/prepare_lucas_data/lucas_2015_to_xspatula.py`

It is configured the same way as the [2009 script][prepare_data]. The constants that differ:

| Constant | Purpose | Default |
|---|---|---|
| `CSV_PATH` | Directory holding the 2015 files above | a machine-specific path — **must be changed** |
| `OUTPUT_ROOT` | Where generated files land | `../import_data/LUCAS_2015` |
| `RECORDS` | Max rows from the main csv **and** max spectra scans in total | a small number for a test run — **set to `0` for the full campaign** |
| `LUCAS_2009_ROOT` | Directory holding `LUCAS.SOIL_corr.csv`, used for the texture backfill | a machine-specific path — **must be changed** |
| `CAMPAIGN_NAME` | Campaign name | `lucas_eu_2015` |
| `LAB_PROVISION` / `SPECTRA_PROVISION` / `LANDSCAPE_PROVISION` | Provisions for the three campaign observation logs | `lucas-wetlab-2015` / `foss xds rca` / `human interpretation` |
| `SPECTROMETER_SERIAL` | Same instrument as 2009, different serial | `lucas 2015` |
| `MISSING_DATE_TOKEN` | Representative survey date, see below | `20150801` |

```bash
cd xspatula_lucas/lucas/prepare_lucas_data
python3 lucas_2015_to_xspatula.py
```

It prints `OK:` for each of 13 steps and ends with `DONE`. The output tree under
`import_data/LUCAS_2015/` mirrors the [2009 output][prepare_data_output]: `process_lab/`,
`process_spectra/`, `process_landscape/`, `process_biogeo/` plus 14 `job_LUCAS_2015_*.json`
files.

## What the script does differently

- **Coordinates** come from the `.dbf` lookup, not the main csv. Geolocation names use the same
  format as 2009 (`<iso>_lucas@<POINT_ID>`). A point revisited in 2015 therefore resolves to the
  **same geolocation row** as in 2009: one physical location, one entity. Sample names
  (`<POINT_ID>@0-20`) are also identical, but samples are unique per sampling log, so 2009 and 2015
  samples never collide.
- **No dates anywhere** in the 2015 files. Fieldwork ran May–October 2015, so a representative
  midpoint, `2015-08-01`, is used for every `observed_at`/`sampled_at` (lab, spectra, landscape) and
  in filenames.
- **Two spectra per point.** Each scan becomes its own spectral observation, distinguished by
  `subsample` `a`, `b` (and `c`, `d`… in the rare case of more scans). 2009 has one scan, always `a`.
  Spectra are read only from the country files of the points actually loaded. A small `RECORDS`
  test run therefore always produces spectra for the points it picked.
- **Indicators.** 2015 has EC (`@ec`) but no CEC, and there is no PTotal file. Otherwise the lab
  columns map to the same indicators as 2009 (see [Observations explained (lab)][obs_lab]).
- **Texture backfill from 2009.** Texture (coarse, clay, sand, silt) was not always re-measured on
  revisited points. For every point with `Revisited_point == "Yes"` that lacks one or more of these
  in 2015, the script fills the gap from the same `POINT_ID` in `LUCAS.SOIL_corr.csv`. It streams
  only the texture columns of the 808 MB file, reads the file only if some loaded point needs it,
  and stops as soon as all needed points are found. On the full dataset, about 60,000 values are
  backfilled across about 15,500 points. Note that these values are 2009 measurements stored under
  the 2015 lab observation.
- **BIOGEO16** is joined from `LUCAS-Master-Grid.csv` exactly as for 2009. Its sampling log and
  observation log (`biogeo16`, `biogeo16@compilation`) are identical in both scripts. See
  [BIOGEO16 explained][obs_biogeo].

## 3. Insert utility and dataset metadata

Same notebooks as for 2009, run once for both campaigns. See [Insert utility][insert_utility] and
[Insert dataset metadata][insert_dataset_meta].

## 4. Load LUCAS 2015

**Path**: `xspatula_lucas/lucas/import_data/load_LUCAS_2015.ipynb`

Set `scheme_file = '../scheme_lucas.json'` (the default), then run the fourteen cells in this fixed
order. Each cell is a `job_file` under `import_data/LUCAS_2015/`. It is the same sequence as
[Load LUCAS 2009][load_lucas_2009], so the Detail column links to the 2009 reference.

| # | Cell | Job file | Detail |
|---|---|---|---|
| 1 | Manage LUCAS 2015 sampling log | `job_LUCAS_2015_sampling_log.json` | [Samples explained][samples] |
| 2 | Manage LUCAS 2015 geolocations | `job_LUCAS_2015_geolocation.json` | [Samples explained][samples] |
| 3 | Manage LUCAS 2015 samples | `job_LUCAS_2015_sample.json` | [Samples explained][samples] |
| 4 | Manage LUCAS 2015 laboratory observation log | `job_LUCAS_2015_observation_log_lab.json` | [Observations explained (lab)][obs_lab] |
| 5 | Manage LUCAS 2015 laboratory observations | `job_LUCAS_2015_observation_lab.json` | [Observations explained (lab)][obs_lab] |
| 6 | Manage LUCAS 2015 spectrometer | `job_LUCAS_2015_spectrometer.json` | [Observations explained (spectra)][obs_spectra] |
| 7 | Manage LUCAS 2015 spectra observation logs | `job_LUCAS_2015_observation_log_spectra.json` | [Observations explained (spectra)][obs_spectra] |
| 8 | Manage LUCAS 2015 spectra observations | `job_LUCAS_2015_observation_spectra.json` | [Observations explained (spectra)][obs_spectra] |
| 9 | Manage LUCAS 2015 landscape observation logs | `job_LUCAS_2015_observation_log_landscape.json` | [Observations explained (landscape)][obs_landscape] |
| 10 | Manage LUCAS 2015 land cover | `job_LUCAS_2015_land_cover.json` | [Observations explained (landscape)][obs_landscape] |
| 11 | Manage LUCAS 2015 land use | `job_LUCAS_2015_land_use.json` | [Observations explained (landscape)][obs_landscape] |
| 12 | Manage EEA biogeographic regions 2016 sampling log | `job_LUCAS_2015_sampling_log_biogeo.json` | [BIOGEO16 explained][obs_biogeo] |
| 13 | Manage EEA biogeographic regions 2016 observation log | `job_LUCAS_2015_observation_log_biogeo.json` | [BIOGEO16 explained][obs_biogeo] |
| 14 | Manage EEA biogeographic regions 2016 observations | `job_LUCAS_2015_observation_biogeo.json` | [BIOGEO16 explained][obs_biogeo] |

If LUCAS 2009 is already loaded, cells 12–13 find the shared biogeo16 logs in place. Cell 2 finds
the existing geolocations of revisited points.

**Small test runs and cell 14**: the first rows of the 2015 main csv are UK points, and BIOGEO16
does not cover the UK. With a small `RECORDS` value, the script may generate no BIOGEO16
observations at all. Cell 14 then fails with `❌ ERROR the path to the json process file(s) does
not exist`. Increase `RECORDS` (or set it to `0`) and re-run the script.

## After this notebook

The LUCAS 2015 campaign is loaded next to LUCAS 2009 and shares geolocations for revisited points.
The [Machine learning][explore_select_data] pipeline works the same way. To extract only the 2015
data, set `"campaign_name": "lucas_eu_2015"` in `select_spectra`. The spectrometer provision is the
same as for 2009. If you combine 2009 and 2015 extractions, set the same target unit for every
shared indicator (see [Indicator units][indicator_units]).

[synopsis]: /lucas_2009/
[prepare_data]: /lucas_2009/prepare_data/
[prepare_data_output]: /lucas_2009/prepare_data/#what-it-generates
[insert_utility]: /lucas_2009/insert_utility/
[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[dataset_meta_campaign]: /lucas_2009/dataset_meta_explained/#campaign
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
[samples]: /lucas_2009/samples_explained/
[obs_lab]: /lucas_2009/observations_explained/#laboratory-observations
[obs_spectra]: /lucas_2009/observations_explained/#spectral-observations
[obs_landscape]: /lucas_2009/observations_explained/#landscape-observations
[obs_biogeo]: /lucas_2009/observations_explained/#biogeographic-region-biogeo16-observations
[explore_select_data]: /lucas_2009/machine_learning/explore_select_data/
[indicator_units]: /lucas_2009/machine_learning/explore_select_data/#indicator-units
