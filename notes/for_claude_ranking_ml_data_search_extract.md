# For Claude: ranked land classes, campaign/sampling_log swap and ML data selection (xspatula_lucas_docs)

Written 2026-10-14 from a session in `xspatula_lucas`. Work through the tasks below from within this
repo. Everything goes into **existing pages**; no new page is needed. The biggest job is rewriting the
machine learning pages, which still show the old `select_spectra` and `data-`/`params-` file names.

Two sibling notes with the same file name exist in `xspatula_ai4sh_docs/notes/` and
`xspatula_core_docs/notes/`. The general descriptions (auto naming rules, foreign key resolution, the
setup database and setup processes pages) belong there. Here, show the LUCAS examples and link to them.

## Background

All work was on branch `feature/campaign-place-sampling-time` in `xspatula_lucas` that is fast-forwarded into main. The commits, oldest first:

| Commit | Content |
|---|---|
| `d6608fd` | Campaign anchored in place, sampling_log in time |
| `aa48203` | Tag-based ML data selection; cleaning of LUCAS 2009 lab values |
| `f85ae87` | `data-`/`params-` prefix dropped from selected Parquet and JSON files |
| `d8716c3` | Nominal covariates plotted as class-count bars |
| `15dc587` | Land cover and land use ranked in one table each (as taxon); `covariate_rank`; `class_label` |

The whole chain has been rebuilt and run on the live DB (setup, utilities, dataset meta, LUCAS 2009
import, select, compile, plot). All examples below are from that run.

The LUCAS 2009 import cleaning (negatives, zeros, `<x` values, Iceland N) is covered by the separate note
`notes/for_claude_lucas-2009_import_fixes.md`. Apply that one as well, but don't duplicate it here.

**Sources of truth.** Verify every example against these files; the code wins over this note:

- `../xspatula_lucas/src/ai4sh/select.py`: module docstring, `SELECT_ROLE_D`, `_Ml_output_paths`,
  `_Covariate_rank`, `_Feature_df`, `_Compile_dataset`, `MISSING_VALUE_CODES`, `Valid_unit_range`
- `../xspatula_lucas/src/postgres/pg_ai4sh.py`: `_Retrieve_landscape_table`, `_Retrieve_landscape_for_scope`
- `../xspatula_lucas/src/ai4sh/plot.py`: `_Plot_indicators`, `_Relabel_classes`, `_Class_bar_plot`
- `../xspatula_lucas/src/ai4sh/parquet_units.py`: `Find_selected_parquet`, `DERIVED_STEM_RE`
- `../xspatula_lucas/setup/zzz/lucas/setup_processes/json/select/*.json` and `plot/plot_v10_sql.json`
  (parameter names, defaults and hints)
- `../xspatula_lucas/lucas/project_lucas_2009/process/` (the LUCAS process files quoted below)
- `../xspatula_lucas/lucas/import_data/utility/landscape/excel/landcover.xlsx`, `landuse.xlsx`
- `../xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py` (constants, lines ~109-128)

## 1. Campaign in place, sampling_log in time

The time part moved from the campaign to the sampling_log. A campaign is now a place (dataset + territory
+ optional site), and each sampling_log is one sampling of that place at a time.

| Level | name | tag |
|---|---|---|
| dataset | `land use and coverage area frame survey (lucas) topsoil data` (alias `lucas`) | `lucas` |
| campaign | `lucas european union` | `lucas_eu` |
| sampling_log | `lucas european union 2009` | `lucas_eu_2009` |
| observation_log | `lucas european union 2009@foss xds rapid content analyzer` | `lucas_eu_2009@foss-xds-rca` |

Other observation logs: `...@lucas wetlab 2009` / `lucas_eu_2009@lucas-lab-2009`,
`...@human interpretation` / `lucas_eu_2009@hum-sens`. Biogeo16: campaign `biogeo16 european union` /
`biogeo16_eu`, sampling_log `biogeo16 european union 2016` / `biogeo16_eu_2016`, observation log tag
`biogeo16_eu_2016@compilat`.

**Pages to fix:**

- `_lucas_2009/auto_naming_explained.md`: the chain table (lines ~50-60) still shows `lucas 2009` /
  `lucas_2009` / `lucas 2009 eu`. Replace with the table above. `datetime_tag` is no longer a
  `campaign.xlsx` column (line ~33): it is given per sampling_log by the prepare script.
