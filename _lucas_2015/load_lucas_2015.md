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
that translates them differ. This page covers those differences and lists the cells of block 4
of the notebook `insert_lucas_data.ipynb`. For the detail behind each step, follow the links back to the LUCAS 2009 pages.

## Prerequisites

- A running Xspatula database and the `xspatula_lucas` repository, as for
  [LUCAS 2009][synopsis].
- [Insert utility][insert_utility] and [Insert dataset metadata][insert_dataset_meta] completed.
  `campaign.xlsx` already registers the campaigns `lucas european union` and
  `biogeo16 european union` (see
  [Dataset metadata explained][dataset_meta_campaign]), and the utility spreadsheets already hold
  the `lucas wetlab 2015` provision and the `@ec` indicator. If you ran notebook blocks 1 and 2
  before these rows existed, run them again.
- LUCAS 2009 does **not** have to be loaded first. But you need `LUCAS.SOIL_corr.csv` on disk
  for the texture backfill (below).

## Dump and restore

Once LUCAS 2015 (and 2009) are loaded, dump the database with
`xspatula_lucas/setup/dump_restore_db.ipynb`. Restoring the dump takes minutes instead of the
hours the import takes. Dump again after every change to the import. See
[Dump once, restore in minutes][synopsis_dump] and, for the full instructions,
[Dump and restore][dump_restore] on the core docs site.

**LUCAS 2015 falls under the same [LUCAS_SOIL licence agreement][lucas_license] as LUCAS 2009.
Sharing the data is not permitted**, so do not pass a LUCAS dump, the generated import files or
extracted datasets to anyone outside your own organisation. See
[LUCAS license][synopsis_license] under the LUCAS 2009 documentation.

## 1. Download the source files

LUCAS 2009 comes as one file with lab data, coordinates, dates and spectra together. LUCAS 2015
splits them up. The soil data are on ESDAC and need the same request form as 2009 (name,
organisation, purpose, acceptance of the [licence agreement][lucas_license]); the master grid is
on Eurostat.

### Download list

