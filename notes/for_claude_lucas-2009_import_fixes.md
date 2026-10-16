# For Claude: document the LUCAS 2009 import fixes (xspatula_lucas_docs)

Written 2026-10-13 from a session in `xspatula_lucas` (branch `feature/campaign-place-sampling-time`).
Work through the tasks below from within this repo. The fixes clean invalid and below-detection
lab values before they enter the database, correct the nitrogen units and several unit labels, and
fill sample coordinates.

## Background

Plotting the 2009 indicators crashed: n-tot had zeros, and the default log transform turned them
into -inf. Tracing it back showed more problems in the source data and the import:

| Problem | Cause |
|---|---|
| n-tot = 0 | Cyprus/Malta report N in whole g/kg, so low values round to 0; the main CSV has zeros too |
| n-tot scale wrong (median 0.0026) | the script divided Iceland N by 10000 and Bulgaria/Romania N by 1000 |
| clay/silt/sand = -888 or -999 | "not measured" codes in the Iceland and Bulgaria/Romania dbf files |
| cec = 0 for all Iceland points | not measured, but stored as 0 |
| wrong unit labels | `provision_indicator.xlsx` (see below) |
| latitude/longitude = 0 in every selection | the script wrote only `x_coordinate`/`y_coordinate` |

**Sources of truth.** Verify every statement against these files; the code wins over this note:

- `../xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`:
  - module docstring
  - `ZERO_IS_VALUE`, `EXCLUDED_VALUES`, `lab_float`, `below_detection_float`, `report_omitted`
  - `load_dbf_records(..., n_factor=...)`
- `../xspatula_lucas/lucas/import_data/utility/observation/excel/provision_indicator.xlsx`
- `../xspatula_lucas/src/ai4sh/select.py` (`MISSING_VALUE_CODES`, `Valid_unit_range`, `_Screen_values`)
- `../xspatula_lucas/src/ai4sh/chemometrics.py` (`_Mask_outside_domain`)
- `../xspatula_lucas/src/postgres/pg_ai4sh.py` (`GEOLOCATION_LAT_LON_SQL`)

## Rules in `lucas_2009_to_xspatula.py`

Every lab value passes through `lab_float(value, column)`. In this order:

1. **Missing** (empty or `NA`): no parameter is written. Unchanged from before.
2. **Below the detection limit** (a text value starting with `<`, e.g. `<6`): written as **half the
   stated limit**. This is a common substitution for left-censored lab data. It keeps low-content
   samples in the data instead of dropping them, and stays below the limit.
3. **Unparseable** (any other text): omitted.
4. **Negative**: omitted. All 2009 lab values are contents, concentrations or pH, so no negative
   value is valid. This also removes the "not measured" codes -999 (Bulgaria/Romania) and -888
   (Iceland texture).
5. **Zero**: omitted, except in the columns listed in `ZERO_IS_VALUE` (`coarse`, `CaCO3`). There
   0 is a real result: no coarse fragments, or a non-calcareous soil. In other columns, 0 means not
   measured (Iceland CEC) or below the reporting precision (e.g. Cyprus/Malta N in whole g/kg).
   - **P = 0 is deliberately omitted, not set to half a limit.** The 4,279 P zeros in
     `LUCAS.SOIL_corr.csv` probably mean "below 5 mg/kg" (the smallest non-zero P is 5), but the
     file states no limit, so the author chose to skip them. Mention this explicitly: P from the
     main file underrepresents low-P soils.
6. **Known entry errors** (`EXCLUDED_VALUES`, keyed by `(POINT_ID, column)`): omitted. Currently
   Iceland points 90035 and 90057, N (C/N 1.2 and 2.5 against ~11–15 for the other points). To
   exclude another single value, add one line to `EXCLUDED_VALUES`.

An omitted value is just left out of that observation's parameters, like a missing value. The rest
of the sample's values are still imported.

When the script runs, it prints a summary before the step list. Result for all 2009 records
(`RECORDS = 0`):

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

## Nitrogen units