- `_lucas_2009/dataset_meta_explained.md`: `campaign.xlsx` lost the `datetime_tag` column. LUCAS 2015
  reuses the same campaign `lucas european union` with a new sampling_log `lucas european union 2015`.
- `_lucas_2009/prepare_data.md`: the constants are now `CAMPAIGN_NAME`/`CAMPAIGN_TAG`,
  `SAMPLING_LOG_NAME`/`SAMPLING_LOG_TAG` and the `BIOGEO_*` equivalents.
- `_lucas_2009/observations_explained.md` (biogeo16 table, line ~379) and
  `_lucas_2009/utility_inherit_auto_explained.md`: replace every old name (`lucas 2009 eu`,
  `biogeo16 2016 eu`, etc.).
- `_lucas_2015/load_lucas_2015.md` line ~137: `"campaign_name": "lucas 2015"` in `select_spectra` is the
  old parameter. It should now be `"sampling_log": "lucas_eu_2015"` (verify the tag in
  `lucas_2015_to_xspatula.py`).

Link to the core page for the rules: `https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/`.

## 2. Land cover and land use ranked in one table (as taxon)

The six rank tables (`landcover_order/_family/_genus`, same for land use) are replaced by one table per
scheme: `landscape_utility.landcover` and `landscape_utility.landuse`. They have the columns
`parent_<scheme>_id`, `class_rank_id`, `name`, `alias`, `tag`, `display_name` and `abstract`. A shared
catalogue `landscape_utility.class_rank` holds `order` (1), `family` (2) and `genus` (3). An observation
references the class at the rank actually recorded.

This matters for LUCAS: about half of the 2009 observations are recorded at family or order rank, not
genus. In the 50-point sample, land cover is 27 family + 23 genus and land use is 37 genus + 10 family +
3 order.

**Excel:** the six rank files are replaced by `landcover.xlsx` and `landuse.xlsx`. Each row is one class,
ordered so that a parent comes before its children:

| class_rank_id__class_rank_name | parent_landcover_id__landcover_alias | name | alias | tag | display_name |
|---|---|---|---|---|---|
| order | | cropland | b | cropland | Cropland |
| family | b | cultivated cropland | b10 | cultivated | Cultivated cropland |
| genus | b10 | common wheat | b11 | wheat | Common wheat |

Verify the rows in the Excel file. Alias and tag are unique across all ranks of a scheme. A name only
needs to be unique under its parent: `water` and `wetland` are both an order and a family. A genus may
sit directly under an order (`w00` under `w`).

`biogeo16.xlsx` gained a `tag` column.

**Pages to fix:**

- `_lucas_2009/utility_explained.md` lines ~80-85: the land use/cover catalogue description and its
  dependency order. There is now one insert per scheme (`landuse.json`, `landcover.json` in
  `insert_landscape_utility.txt`), after `class_rank` is created by setup.
- `_lucas_2009/insert_utility.md`: the landscape cell inserts `landuse` and `landcover` instead of six
  files.
- `_lucas_2009/observations_explained.md` line ~354: the key `landcover_genus_id__landcover_genus_name`
  is now `landcover_id__landcover_name` (and `landuse_id__landuse_name`). The value is the LUCAS code
  (alias) at whatever rank it was recorded, e.g. `b13` or `e20`.
- `_lucas_2009/foreign_key_explained.md`: add the two new `utility.foreign_key` rows
  (`parent_landcover_id` and `parent_landuse_id` → `landscape_utility.landcover`/`landuse`, search
  `alias`, then `tag`). This is the second real use of the table after `unit_translate`: a self
  reference can't resolve by the table-name convention because no table named `parent_landcover` exists.
  Also add the two import rules, and link to the core foreign key page for the full description:
  - an empty `parent_*` key is inserted as NULL (an order has no parent)
  - a value that is not found now stops the record with
    `❌ ERROR Can not find foreign key value <u999> for parent_landuse_id__landuse_alias`

## 3. Machine learning: selecting data (rewrite `_machine_learning/explore_select_data.md`)

The single `select_spectra` with free-text file names is replaced by separate, tag-named selections and
a compile step. The page must be rewritten from "Find the available indicators" through "Plot
indicators". Keep the spectra plot sections, but update their `project_root_fp`.