| # | Download | From | Gives the script file |
|---|---|---|---|
| 1 | LUCAS 2015 TOPSOIL data, package `LUCAS2015_topsoildata_20200323` (CSV, with XLS and shapefile versions) | [ESDAC: LUCAS 2015 TOPSOIL data](https://esdac.jrc.ec.europa.eu/content/lucas2015-topsoil-data) | `LUCAS_Topsoil_2015_20200323.csv` and `LUCAS_Topsoil_2015_20200323.dbf` |
| 2 | Multispectral reflectance data (text version), one file per country | same page | `spectra/spectra_ <CC> .csv` (28 files) |
| 3 | LUCAS Master Grid, [`lucas-master-grid.zip`](https://ec.europa.eu/eurostat/cache/lucas/lucas-master-grid.zip) | [Eurostat: LUCAS primary data](https://ec.europa.eu/eurostat/web/lucas/database/primary-data) | `LUCAS-Master-Grid.csv` — the same file as for 2009 |
| 4 | `LUCAS.SOIL_corr.csv` from the 2009 download | [Prepare data (2009)][prepare_data] | used in place, from `LUCAS_2009_ROOT` |

What the script reads from each:

| File | Content |
|---|---|
| `LUCAS_Topsoil_2015_20200323.csv` | Main file: one row per point with lab data, `LC1`/`LU1` and `Revisited_point`. **No coordinates, no dates** |
| `LUCAS_Topsoil_2015_20200323.dbf` | `Point_ID` → Long/Lat lookup: the attribute table of the shapefile in the same package |
| `spectra/spectra_ <CC> .csv` | Spectra, one file per country, file names as downloaded (with the blanks). Usually **two scans per point** |
| `LUCAS-Master-Grid.csv` | `POINT_ID` → `BIOGEO16` |
| `LUCAS.SOIL_corr.csv` (2009) | Only the texture columns, for the [texture backfill](#what-the-script-does-differently) of revisited points |

Place files 1–3 directly under one directory (`CSV_PATH`); unlike the 2009 script, the 2015
script reads the master grid from `CSV_PATH`:

```
CSV_PATH/
├── LUCAS_Topsoil_2015_20200323.csv
├── LUCAS_Topsoil_2015_20200323.dbf
├── LUCAS-Master-Grid.csv
└── spectra/
    ├── spectra_ AT .csv
    ├── spectra_ BE .csv
    └── ...
LUCAS_2009_ROOT/
└── LUCAS.SOIL_corr.csv
```

The ESDAC page also offers the readme `readme_LUCAS2015_20200323.docx` and the survey report
`JRC121325_lucas_2015_topsoil_survey_final_1.pdf`; worth reading, but not read by the script.

## 2. Configure and run the script

**Path**: `xspatula_lucas/lucas/prepare_lucas_data/lucas_2015_to_xspatula.py`

It is configured the same way as the [2009 script][prepare_data]. The constants that differ:

| Constant | Purpose | Default |
|---|---|---|
| `CSV_PATH` | Directory holding the 2015 files above | a machine-specific path — **must be changed** |
| `OUTPUT_ROOT` | Where generated files land | `../import_data/LUCAS_2015` |
| `RECORDS` | Max rows from the main csv **and** max spectra scans in total | a small number for a test run — **set to `0` for the full campaign** |
| `LUCAS_2009_ROOT` | Directory holding `LUCAS.SOIL_corr.csv`, used for the texture backfill | a machine-specific path — **must be changed** |
| `CAMPAIGN_NAME` / `CAMPAIGN_TAG` | Campaign, the same as for 2009 (auto named by the framework, see [Auto naming and tags explained]({{ site.baseurl }}/lucas_2009/auto_naming_explained/)) | `lucas european union` / `lucas_eu` |
| `SAMPLING_LOG_NAME` / `SAMPLING_LOG_TAG` | The 2015 sampling log of that campaign | `lucas european union 2015` / `lucas_eu_2015` |
| `LAB_PROVISION` / `SPECTRA_PROVISION` / `LANDSCAPE_PROVISION` | Provisions for the three campaign observation logs | `lucas wetlab 2015` / `foss xds rapid content analyzer` / `human interpretation` |
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
  columns map to the same indicators as 2009 (see [Observations explained (lab)][obs_lab]). The
  2015 lab values are clean (no "not measured" codes, no negatives), so the 2009
  [data cleaning][data_cleaning] rules are not needed. The unit labels in
  `provision_indicator.xlsx` for `lucas wetlab 2015` are `g*kg^-1` for `c-org`, `n-tot` and `caco3`
  and `mg*kg^-1` for `k-ext` (earlier versions said weight percent and cmolc/kg; the values were
  right, only the labels were wrong). This also removed the "c-org has mixed native units" warning
  when 2009 and 2015 were selected together.
- **Coordinates.** As in 2009, the geolocation records carry `latitude`/`longitude` next to
  `x_coordinate`/`y_coordinate`.
- **Texture backfill from 2009.** Texture (coarse, clay, sand, silt) was not always re-measured on
  revisited points. For every point with `Revisited_point == "Yes"` that lacks one or more of these
  in 2015, the script fills the gap from the same `POINT_ID` in `LUCAS.SOIL_corr.csv`. It streams
  only the texture columns of the 808 MB file, reads the file only if some loaded point needs it,
  and stops as soon as all needed points are found. On the full dataset, about 60,000 values are
  backfilled across about 15,500 points. Note that these values are 2009 measurements stored under
  the 2015 lab observation.
- **BIOGEO16** is joined from `LUCAS-Master-Grid.csv` exactly as for 2009. Its sampling log and
  observation log (`biogeo16 european union 2016`, `biogeo16 european union 2016@compilation`) are identical in both scripts. See
  [BIOGEO16 explained][obs_biogeo].

## 3. Insert utility and dataset metadata

Blocks 1 (utilities) and 2 (dataset metadata) of `insert_lucas_data.ipynb` cover both survey
rounds, so they run once. See [Insert utility][insert_utility] and
[Insert dataset metadata][insert_dataset_meta].

## 4. Load LUCAS 2015

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_data.ipynb`, block **4. LUCAS 2015 data**

Block 4 has the same sequence as [block 3, LUCAS 2009][load_lucas_2009], so the Detail column
links to the 2009 reference. Each cell is a `job_file` under `import_data/LUCAS_2015/`. Run them
in this fixed order:

| Cell | Job file | Detail |
|---|---|---|
| 4.1 Load LUCAS 2015 sampling log | `job_LUCAS_2015_sampling_log.json` | [Auto naming][auto_naming] |
| 4.2 Load LUCAS 2015 geolocations | `job_LUCAS_2015_geolocation.json` | [Samples explained][samples] |
| 4.3 Load LUCAS 2015 samples | `job_LUCAS_2015_sample.json` | [Samples explained][samples] |
| 4.4.1 Load LUCAS 2015 laboratory observation log | `job_LUCAS_2015_observation_log_lab.json` | [Observations explained (lab)][obs_lab] |
| 4.4.2 Load LUCAS 2015 laboratory observations | `job_LUCAS_2015_observation_lab.json` | [Observations explained (lab)][obs_lab] |
| 4.5.1 Load LUCAS 2015 spectrometer | `job_LUCAS_2015_spectrometer.json` | [Observations explained (spectra)][obs_spectra] |
| 4.5.2 Load LUCAS 2015 spectra observation log | `job_LUCAS_2015_observation_log_spectra.json` | [Observations explained (spectra)][obs_spectra] |
| 4.5.3 Load LUCAS 2015 spectra observations | `job_LUCAS_2015_observation_spectra.json` | [Observations explained (spectra)][obs_spectra] |
| 4.6.1 Load LUCAS 2015 landscape observation logs | `job_LUCAS_2015_observation_log_landscape.json` | [Observations explained (landscape)][obs_landscape] |
| 4.6.2 Load LUCAS 2015 land cover | `job_LUCAS_2015_land_cover.json` | [Observations explained (landscape)][obs_landscape] |
| 4.6.3 Load LUCAS 2015 land use | `job_LUCAS_2015_land_use.json` | [Observations explained (landscape)][obs_landscape] |
| 4.7.1 Load BIOGEO16 sampling log | `job_LUCAS_2015_sampling_log_biogeo.json` | [BIOGEO16 sampling log][obs_biogeo_sl] |
| 4.7.2 Load BIOGEO16 observation log | `job_LUCAS_2015_observation_log_biogeo.json` | [BIOGEO16 observation log][obs_biogeo_ol] |
| 4.7.3 Load LUCAS 2015 biogeographic regions | `job_LUCAS_2015_observation_biogeo.json` | [BIOGEO16 explained][obs_biogeo] |

To load only LUCAS 2015, switch off the cells of block 3 (LUCAS 2009) with
`%%script false --no-raise-error`; see [the synopsis][synopsis_partial].

If LUCAS 2009 is already loaded, cells 4.7.1–4.7.2 find the shared biogeo16 logs in place. Cell 4.2 finds
the existing geolocations of revisited points.

**Small test runs and cell 4.7.3**: the first rows of the 2015 main csv are UK points, and BIOGEO16
does not cover the UK. With a small `RECORDS` value, the script may generate no BIOGEO16
observations at all. Cell 4.7.3 then fails with `❌ ERROR the path to the json process file(s) does
not exist`. Increase `RECORDS` (or set it to `0`) and re-run the script.

## After block 4

LUCAS 2015 is loaded as a second sampling log, `lucas european union 2015` / `lucas_eu_2015`, of
the same campaign as LUCAS 2009, and shares geolocations for revisited points. Now is the time
to [dump the database][synopsis_dump], for your own use only. The
[Machine learning][explore_select_data] pipeline works the same way. To extract only the 2015
data, set `"sampling_log": "lucas_eu_2015"` in the select processes; the output files then land
under `.../lucas_eu_2015/`. To get both rounds, leave `sampling_log` empty and set
`"campaign": "lucas_eu"`. The spectrometer provision is the
same as for 2009. If you combine 2009 and 2015 extractions, set the same target unit for every
shared indicator (see [Indicator units][indicator_units]).

[synopsis]: {{ site.baseurl }}/lucas_2009/
[synopsis_dump]: {{ site.baseurl }}/lucas_2009/#dump-once-restore-in-minutes
[synopsis_license]: {{ site.baseurl }}/lucas_2009/#lucas-license-the-data-may-not-be-shared
[dump_restore]: https://xspatula.github.io/xspatula_core_docs/setup_db/dump_restore/
[lucas_license]: https://esdac.jrc.ec.europa.eu/projects/LUCAS/Documents/LUCAS_SOIL_LIC_AGR_final_for_web.pdf
[prepare_data]: {{ site.baseurl }}/lucas_2009/prepare_data/
[prepare_data_output]: {{ site.baseurl }}/lucas_2009/prepare_data/#what-it-generates
[insert_utility]: {{ site.baseurl }}/lucas_2009/insert_utility/
[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[dataset_meta_campaign]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/#campaign
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[samples]: {{ site.baseurl }}/lucas_2009/samples_explained/
[obs_lab]: {{ site.baseurl }}/lucas_2009/observations_explained/#laboratory-observations
[obs_spectra]: {{ site.baseurl }}/lucas_2009/observations_explained/#spectral-observations
[obs_landscape]: {{ site.baseurl }}/lucas_2009/observations_explained/#landscape-observations
[obs_biogeo]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeographic-region-biogeo16-observations
[explore_select_data]: {{ site.baseurl }}/lucas_2009/machine_learning/select_data/
[indicator_units]: {{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#indicator-units
[data_cleaning]: {{ site.baseurl }}/lucas_2009/prepare_data/#data-cleaning
[auto_naming]: {{ site.baseurl }}/lucas_2009/auto_naming_explained/#the-lucas-chain
[obs_biogeo_sl]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeo16-sampling-log
[obs_biogeo_ol]: {{ site.baseurl }}/lucas_2009/observations_explained/#biogeo16-observation-log
[synopsis_partial]: {{ site.baseurl }}/lucas_2009/#data-loading-pipeline