All N is now stored in **g/kg**, the unit of `LUCAS.SOIL_corr.csv`. The old docstring said
"weight percent" and applied wrong divisors. The units were checked against the median C/N ratio
(OC in g/kg divided by N), which is ~11 for mineral topsoils:

| Source | Unit in file | C/N as read | Conversion |
|---|---|---|---|
| `LUCAS.SOIL_corr.csv` | g/kg | 11.2 | none |
| `SoilAttr_LUCAS_2012_BG_RO.dbf` | g/kg | 10.7 | none (was ÷1000) |
| `SoilAttr_LUCAS_2009_CYP_MLT.dbf` | g/kg, whole numbers | 11.1 | none |
| `SoilAttr_ICELAND.dbf` | weight percent | 147 if read as g/kg | ×10 (was ÷10000) |

## Unit labels in `provision_indicator.xlsx`

These labels were wrong; the values themselves were fine. Corrected:

| Provision | Indicator | Was | Now |
|---|---|---|---|
| lucas wetlab 2009 / 2015 | n-tot | weight percent | g\*kg^-1 |
| lucas wetlab 2009 / 2015 | caco3 | weight percent | g\*kg^-1 |
| lucas wetlab 2009 / 2015 | k-ext | cmolc\*kg^-1 | mg\*kg^-1 |
| lucas wetlab 2015 | c-org | weight percent | g\*kg^-1 |

This also explains the earlier "c-org has mixed native units" warning when selecting 2009 + 2015:
the values were consistent, only the 2015 label was wrong.

## Coordinates

- Both prepare scripts (2009 and 2015) now write `latitude`/`longitude` next to
  `x_coordinate`/`y_coordinate` in the geolocation JSON.
- For databases loaded before the fix, the ML selection reads y/x as latitude/longitude when
  latitude and longitude are 0 and the spatial reference is `geographic`.

## Safety nets in the ML part (all datasets, not only LUCAS)

- **Selection** (`select.py`, `_Screen_values`): every numeric indicator or covariate is checked
  after unit translation:
  - values that are missing codes (-9999, -999, -888) are set to missing
  - so are values outside the valid range of their unit: percent 0–100, permill 0–1000, g/kg
    0–1000, mg/kg 0–1e6, pH 0–14, and no negative contents, concentrations, conductivities, lengths
    or areas
  - each removal is reported per indicator
- **Transformation** (`chemometrics.py`): log of values ≤ 0, sqrt of values < 0 and reciprocal of 0
  give a missing value with a warning, instead of ±inf that crashes plots.

## Tasks in this repo

1. `_lucas_2009/prepare_data.md`:
   - add a section **"Data cleaning"** after "2. Configure and run the script", covering:
     - the six rules above (including the P = 0 decision)
     - `ZERO_IS_VALUE` and `EXCLUDED_VALUES` as the two places to adjust
     - the printed summary
     - the N unit table
   - mention in "Configure and run" that the summary is printed before the step list
2. `_lucas_2009/observations_explained.md`:
   - the paragraph after the lab observation JSON example currently says only missing/unparseable
     values are omitted; extend it to "missing, invalid or excluded values are omitted;
     below-detection values are written as half the limit" and link to the new section
   - add the units to the CSV column table (g/kg for OC, N, CaCO3; mg/kg for P, K; weight percent
     for coarse, clay, silt, sand; cmolc/kg for CEC)
3. Wherever the docs show the 2009 N unit as weight percent, or the old divisors, correct them.
   Search for `weight percent`, `divide`, `10000`.
4. If an ML/selection page exists (or is written later), add the two safety nets as a short
   "Invalid values" subsection.
5. Rebuild order to state wherever the full LUCAS load is described:
   1. run the utility translate and insert, so the corrected unit labels are in the database
   2. run `lucas_2009_to_xspatula.py`
   3. load 2009 (and 2015)

## Not in scope

- The 2015 script: its lab values are clean (no codes, no negatives). Only the coordinate change
  applies.
- `xspatula_ai4sh` / `xspatula_core` docs: nothing to port until the code is ported.