**Notebooks.** `explore_select_data.ipynb` is still the main notebook. The session also used three
notebooks that are **untracked in `xspatula_lucas`**: `select_spectra_only.ipynb`,
`select_indicator_only.ipynb` and `select_landscape_covariate_only.ipynb`. Before you reference them,
check whether they have been committed; if not, describe the cells through `explore_select_data.ipynb`
or ask.

**The select processes** (one function, `role` set by the process, see `SELECT_ROLE_D`):

| Process | Role | Saves |
|---|---|---|
| `select_spectra_only` | `spectra` | spectra only |
| `select_indicator_only` | `indicator` | numeric (and nominal) indicators |
| `select_covariate_only` | `covariate` | covariates, e.g. land cover, land use, biogeo16 |
| `select_spectra` | `all` | spectra + indicators (+ covariates) in one file |
| `compile_dataset` | — | joins saved selections on `sample_id` |
| `select_indicator` | — | prints what is available, saves nothing |

The process decides the role, not the data: pH can be an indicator in one file and a covariate in another.

**Scope parameters** (all select processes): `dataset`, `campaign`, `sampling_log` (name, tag or alias),
`sample_array`, `min_profile`, `max_profile`. Show the real files:

- `process/select_lucas_spectra_2009_10nm.json` (`select_spectra_only`: `spectra_provision`
  `foss-xds-rca`, `spectra_preparation_name` `ds2`, 400-2500 nm at 10 nm, `output_spectra_signal`
  `absorb`)
- `process/select/select_save_lucas2009_indicator.json` (`select_indicator_only`, `indicator_array`)
- `process/select/select_save_lucas2009_covariate.json` (`select_covariate_only`, `covariate_array`,
  `covariate_rank`)
- `process/compile_lucas_2009_10nm.json`
- `process/select_lucas_2009_10nm.json` (the all-in-one `select_spectra`)
- `process/select/select_list_lucas2009_indicator.json` (`select_indicator`; still `dataset_name`/`campaign_name`)

Old → new parameter names for `select_spectra`: `dataset_name` → `dataset`, `campaign_name` →
`campaign`, new `sampling_log`, `provision_name` → `spectra_provision`, `preparation_name` →
`spectra_preparation_name`, `as_absorbance: true` → `output_spectra_signal: "absorb"`.

**Output names are built from tags** (`_Ml_output_paths`). Defaults:

```
{output_root_fp}/{role}/{provision tag}/{preparation alias}/{scope tag}/{begin}-{end}_{bandwidth}.parquet   spectra
{output_root_fp}/{role}/{scope tag}/{aliases}_{min}-{max}.parquet                                        indicator, covariate
{output_root_fp}/compiled/{output_name}/{output_name}.parquet                                             compile
```

The scope tag is the most specific one given: sampling_log > campaign > dataset. The LUCAS files are:

```
data/spectra/foss-xds-rca/ds2/lucas_eu_2009/400-2500_10.parquet
data/indicator/lucas_eu_2009/cec-clay-silt-sand-c-org-n-tot-ph-water_0-20.parquet
data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.parquet
data/compiled/lucas_2009_10nm/lucas_2009_10nm.parquet
```

Each `.parquet` has a companion `.json` with the same name. There is **no `data-`/`params-` prefix any
more**. An indicator list longer than 60 characters is cut and given an 8-character hash. A covariate
rolled up to a rank gets the rank in the name (`landcover.order`). `output_sub_fp` and `output_name` can be
set to override the defaults.

**Key columns** in every output: `sample_id` (the join key; the same LUCAS point exists in 2009 and 2015),
`sample_name`, `sampling_log_tag`, `latitude_dd`, `longitude_dd`, `profile_min`, `profile_max`.

**Compile** inner-joins the inputs on `sample_id`, so only samples present in all inputs are kept. A
duplicate feature column is an error. Units and the params are merged, and the params JSON lists each
input with its sample count. The compiled folder is the `project_root_fp` for all ML steps.

**Value screening** (new section, near "Indicator units"): `select` drops missing-value codes
(`-9999`, `-999`, `-888`) and values outside the physical range of their unit (e.g. percent 0-100,
`g*kg^-1` 0-1000, pH 0-14, no negative contents). It prints what it dropped. This is the safety net
behind the import cleaning.

