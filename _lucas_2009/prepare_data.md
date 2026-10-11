---
title: "Prepare Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/prepare_data/
author_profile: false
---

Generate the JSON job, pilot, and process files needed to load the LUCAS 2009 campaign data and metadata.

## 1. Download the source files

Register at [esdac.jrc.ec.europa.eu/projects/lucas](https://esdac.jrc.ec.europa.eu/projects/lucas)
and download `LUCAS.SOIL_corr.csv` — the corrected LUCAS 2009 topsoil dataset, one row per
sampling point, with lab-measured soil properties and FOSS XDS RCA spectral scan columns
(`spc.<wavelength>`) together in a single file. Place it, and the optional complements below,
directly in one directory (`CSV_PATH`):

| File | Content | Needed for |
|---|---|---|
| `LUCAS.SOIL_corr.csv` | Main 2009 campaign, lab data + spectra | Everything |
| `PTotal2009.dbf` | Total phosphorus, merged by `POINT_ID` into the lab observation | `@p-tot` |
| `SoilAttr_ICELAND.dbf`, `SoilAttr_LUCAS_2009_CYP_MLT.dbf`, `SoilAttr_LUCAS_2012_BG_RO.dbf` | Lab data for Iceland, Cyprus/Malta and Bulgaria/Romania (no spectra, no dates) | Complement points |
| `LUCAS-Master-Grid.csv` | The LUCAS master grid, with a `BIOGEO16` column per `POINT_ID` | BIOGEO16 observations |

`LUCAS-Master-Grid.csv` is not year specific and can live elsewhere — its directory is set
separately in `MASTER_GRID_ROOT`.

## 2. Configure and run the script

The content of `LUCAS.SOIL_corr.csv` can not be loaded directly to the database. It must first be translated to a format that is understood by a defined process in the Xspatula LUCAS framework. It is possible to define a process for reading  `LUCAS.SOIL_corr.csv`, but it will be both complicated and not useful for anything else. The script `lucas_2009_to_xspatula.py` instead translates `LUCAS.SOIL_corr.csv` to generic processes that are already defined as part of the `xspatula_lucas` project.

**Path**: `xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`

Open it and check these constants before running:

| Constant | Purpose | Default |
|---|---|---|
| `CSV_PATH` | Absolute path to the directory holding the downloaded files | a machine-specific path — **must be changed** |
| `OUTPUT_ROOT` | Where generated files land | `../import_data/LUCAS_2009` (resolved relative to the script's own directory) |
| `MASTER_GRID_ROOT` | Directory holding `LUCAS-Master-Grid.csv` | a machine-specific path — **must be changed** |
| `INCLUDE_PTOTAL` / `INCLUDE_ICELAND` / `INCLUDE_CYP_MLT` / `INCLUDE_BG_RO` | Read the complement files or not | `True` |
| `RECORDS` | How many rows to process from *each* enabled file | a small number for a test run — **set to `0` for the full campaign** |
| `CONTACT_NAME` | Contact name for LUCAS data | `inherit` takes the data from a foreign key parent table |
| `CONTACT_EMAIL` | Contact email for LUCAS data | `inherit` takes the data from a foreign key parent table |
| `CAMPAIGN_NAME` | Campaign the sampling log belongs to | `lucas 2009` |
| `SAMPLING_LOG_NAME` | Sampling log referenced by samples and observation logs | `lucas 2009 eu` |
| `LAB_PROVISION` / `SPECTRA_PROVISION` / `LANDSCAPE_PROVISION` | Provision names for the campaign's observation logs | `lucas wetlab 2009` / `foss xds rapid content analyzer` / `human interpretation` |
| `*_OBSERVATION_LOG_NAME` | Observation log names, `<SAMPLING_LOG_NAME>@<provision>` | e.g. `lucas 2009 eu@lucas wetlab 2009` |
| `BIOGEO_CAMPAIGN_NAME` / `BIOGEO_SAMPLING_LOG_NAME` / `BIOGEO_PROVISION` | Campaign, sampling log and provision for BIOGEO16 | `biogeo16 2016` / `biogeo16 2016 eu` / `compilation` |
| `SPECTROMETER_PROVISION_ID` / `SPECTROMETER_SERIAL` | Spectrometer FK values | `foss-xds-rca` / `lucas 2009` |

The naming constants must match what the framework composes. The script no longer writes a `name`
for the sampling log and observation logs: the framework auto names them from the campaign,
territory and provision. The constants are still needed because samples and observations refer
to these names. If you change `datetime_tag` in `campaign.xlsx` or a provision name, update the
constants to match. See [Auto naming and tags explained]({{ site.baseurl }}/lucas_2009/auto_naming_explained/).

**Compilation error** when trying to run the script? Make sure that the Python environment you are running includes the packages for `numpy` and `dbfread`. To create a virtual Python environment with these included under Anaconda, see the README file under `xspatula_lucas/setup/anaconda/prepare_lucas_py_3.12.yml`.

Run with Python 3 (no extra CLI arguments — everything is controlled by the constants above):

```bash
cd xspatula_lucas/lucas/prepare_lucas_data
python3 lucas_2009_to_xspatula.py
```

It prints `OK: <step>` for each of 12 steps, or `FAILED: <step> - <error>` if one fails, then a
final `DONE` line. A file-not-found error at the start means `CSV_PATH` or `MASTER_GRID_ROOT` is wrong.

## What it generates

| Step | Output directory | One record per |
|---|---|---|
| 1. Sampling log | `process_lab/sampling_log/` | campaign (static, 1 record) |
| 1b. BIOGEO16 sampling log | `process_biogeo/sampling_log/` | campaign `biogeo16 2016` (static, 1 record) |
| 2. Observation logs | `process_lab/`, `process_spectra/`, `process_landscape/`, `process_biogeo/` `observation_log/` | provision (static, 4 records) |
| 3. Spectrometer | `process_spectra/spectrometer/` | instrument (static, 1 record) |
| 4. Geolocation | `process_lab/geolocation/` | unique `POINT_ID` |
| 5. Sample | `process_lab/sample/` | unique `POINT_ID` |
| 6. Lab observation | `process_lab/observation/` | row with at least one measured indicator |
| 7. Spectral observation | `process_spectra/observation/` | `LUCAS.SOIL_corr.csv` row |
| 8. Land cover | `process_landscape/land_cover/` | `LUCAS.SOIL_corr.csv` row with `LC1` |
| 9. Land use | `process_landscape/land_use/` | `LUCAS.SOIL_corr.csv` row with `LU1` |
| 10. BIOGEO16 observation | `process_biogeo/observation/` | `LUCAS.SOIL_corr.csv` point with a BIOGEO16 match |
| 11. Job files | `job_LUCAS_2009_*.json` in the output root | notebook cell (14 files) |

Steps 4–10 are limited to `RECORDS` rows per input file if you haven't set it to `0`. Each directory gets both a
`xspatula_add_<category>_pilot.txt` pilot file (a numbered list of the process files in it) and
the process files themselves under a `manage_process/` subfolder.

## Next step

Proceed to [Insert utility][insert_utility].

[insert_utility]: {{ site.baseurl }}/lucas_2009/insert_utility/
[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
