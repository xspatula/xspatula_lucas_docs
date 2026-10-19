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

The soil data are published by ESDAC (the European Soil Data Centre at the JRC) and are free, but
you must fill in a request form (name, organisation, purpose, acceptance of the
[LUCAS_SOIL licence agreement][lucas_license]) before you get download access. The licence is
personal: you may not pass the downloaded files, or anything generated from them, to anyone
outside your own organisation (see [LUCAS license][synopsis_license]). The master grid with the BIOGEO16 regions is published by Eurostat without
registration.

### Download list

| # | Download | From | Gives the script file | Needed for |
|---|---|---|---|---|
| 1 | `LUCAS_TOPSOIL_v1_spectral` (RDATA or .csv) — lab data, coordinates, dates, land cover/use and absorbance spectra in one table | [ESDAC: LUCAS 2009 TOPSOIL data](https://esdac.jrc.ec.europa.eu/content/lucas-2009-topsoil-data) | `LUCAS.SOIL_corr.csv` | Everything: samples, lab and spectral observations, land cover/use |
| 2 | Total phosphorus (CSV or shapefile), added 2023-02-02 | same page | `PTotal2009.dbf` | `@p-tot` (optional, `INCLUDE_PTOTAL`) |
| 3 | Shapefile package (version 2019-10-03), layers `SoilAttr_ICELAND`, `SoilAttr_CYP_MLT` and `SoilAttr_BG_RO` | same page | `SoilAttr_ICELAND.dbf`, `SoilAttr_LUCAS_2009_CYP_MLT.dbf`, `SoilAttr_LUCAS_2012_BG_RO.dbf` | Complement points for Iceland, Cyprus/Malta and Bulgaria/Romania (optional, `INCLUDE_*`) |
| 4 | LUCAS Master Grid, [`lucas-master-grid.zip`](https://ec.europa.eu/eurostat/cache/lucas/lucas-master-grid.zip) | [Eurostat: LUCAS primary data](https://ec.europa.eu/eurostat/web/lucas/database/primary-data) | `LUCAS-Master-Grid.csv` | BIOGEO16 observations |

Notes on the files:

- **`LUCAS.SOIL_corr.csv`** is the `LUCAS.SOIL_corr` table of the spectral download, saved as
  CSV: one row per sample, with the lab columns, `GPS_LAT`/`GPS_LONG`, `LC1`/`LU1`, the survey
`date` and one `spc.<wavelength>`
  column per band (400–2499.5 nm at 0.5 nm). The file is about 800 MB.
- **The `.dbf` files** are the attribute tables of the shapefiles. Only the `.dbf` is read; the
  script needs neither geometry nor projection files. Rename the files to the names in the table
  (or change `PTOTAL_FILENAME`, `ICELAND_FILENAME`, `CYP_MLT_FILENAME`, `BG_RO_FILENAME` in the
  script). The complements carry no spectra and no dates.
- **The master grid** is not year specific (the same file serves LUCAS 2015). Save it as CSV
  with the name `LUCAS-Master-Grid.csv`; the script reads the columns `POINT_ID` and `BIOGEO16`.
  It can sit in another directory, set separately in `MASTER_GRID_ROOT`.

Place files 1–3 directly in one directory (`CSV_PATH`):

```
CSV_PATH/
├── LUCAS.SOIL_corr.csv
├── PTotal2009.dbf
├── SoilAttr_ICELAND.dbf
├── SoilAttr_LUCAS_2009_CYP_MLT.dbf
└── SoilAttr_LUCAS_2012_BG_RO.dbf
MASTER_GRID_ROOT/
└── LUCAS-Master-Grid.csv
```

The ESDAC page also offers `LUCAS_TOPSOIL_v1` (Excel, lab data and coordinates without
spectra) and the report `EUR26102EN.pdf` (survey methodology). Neither is read by the script.

## 2. Configure and run the script

The content of `LUCAS.SOIL_corr.csv` can not be loaded directly to the database. It must first be translated to a format that is understood by a defined process in the Xspatula LUCAS framework. It is possible to define a process for reading  `LUCAS.SOIL_corr.csv`, but it will be both complicated and not useful for anything else. The script `lucas_2009_to_xspatula.py` instead translates `LUCAS.SOIL_corr.csv` to generic processes that are already defined as part of the `xspatula_lucas` project.

**Path**: `xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`

Open it and check these constants before running:

| Constant | Purpose | Default |
|---|---|---|
| `CSV_PATH` | Absolute path to the directory holding the downloaded files | a machine-specific path — **must be changed** |
| `OUTPUT_ROOT` | Where generated files land | `../import_data/LUCAS_2009` (resolved relative to the script's own directory) |
| `MASTER_GRID_ROOT` | Directory holding `LUCAS-Master-Grid.csv` | a machine-specific path — **must be changed** |
| `INCLUDE_MAIN_2009` | Read `LUCAS.SOIL_corr.csv` (spectra, land cover/use, BIOGEO16) or not | `True` |
| `INCLUDE_PTOTAL` / `INCLUDE_ICELAND` / `INCLUDE_CYP_MLT` / `INCLUDE_BG_RO` | Read the complement files or not | `True` |
| `RECORDS` | How many rows to process from *each* enabled file | a small number for a test run — **set to `0` for the full campaign** |
| `CONTACT_NAME` | Contact name for LUCAS data | `inherit` takes the data from a foreign key parent table |
| `CONTACT_EMAIL` | Contact email for LUCAS data | `inherit` takes the data from a foreign key parent table |
| `CAMPAIGN_NAME` / `CAMPAIGN_TAG` | Campaign (the place) the sampling log belongs to | `lucas european union` / `lucas_eu` |
| `SAMPLING_LOG_NAME` / `SAMPLING_LOG_TAG` | Sampling log (the 2009 survey) referenced by samples and observation logs | `lucas european union 2009` / `lucas_eu_2009` |
| `LAB_PROVISION` / `SPECTRA_PROVISION` / `LANDSCAPE_PROVISION` | Provision names for the campaign's observation logs | `lucas wetlab 2009` / `foss xds rapid content analyzer` / `human interpretation` |
| `*_OBSERVATION_LOG_NAME` | Observation log names, `<SAMPLING_LOG_NAME>@<provision>` | e.g. `lucas european union 2009@lucas wetlab 2009` |
| `BIOGEO_CAMPAIGN_NAME` / `BIOGEO_CAMPAIGN_TAG` | Campaign for BIOGEO16 | `biogeo16 european union` / `biogeo16_eu` |
| `BIOGEO_SAMPLING_LOG_NAME` / `BIOGEO_SAMPLING_LOG_TAG` / `BIOGEO_PROVISION` | Sampling log and provision for BIOGEO16 | `biogeo16 european union 2016` / `biogeo16_eu_2016` / `compilation` |
| `SPECTROMETER_PROVISION_ID` / `SPECTROMETER_SERIAL` | Spectrometer FK values | `foss-xds-rca` / `lucas 2009` |

The naming constants must match what the framework composes. The script no longer writes a `name`
for the sampling log and observation logs: the framework auto names them from the campaign,
the `datetime_tag` (`2009`, written into the sampling log record by the script) and the
provision. The sampling log record refers to its campaign by `CAMPAIGN_TAG`, and the observation
logs refer to the sampling log by `SAMPLING_LOG_TAG`. The name constants are still needed because
samples and observations refer to these names. If you change the territory in `campaign.xlsx`,
the `datetime_tag` in `step1_sampling_log()` or a provision name, update the constants to match. See [Auto naming and tags explained]({{ site.baseurl }}/lucas_2009/auto_naming_explained/).

**Compilation error** when trying to run the script? Make sure that the Python environment you are running includes the packages for `numpy` and `dbfread`. To create a virtual Python environment with these included under Anaconda, see the README file under `xspatula_lucas/setup/anaconda/prepare_lucas_py_3.12.yml`.

Run with Python 3 (no extra CLI arguments — everything is controlled by the constants above):

```bash
cd xspatula_lucas/lucas/prepare_lucas_data
python3 lucas_2009_to_xspatula.py
```

Before the step list it prints a summary of the lab values it omitted or substituted (see
[Data cleaning](#data-cleaning) below). Then it prints `OK: <step>` for each of 12 steps, or
`FAILED: <step> - <error>` if one fails, and a final `DONE` line. A file-not-found error at the
start means `CSV_PATH` or `MASTER_GRID_ROOT` is wrong.

## Data cleaning

The source files contain "not measured" codes, zeros that are not real results, values below the
detection limit and a few entry errors. `lucas_2009_to_xspatula.py` cleans them before anything is
written, so they never enter the database. Every lab value passes through `lab_float(value,
column)`, which applies these rules in order:

1. **Missing** (empty or `NA`): no parameter is written.
2. **Below the detection limit** (text starting with `<`, e.g. `<6`): written as **half the
   stated limit** (`3`). This is a common substitution for left-censored lab data: low-content
   samples stay in the data, and the value stays below the limit.
3. **Unparseable** (any other text): omitted.
4. **Negative**: omitted. All 2009 lab values are contents, concentrations or pH, so no negative
   value is valid. This also removes the "not measured" codes `-999` (Bulgaria/Romania) and
   `-888` (Iceland texture).
5. **Zero**: omitted, except in the columns listed in `ZERO_IS_VALUE` (`coarse`, `CaCO3`), where
   0 is a real result (no coarse fragments, a non-calcareous soil). In other columns 0 means not
   measured (Iceland CEC) or below the reporting precision (Cyprus/Malta report N in whole g/kg).
   - **P = 0 is deliberately omitted, not set to half a limit.** The 4,279 P zeros in
     `LUCAS.SOIL_corr.csv` probably mean "below 5 mg/kg" (the smallest non-zero P is 5), but the
     file states no limit. As a consequence, P from the main file underrepresents low-P soils.
6. **Known entry errors** (`EXCLUDED_VALUES`, keyed by `(POINT_ID, column)`): omitted. Currently
   the N values of Iceland points 90035 and 90057 (C/N 1.2 and 2.5, against ~11–15 for the other
   points).

An omitted value is left out of that observation's parameters, like a missing value; the sample's
other values are still imported.

**Where to adjust**: two constants near the top of the script.

| Constant | Content | Change it to |
|---|---|---|
| `ZERO_IS_VALUE` | `{"coarse", "CaCO3"}` | keep 0 as a real result in another column |
| `EXCLUDED_VALUES` | `{("90035", "N"): "entry error ...", ...}` | exclude another single value: add one line `(POINT_ID, column): reason` |

**Printed summary** for all 2009 records (`RECORDS = 0`):

```
  Omitted lab values (not written to the observation JSON):
    CEC: 23 negative (-999)
    CEC: 529 zero
    CaCO3: 8 negative (-999)
    K: 2 negative (-999)
    K: 121 zero
    N: 1 excluded, POINT_ID 90035: entry error (16.3 weight percent, C/N 1.2)
    N: 1 excluded, POINT_ID 90057: entry error (1.63 weight percent, C/N 2.5)
    N: 7 negative (-999)
    N: 28 zero
    OC: 15 negative (-999)
    OC: 41 zero
    P: 13 negative (-999)
    P: 4279 zero
    clay: 16 negative (-888)
    clay: 13 negative (-999)
    clay: 13 zero
    pH.in.CaCl2: 2 negative (-999)
    pH.in.H2O: 2 negative (-999)
    sand: 16 negative (-888)
    sand: 13 negative (-999)
    sand: 8 zero
    silt: 1 negative (-44)
    silt: 1 negative (-5)
    silt: 16 negative (-888)
    silt: 13 negative (-999)
    silt: 2 zero
  Below detection limit, written as half the limit:
    CaCO3: 689 <0.5 -> 0.25
    OC: 234 <6 -> 3
    P: 684 <5 -> 2.5
```

### Nitrogen units

All N is stored in **g/kg**, the unit of `LUCAS.SOIL_corr.csv`. The unit of each source file was
checked against the median C/N ratio (OC in g/kg divided by N), which is ~11 for mineral topsoils:

| Source | Unit in file | C/N as read | Conversion |
|---|---|---|---|
| `LUCAS.SOIL_corr.csv` | g/kg | 11.2 | none |
| `SoilAttr_LUCAS_2012_BG_RO.dbf` | g/kg | 10.7 | none |
| `SoilAttr_LUCAS_2009_CYP_MLT.dbf` | g/kg, whole numbers | 11.1 | none |
| `SoilAttr_ICELAND.dbf` | weight percent | 147 if read as g/kg | ×10 (`n_factor=10` in `load_dbf_records`) |

### Coordinates

The geolocation records carry `latitude`/`longitude` next to `x_coordinate`/`y_coordinate`.
Databases loaded before this was added have latitude and longitude = 0; the ML selection then
reads y/x as latitude/longitude when the spatial reference is `geographic`.

### Rebuild order

If you load (or reload) the data after these fixes:

1. Run the utility translate and insert ([Insert utility][insert_utility]), so the corrected unit
   labels in `provision_indicator.xlsx` are in the database.
2. Run `lucas_2009_to_xspatula.py`.
3. Load 2009 ([Load LUCAS 2009][load_lucas_2009]), and 2015 if you use it.

The selection step in machine learning screens the values once more; see
[Invalid values]({{ site.baseurl }}/lucas_2009/machine_learning/parquet_files/#invalid-values).

## What it generates

| Step | Output directory | One record per |
|---|---|---|
| 1. Sampling log | `process_lab/sampling_log/` | survey round `lucas_eu_2009` (static, 1 record) |
| 1b. BIOGEO16 sampling log | `process_biogeo/sampling_log/` | campaign `biogeo16_eu` (static, 1 record) |
| 2. Observation logs | `process_lab/`, `process_spectra/`, `process_landscape/`, `process_biogeo/` `observation_log/` | provision (static, 4 records) |
| 3. Spectrometer | `process_spectra/spectrometer/` | instrument (static, 1 record) |
| 4. Geolocation | `process_lab/geolocation/` | unique `POINT_ID` |
| 5. Sample | `process_lab/sample/` | unique `POINT_ID` |
| 6. Lab observation | `process_lab/observation/` | row with at least one measured indicator |
| 7. Spectral observation | `process_spectra/observation/` | `LUCAS.SOIL_corr.csv` row |
| 8. Land cover | `process_landscape/land_cover/` | `LUCAS.SOIL_corr.csv` row with `LC1` |
| 9. Land use | `process_landscape/land_use/` | `LUCAS.SOIL_corr.csv` row with `LU1` |
| 10. BIOGEO16 observation | `process_biogeo/observation/` | `LUCAS.SOIL_corr.csv` point with a BIOGEO16 match |
| 11. Job files | `job_LUCAS_2009_*.json` in the output root | cell of block 3 in `insert_lucas_data.ipynb` (14 files) |

Steps 4–10 are limited to `RECORDS` rows per input file if you haven't set it to `0`. Each directory gets both a
`xspatula_add_<category>_pilot.txt` pilot file (a numbered list of the process files in it) and
the process files themselves under a `manage_process/` subfolder.

## Next step

Proceed to [Insert utility][insert_utility].

[insert_utility]: {{ site.baseurl }}/lucas_2009/insert_utility/
[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[lucas_license]: https://esdac.jrc.ec.europa.eu/projects/LUCAS/Documents/LUCAS_SOIL_LIC_AGR_final_for_web.pdf
[synopsis_license]: {{ site.baseurl }}/lucas_2009/#lucas-license-the-data-may-not-be-shared
