---
title: "Prepare Data"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/prepare_data/
author_profile: false
---

Generates the JSON job, pilot, and process files needed to load the LUCAS 2009 campaign — one
process file per sampling point, per lab observation, and per spectral observation.

## 1. Download the source CSV

Register at [esdac.jrc.ec.europa.eu/projects/lucas](https://esdac.jrc.ec.europa.eu/projects/lucas)
and download `LUCAS.SOIL_corr.csv` — the corrected LUCAS 2009 topsoil dataset, one row per
sampling point, with lab-measured soil properties and FOSS XDS RCA spectral scan columns
(`spc.<wavelength>`) together in a single file.

## 2. Configure and run the script

**Path**: `xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`

Open it and check these constants before running:

| Constant | Purpose | Default |
|---|---|---|
| `CSV_PATH` | Absolute path to the downloaded CSV | a machine-specific path — **must be changed** |
| `OUTPUT_ROOT` | Where generated files land | `../import_data/LUCAS_2009` (resolved relative to the script's own directory) |
| `RECORDS` | How many CSV rows to process | `25` for a test run — **set to `0` for the full campaign** |
| `CAMPAIGN_NAME` | Campaign name written into every generated record | `lucas_eu_2009` |
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
| 1. Campaign & sampling log | `process_lab/campaign/`, `process_lab/sampling_log/` (+ duplicate under `process_spectra/`, see below) | campaign (static) |
| 2. Observation log | `process_lab/observation_log/`, `process_spectra/observation_log/` | provision (static, 2 records) |
| 3. Spectrometer | `process_spectra/spectrometer/` | instrument (static, 1 record) |
| 4. Geolocation | `process_lab/geolocation/` | unique `POINT_ID` |
| 5. Sample | `process_lab/sample/` | unique `POINT_ID` |
| 6. Lab observation | `process_lab/observation/` | CSV row with at least one measured indicator |
| 7. Spectral observation | `process_spectra/observation/` | CSV row |

Steps 4–7 are limited to `RECORDS` rows if you haven't set it to `0`. Each directory gets both a
`xspatula_add_<category>_pilot.txt` pilot file (a numbered list of the process files in it) and
the process files themselves under a `manage_process/` subfolder.

## Known quirk: duplicate campaign/sampling log

Step 1 writes a `lucas_eu_2009` campaign **and** sampling log record under **both**
`process_lab/` and `process_spectra/` — but only the `process_lab/` copy is ever run (see
[Load campaign][load_campaign]; the `process_spectra/` copy is generated and left unused). This
also overlaps with the campaign record created via `campaign.xlsx` in
[Insert dataset metadata][insert_dataset_meta] — read
[Dataset metadata → Manage campaign][manage_campaign] before running both against the same
database.

## Next step

Proceed to [Insert dataset metadata][insert_dataset_meta].

[load_campaign]: /lucas_2009/load_campaign/
[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[manage_campaign]: /dataset_meta/manage_campaign/
