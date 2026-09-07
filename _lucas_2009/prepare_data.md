---
title: "Prepare Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/prepare_data/
author_profile: false
---

Generate the JSON job, pilot, and process files needed to load the LUCAS 2009 campaign data and metadata.

## 1. Download the source CSV

Register at [esdac.jrc.ec.europa.eu/projects/lucas](https://esdac.jrc.ec.europa.eu/projects/lucas)
and download `LUCAS.SOIL_corr.csv` — the corrected LUCAS 2009 topsoil dataset, one row per
sampling point, with lab-measured soil properties and FOSS XDS RCA spectral scan columns
(`spc.<wavelength>`) together in a single file.

## 2. Configure and run the script

The content of `LUCAS.SOIL_corr.csv` can not be loaded directly to the database. It must first be translated to a format that is understood by a defined process in the Xspatula LUCAS framework. It is possible to define a process for reading  `LUCAS.SOIL_corr.csv`, but it will be both complicated and not useful for anything else. The script `lucas_2009_to_xspatula.py` instead translates `LUCAS.SOIL_corr.csv` to generic processes that are already defined as part of the `xspatula_lucas` project.

**Path**: `xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`

Open it and check these constants before running:

| Constant | Purpose | Default |
|---|---|---|
| `CSV_PATH` | Absolute path to the downloaded CSV | a machine-specific path — **must be changed** |
| `OUTPUT_ROOT` | Where generated files land | `../import_data/LUCAS_2009` (resolved relative to the script's own directory) |
| `RECORDS` | How many CSV rows to process | `25` for a test run — **set to `0` for the full campaign** |
| `CONTACT_NAME` | Contact name for LUCAS data | `inherit` takes the data from a foreign key parent table |
| `CONTACT_EMAIL` | Contact email for LUCAS data | `inherit` takes the data from a foreign key parent table |
| `CAMPAIGN_NAME` | Campaign name | `lucas_eu_2009` |
| `LAB_PROVISION` / `SPECTRA_PROVISION` | Provision names for the two observation logs | `lucas-wetlab-2009` / `foss xds rca` |
| `SPECTROMETER_PROVISION_ID` / `SPECTROMETER_SERIAL` | Spectrometer FK values | `foss-xds-rca` / `lucas 2009` |

Run with Python 3 (no extra CLI arguments — everything is controlled by the constants above):

```bash
cd xspatula_lucas/lucas/prepare_lucas_data
python3 lucas_2009_to_xspatula.py
```

It prints `OK: <step>` for each of 7 steps, or `FAILED: <step> - <error>` if one fails, then a
final `DONE` line. A `FileNotFoundError` at the start means `CSV_PATH` is wrong.

## What it generates

| Step | Output directory | One record per |
|---|---|---|
| 1. Sampling log | `process_lab/sampling_log/` | campaign (static, 1 record) |
| 2. Observation log | `process_lab/observation_log/`, `process_spectra/observation_log/` | provision (static, 2 records) |
| 3. Spectrometer | `process_spectra/spectrometer/` | instrument (static, 1 record) |
| 4. Geolocation | `process_lab/geolocation/` | unique `POINT_ID` |
| 5. Sample | `process_lab/sample/` | unique `POINT_ID` |
| 6. Lab observation | `process_lab/observation/` | CSV row with at least one measured indicator |
| 7. Spectral observation | `process_spectra/observation/` | CSV row |

Steps 4–7 are limited to `RECORDS` rows if you haven't set it to `0`. Each directory gets both a
`xspatula_add_<category>_pilot.txt` pilot file (a numbered list of the process files in it) and
the process files themselves under a `manage_process/` subfolder.

## Next step

Proceed to [Insert utility][insert_utility].

[insert_utility]: /lucas_2009/insert_utility/
[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