**Covariates and ranks** (new section): `covariate_rank` is `order`, `family`, `genus` or `lowest` (the
rank recorded, the default). It can be one rank for all, or JSON per covariate:
`{"landcover": "family", "landuse": "order"}`. A recorded class is rolled up through its parents to the
requested rank. A class recorded at a coarser rank than requested is kept as recorded, with a warning
giving the count (e.g. `landuse: 3 of 50 samples recorded at a coarser rank than family`). The parquet
stores the class **tag**. The companion JSON gets `covariate_rank` and `class_labels` (tag → `name`,
`display_name`, `alias`, `rank`); compile keeps both. Quote a few lines of
`data/covariate/lucas_eu_2009/landcover.order-landuse.order-biogeo16_0-20.json`.

## 4. Plotting indicators and covariates (update "Plot indicators")

- `plot_indicators` reads both `indicator_array` and `covariate_array`. Show
  `process/plot/plot_indicator.json` (now pointing at `data/indicator/lucas_eu_2009`, with
  `single_canvas_plot`/`individual_canvas_plot`) and `process/plot/plot_covariate.json`.
- Nominal columns (classes) can't be histogrammed or transformed. They are drawn as horizontal bars of
  class counts: largest on top, at most `max_classes` (default 15), the rest summed as
  `other (N classes)`. Files are `classes_<name>.png` and `classes_all-covariates.png`.
- `class_label`: `display_name` (default), `tag`, `name` or `alias`. It only changes the plot labels, not
  the data. A rank other than `lowest` is shown in the title, e.g. "Land cover (order)".
- Bar style per covariate in `lucas/default/plot/targetfeaturesymbols.json`: `color`, `alpha`, `hatch`
  (`/`, `\`, `|`, `-`, `+`, `x`, `o`, `O`, `.`, `*`; double a character for denser hatching),
  `fill: false` (outline only) and `edgecolor`. Current defaults: landcover olivedrab `//`, landuse
  goldenrod `\\`, biogeo16 teal `xx`, all at alpha 0.6.
- Transformation and standardisation defaults for the three covariates are `false`
  (`lucas/default/transform/*.json`); they skip non-numeric columns anyway.
- Indicator symbols now use `"unit": "native"` (the unit comes from the parquet metadata).

## 5. The other ML pages: drop the prefix and use the compiled folder

All ML process files now use `project_root_fp: ./project_lucas_2009/data/compiled/lucas_2009_10nm` and
dataframes named `lucas_2009_10nm_<steps>` (e.g. `lucas_2009_10nm_ol`, `lucas_2009_10nm_mc_d1`).
Verify against `process/*.json`.

- `_machine_learning/ml_preprocess.md`: replace every
  `data-foss xds rapid content analyzer_400-2500_10...` (lines ~35, 53, 147, 195, 214, 291, 317) with
  `lucas_2009_10nm...`. Remove the three "Fix needed" remarks about `data-neospectra_...` names (lines
  ~156, 302, 327); the shipped files are now correct. Under "How output is saved and chained", say that
  derived files keep the stem plus step suffixes (`_ol`, `_mc`, `_snv`, ...), see `DERIVED_STEM_RE`.
- `_machine_learning/inspect_dataset.md` line ~32 and the same example in `explore_select_data.md`
  line ~159: `parquet_file` becomes `lucas_2009_10nm.parquet` with the compiled `project_root_fp`
  (check `process/pandas/inspect_pandas_df.json`).
- `_machine_learning/ml_model.md` line ~92: `source_parquet` example.
- If a selection folder holds several candidate files, the newest is used with a warning
  (`Find_selected_parquet`). Stale `data-*.parquet` files from before the change are ignored. Worth one
  sentence on the explore page.

## Checklist

- [ ] Chain names replaced everywhere (`grep -rn "lucas 2009 eu\|lucas_2009_eu\|lucas_2009\b\|biogeo16 2016 eu"`)
- [ ] No `landcover_genus`/`landuse_genus`/`land_cover_order` left
  (`grep -rn "genus_id__land\|land_cover_order\|land_use_order"`)
- [ ] No `data-`/`params-` file names left in `_machine_learning/` (`grep -rn "data-\|params-" _machine_learning`)
- [ ] `explore_select_data.md` rewritten: processes table, scope parameters, naming, key columns,
      compile, value screening, covariate ranks, plotting
- [ ] Links to the core auto naming and foreign key pages, and to the ai4sh setup pages, where the
      general rules are described
- [ ] Build locally (`notes/local_serve_cmd.txt`) and check that the internal links resolve
