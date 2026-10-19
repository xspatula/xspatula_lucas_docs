# For Claude: split "Explore & Select Data" into Select / Parquet files / Plot (xspatula_lucas_docs)

Written 2026-10-19 from a session in `xspatula_lucas`. Work through the tasks below from within this repo.

The current page `_machine_learning/explore_select_data.md` (606 lines) is replaced by **three pages**, and
the existing `_machine_learning/inspect_dataset.md` is folded into the second one. A sibling note with the same
file name exists in `xspatula_ai4sh_docs/notes/`. That one creates a new, shorter ML section for AI4SH. This
page is the extended version: it covers texture, land cover, land use and biogeo16 filtering in depth,
because LUCAS has that data and AI4SH does not.

## Background

All code changes are on `main` in `xspatula_lucas` (pushed), oldest first:

| Commit | Content |
|---|---|
| `a021e88` | Class filters on all select processes; `identify_indicator` with counts; `sample_source`; `class_*` and `campaign_tag` columns; filter part in output names |
| `ff30aa9` | `.previous_dataframe` is an append-only history; select/compile register their parquet; inspect/plot/preprocess read the latest |
| `836fad8` | `process/inspect/inspect_latest_indicator_df.json`; `select_indicator_only.ipynb` reorganised (identify → select → inspect → plot) |
| `51960e6` | `plot/plot_indicator.json` renamed `plot/plot_lucas2009_indicators.json` |
| `6a1fdd8`, `6ade926` | `src/` made identical to `xspatula_ai4sh` (`project_paths.py`); not user visible |
| *(pending commit)* | Class filter fix: a class name found under several parents (e.g. juxtaposition per setting system) matches all of them; two entries naming the same class is an error |

**Sources of truth.** Verify every example against these files; the code wins over this note:

- `../xspatula_lucas/src/ai4sh/select.py`: module docstring, `CLASS_FILTERS`, `CLASS_COLUMN_PREFIX`,
  `DEFAULT_TEXTURE_CLASSIFICATION`, `DEFAULT_MIN_SAMPLES`, `KEY_COLUMNS`, `_Resolve_filters`,
  `_Read_sample_source`, `_Class_columns_df`, `_Filter_name_part`, `_Ml_output_paths`, `_Identify_indicator`,
  `_Select_data`, `_Compile_dataset`
- `../xspatula_lucas/src/postgres/pg_ai4sh.py`: `_Scope_sql`, `_Filter_class_sql`, `_Retrieve_class_descendants`,
  `_Retrieve_indicator_counts_for_scope`
- `../xspatula_lucas/src/ai4sh/previous_dataframe.py` (module docstring describes the file format)
- `../xspatula_lucas/src/ai4sh/pandas_df_management.py`, `parquet_units.py` (`Find_selected_parquet`)
- `../xspatula_lucas/setup/zzz/lucas/setup_processes/json/select/*.json` (parameter names, defaults, hints)
- `../xspatula_lucas/lucas/project_lucas_2009/process/select/`, `process/inspect/`, `process/plot/`
- `../xspatula_lucas/lucas/project_lucas_2009/select_indicator_only.ipynb` (the cell sequence to document)

## Prerequisites in `xspatula_lucas` (do these first, or flag them to the user)

1. Two notebooks still reference job files removed in `51960e6`/`a021e88`. Fix the paths before documenting
   the notebooks:
   - `explore_select_data.ipynb` → `process/plot/plot_indicator.json` (now `plot/plot_lucas2009_indicators.json`)
     and `process/select/select_list_lucas2009_indicator.json` (now `select/select_list_lucas_indicator.json`)
   - `select_landscape_covariate_only.ipynb` → `process/select/select_list_lucas2009_indicator.json`
2. The markdown cells of `select_indicator_only.ipynb` still have `???` placeholders, the `job_file`
   labels name the old `select_list_lucas2009_indicator.json`, and the documentation links are
   `add_link_to_documentation`. Point them at the three new pages once they exist.
3. Run `setup/setup_processes.ipynb` so the new select parameters are registered. Until then the process
   files with filters fail parameter validation.

## The split, and why it differs slightly from the request

The request was: 1. selection (with classification filtering), 2. saving/inspecting the parquet, what it
carries and its naming, 3. plotting. Agreed, with four adjustments:

- **Fold `inspect_dataset.md` into page 2.** It is a 91-line page about exactly page 2's topic. Two pages
  on inspecting the same file is one too many. Delete it and move its parameter reference and example run
  into page 2.
- **Indicator units and invalid values go to page 2, not page 1.** They are applied at selection time,
  but what the reader needs is "what unit is this column in, and why are some values missing". That is
  what the parquet carries.
- **Compile goes to page 2.** It takes saved parquet files and produces a parquet file. It is not a
  database selection.
- **Covariate ranks stay on page 1** (`covariate_rank` is a selection parameter). The `class_labels`/
  `covariate_rank` JSON excerpt goes to page 2.

| # | File | Title | Permalink |
|---|---|---|---|
| 1 | `_machine_learning/select_data.md` | Select Data | `/lucas_2009/machine_learning/select_data/` |
| 2 | `_machine_learning/parquet_files.md` | Parquet Files | `/lucas_2009/machine_learning/parquet_files/` |
| 3 | `_machine_learning/plot_data.md` | Plot Data | `/lucas_2009/machine_learning/plot_data/` |

Delete `explore_select_data.md` and `inspect_dataset.md`. The site has no `jekyll-redirect-from`, so
update every internal link (list below) instead of relying on redirects.

Front matter: copy the current page's (`layout: single`, `sidebar: nav: "lucas_2009"`,
`author_profile: false`).

## Page 1: Select Data

Keep from the current page: intro (rewritten for three pages), LUCAS licence paragraph, Notebooks table
(updated), The select processes table, Scope parameters, Select spectra, Select indicators, Select
covariates, All in one: select_spectra (shortened), Covariates and ranks (without the JSON excerpt).

Remove: the "Find the available indicators" section with `select_indicator`/`dataset_name`. That process
is gone, replaced by `identify_indicator`. Also drop the old-parameter-names table: it dates from before
the tag rename and is noise now. Keep one line saying `dataset_name`/`campaign_name` are no longer
accepted.

### Workflow at the top

Show the cell sequence of `select_indicator_only.ipynb` as the canonical workflow, numbered:

1. **Identify** (`select/select_list_lucas_indicator.json`, process `identify_indicator`): how many
   samples, per class filter entry, spectra sensor and indicator. Nothing saved.
2. **Select** (`select/select_save_lucas2009_indicator.json`, process `select_indicator_only`): saves
   parquet + JSON. → page 2
3. **Inspect** (`inspect/inspect_latest_indicator_df.json`): shows what was just saved. → page 2
4. **Plot** (`plot/plot_lucas2009_indicators.json`). → page 3

Spectra and covariates follow the same pattern (`select_spectra_only.ipynb`,
`select_landscape_covariate_only.ipynb`).

### The select processes table

Replace the `select_indicator` row with
`identify_indicator | — | prints sample counts per indicator, spectra sensor and filter class; saves nothing`.

### New section: Identify indicators and sample counts

Process file `select/select_list_lucas_indicator.json` (quote the real file; it lists every filter key
empty, plus `min_samples: 30`). Parameters: the scope parameters, every class filter (next section),
`sample_source`, and `min_samples` (default 30; counts below it are flagged ⚠️).

Real output, LUCAS 2009 unfiltered, from the cell in `select_indicator_only.ipynb` (dataset `lucas`, all
sampling logs). Trim the list:

```
    Querying available data for dataset "land use and coverage area frame survey (lucas) topsoil data"

    Samples in scope: 399

    Spectral sensors (provisions):
        - foss xds rapid content analyzer: 150 samples

    Indicators (samples with a value):
        - carbonate (caco3) [g*kg^-1]: 399
        - cation exchange capacity (cec) [cmolc*kg^-1]: 149
        - clay (<0.002 mm) (clay) [weight percent]: 382
        ...
        - soil texture class usda (soil texture usda) [class]: 382
        - total nitrogen (n-tot) [g*kg^-1]: 392
```

Explain: counts are samples with a value. Missing-value codes (-9999, -999, -888) are not counted; the
range screening ([Invalid values], page 2) can lower the saved count further. Nominal indicators get
unit `class`. The depth filter (`min_profile`/`max_profile`) is applied here too.

Then the **too-narrow example**. This is the reason the counts exist, so give it its own subheading
("When the selection is too narrow"). Real output, LUCAS 2009, texture + land cover:

```
    Querying available data for dataset "land use and coverage area frame survey (lucas) topsoil data" / sampling_log "lucas european union 2009"
    Filter texture: ['loam', 'silt loam', 'clay']
    Filter texture_classification: soil texture class usda
    Filter landcover: ['cropland', 'grassland']

    Samples in scope: 13  ⚠️  < 30

    Samples per class_texture:
        - loam: 6  ⚠️  < 30
        - silt loam: 5  ⚠️  < 30
        - clay: 2  ⚠️  < 30

    Samples per class_landcover:
        - cropland: 8  ⚠️  < 30
        - grassland: 5  ⚠️  < 30

    Spectral sensors (provisions):
        - foss xds rapid content analyzer: 13 samples  ⚠️  < 30
```

Message: 13 samples is too few for a model, let alone one per class. Broaden: drop a filter, add
classes, go up a rank (`cropland` instead of `wheat`), or widen the scope (both sampling logs). Same
filters over the whole `lucas` dataset give 105 samples (silt loam 52, loam 51, clay 2; grassland 78,
cropland 27). Use this as the "after" example: clay stays far too thin, so drop it.

### New section: Class filters (the core of this page, extended)

Opening paragraph, the concept from `notes/for_claude_advanced_select.md` in `xspatula_lucas`: a general
model over all classes vs. separate models per class. Splitting at selection means each class can get its
own preprocessing (indicator transformation, spectral enhancement), which is lost if the split happens
only at the modelling stage. The parquet still carries the class (page 2), so a general model can be
coloured or split by class later.

Parameter table (from `CLASS_FILTERS` + the registration JSON):

| Parameter | Filters on | Column in parquet | Name part |
|---|---|---|---|
| `texture_class_array` (+ `texture_classification`) | soil texture class of the sample (observation) | `class_texture` | `tx-` |
| `landcover_array` | land cover at the sample position | `class_landcover` | `lc-` |
| `landuse_array` | land use at the sample position | `class_landuse` | `lu-` |
| `biogeo16_array` | biogeographic region at the sample position | `class_biogeo16` | `bg-` |
| `juxtaposition_array`, `composition_array`, `proximity_array` | sample setting | `class_juxtaposition`, ... | `jx-`, `cp-`, `px-` |

Common rules (all verified in code):

- Each filter is a list. A sample passes if its class is any entry in the list (OR within a filter).
  Several filters combine with AND.
- An entry is a class name, tag, alias or display name, case-insensitive.
- An unknown entry stops the process and lists the valid classes. Two entries naming the same class stop
  it too.
- Samples without a recorded class are excluded by that filter. **LUCAS coverage in the current DB:** land
  cover and land use for 50 of 199 samples in 2009 and 200 of 200 in 2015; biogeo16 for 50 (2009) and 51
  (2015); texture for 186 (2009) and 196 (2015). Re-check these numbers with `identify_indicator` before
  publishing. State that coverage depends on what has been imported.
- Filters restrict **every** query (keys, spectra, indicators, covariates): they are applied once in the
  scope SQL (`_Scope_sql`). So a filtered spectra selection and a filtered indicator selection hold the
  same samples, before the inner joins on data availability.

Then one subsection per classification:

**Soil texture** (`texture_class_array`, `texture_classification`).
- LUCAS records two texture classifications as nominal indicators: `soil texture class usda` (alias
  `soil texture usda`, the default) and `soil texture class iuss` (alias `soil texture iuss`).
  `texture_classification` takes the name or alias.
- USDA classes with LUCAS sample counts (all sampling logs; re-check): clay 124, clay loam 64, loam 121,
  loamy sand 23, sand 6, sandy clay 0, sandy clay loam 4, sandy loam 66, silt 3, silt loam 173,
  silty clay 14, silty clay loam 56. Show as a table. The thin classes (sand, silt, sandy clay loam) make
  the point that one model per USDA class is rarely possible: group them (e.g. coarse/medium/fine). Note
  that grouping in one run is a planned feature. Today, run one selection per group with the group's
  classes in the list.
- IUSS classes: clay, clay loam, light clay, loam, loamy sand, sand, sandy clay, sandy clay loam,
  sandy loam (silt loam, silty clay, silty clay loam exist in the lookup with 0 samples).
- The "major group" classifications (`... major group`: heavy clayey / light clayey / loamy / sandy soils)
  exist as indicators but have no LUCAS data. Say so, since they would be the natural coarse grouping
  once filled.
- Texture is an observation of the sample, unlike the three landscape classifications.

**Land cover** (`landcover_array`) and **land use** (`landuse_array`): hierarchical.
- Ranked order > family > genus (link [Utility explained] and [Covariates and ranks]). Land cover:
  10 orders, 26 families, 73 genera. Land use: 4 orders, 17 families, 26 genera (from
  `landscape_utility`; re-check).
- **No rank parameter for filtering.** An entry can be at any rank and includes everything below it:
  `cropland` (order) = `cultivated` + `agroforest` (families) + all their genera. `["wheat", "barley"]`
  selects just those two genera. The rank is whatever you name.
- Entries in one filter must not overlap: `["primary production", "agriculture"]` is an error (agriculture
  lies within primary production). Real message:
  `❌ ERROR: landuse_array entries overlap (one lies within the other): agriculture and primary production`
- `class_landcover` holds **the filter entry the sample matched**, not the recorded class. A wheat sample
  under the filter `["cropland", "grassland"]` gets `cropland`. That is the value to split or colour by.
  Contrast with `covariate_rank`, which rolls the covariate *column* to a rank. Explain when to use which:
  filter = which samples and which group; covariate = the class as a model input or for plotting.
- Example orders: land cover tags `artificial, cropland, woodland, shrubland, grassland, bare-land, water,
  wetland, ice-snow, unclassified`; families of grassland: `grass-l-trees, grass-l-no-trees, spontan`. Land
  use orders: primary production (`prim-prod`), industrial production (`industry`), infrastructure services
  (`infrastructure`), unused. Show a short table, not the full lists. Link `landcover.xlsx`/`landuse.xlsx`
  for the full lists.
- Verification example (sum of children = parent): `landuse_array: ["primary production"]` → 220 samples;
  its five families together → 220.
- Joined by position (sample geotag → `landscape.<x>_observation`). If a position has several
  observations, the one from the sample's own sampling log wins, then the most recent: the same rule as
  the covariate selection.

**Biogeographic region** (`biogeo16_array`): flat, no ranks.
- Tags: alpine, anatolian, arctic, atlantic, black-sea, boreal, continental, macaronesia, mediterranean,
  outside, pannonian, steppic, na. In the current DB only continental (53), alpine (43) and pannonian (5)
  occur. Warn that a biogeo16 filter on the current LUCAS subset keeps at most about a quarter of the
  samples.

**Sample setting** (`juxtaposition_array`, `composition_array`, `proximity_array`): one short paragraph.
Every LUCAS sample has the same value (no juxtaposition / composite / general), so these filters do
nothing for LUCAS. Link to the AI4SH docs, where they matter. Mention that a juxtaposition name existing
under several setting systems matches all of them.

### New section: Reuse a selection's samples (`sample_source`)

The answer to "how do I get the spectra for exactly my filtered indicator samples":

- `sample_source` = a saved parquet (relative to the project root, `.parquet` optional).
- Its `sample_id`s restrict the scope. Its `class_*` columns and its `sample_filter` are inherited, so
  the spectra file gets the same filter part in its name.
- Why not repeat the filter: the indicator selection may already have dropped samples (inner join on
  indicators, `data_range`). `sample_source` gives exactly the same samples; `sample_id` is the DB key and
  unique across datasets.
- Quote `select/select_lucas_spectra_2009_10nm_landcover.json`. Real run: 37 indicator samples → spectra
  `400-2500_10__lc-cropland-grassland.parquet` with the same 37 samples.

### Covariates and ranks

Keep the current text, minus the JSON excerpt (moves to page 2). Add one sentence linking to the
class filter section on the filter-vs-covariate distinction.

## Page 2: Parquet Files

Sections, in order:

1. **What a select process saves.** `.parquet` + companion `.json`, same name.
2. **Output names.** Current "Output names" section, plus:
   - Filter part: a default `output_name` gets `__<filter part>`: per filter its abbreviation and entries,
     filters joined by `_`, cut and given an 8-character hash if longer than 60 characters. Examples:
     `clay-c-org-ph-water_0-20__lc-cropland-grassland`,
     `400-2500_10__lc-cropland-grassland` (spectra via `sample_source`).
   - `output_suffix`: replaces the auto filter part, and is appended (`__suffix`) even to a given
     `output_name`. `none` leaves it out. Default `default`.
   - Unfiltered names are unchanged from before (the unfiltered LUCAS 2009 indicator file was verified
     byte-identical apart from the new column).
3. **Columns.** Key columns (update the list: `campaign_tag` is new, between `sample_name` and
   `sampling_log_tag`), then `class_*` columns (one per filter, value = matched entry), then the data
   columns. Explain `campaign_tag` + `sampling_log_tag`: for plotting by campaign/survey round later. ML
   preprocessing and modelling treat key and `class_*` columns as metadata, never as features
   (`_is_metadata_col`).
4. **The companion JSON.** Show the real JSON of the filtered selection (strip `class_labels`):

   ```json
   {
     "role": "indicator",
     "dataset_tag": "lucas",
     "sampling_log_tag": "lucas_eu_2009",
     "min_profile": 0,
     "max_profile": 20,
     "indicator_array": ["clay", "c-org", "ph-water"],
     "indicator_units": {"clay": "weight percent", "ph-water": "ph", "c-org": "g*kg^-1"},
     "data_range": {},
     "sample_filter": {"landcover": ["cropland", "grassland"]},
     "sample_source": "",
     "class_columns": ["class_landcover"],
     "output_name": "clay-c-org-ph-water_0-20__lc-cropland-grassland",
     "n_samples": 37
   }
   ```

   (shortened: the real file also has the `*_name` fields; say so.) Then the covariate JSON excerpt with
   `covariate_rank` and `class_labels`, moved from the current page. Explain `sample_filter` (with
   `texture_classification` when texture is filtered) as the record of how the file was made.
5. **Indicator units**: current section, moved here unchanged.
6. **Invalid values**: current section, moved here unchanged.
7. **The previous dataframe (`.previous_dataframe`).** New:
   - Every select process, `compile_dataset` and every ML preprocessing step append a record to the
     hidden file `.previous_dataframe` in the folder they saved to: process, time, note, file.
   - The last record is "the previous dataframe". A process with an empty `parquet_file`/`dataframe`
     reads it, so a notebook runs select → inspect → plot → preprocess without repeating file names.
   - The earlier records are the folder's history, oldest first. Show a real two-record example:
     ```json
     {
       "history": [
         {"process": "select_indicator_only", "created_at": "2026-10-19T08:49:03",
          "note": "indicator: 199 samples",
          "entries": [{"dataframe": "clay-c-org_0-20.parquet", "indicator": null, "regressor": null, "selector": null}]},
         {"process": "select_indicator_only", "created_at": "2026-10-19T08:49:04",
          "note": "indicator: 23 samples, filter {'landcover': ['cropland']}",
          "entries": [{"dataframe": "clay-c-org_0-20__lc-cropland.parquet", "indicator": null, "regressor": null, "selector": null}]}
       ]
     }
     ```
   - Spectra, indicator and covariate folders each keep their own history.
   - Folders created before this change have none until a select runs there again (inspect then says
     `ERROR: no .previous_dataframe found in ...`).
   - Plot picks the latest *selected* (not derived) file from the history, falling back to the newest by
     date (`Find_selected_parquet`). Update the current "several candidate selections" paragraph
     accordingly.
8. **Inspect the dataset.** Merge `inspect_dataset.md` here. Lead with
   `inspect/inspect_latest_indicator_df.json` (`"parquet_file": ""`; also `none`/`previous`), and its
   real output line `Previous: select_indicator_only — clay-c-org_0-20__lc-cropland.parquet (...)`.
   Then the explicit-file variant (`pandas/inspect_pandas_df.json`) and the parameter reference from
   `inspect_dataset.md`.
9. **Compile the project dataset**: current section, plus: `class_*` columns repeated across inputs are
   kept once; the compiled JSON merges `class_columns`; compile registers its output in
   `.previous_dataframe`.

## Page 3: Plot Data

Move "Plot indicators and covariates", "Spectra plots" and the four spectra plot sections unchanged,
except:

- Process file name: `plot/plot_lucas2009_indicators.json` (was `plot/plot_indicator.json`).
- `project_root_fp` points at a folder. Which file in it is plotted: the latest selected parquet in its
  `.previous_dataframe` (link page 2).
- Without `indicator_array`, all non-key, non-`class_*`, non-band columns are plotted. To plot the class
  distribution of a filter, put the `class_*` column in `covariate_array`. Verified: `indicator_array:
  ["clay"]`, `covariate_array: ["class_landcover"]` on the cropland/grassland selection prints
  `Plotting 1 numeric: ['clay'] and 1 nominal: ['class_landcover']` and saves
  `classes_class_landcover.png` + `boxplot_clay.png`. No `targetfeaturesymbols` entry exists for
  `class_*` names, so default bar styling applies.
- Add a short "Coming next" paragraph: colour by class and marker shape by campaign/sampling log are
  planned. The columns needed (`class_*`, `campaign_tag`, `sampling_log_tag`) are already in every parquet.
  No parameters yet; don't invent any.

## Site wiring

- `_config.yml` `nav_order: machine_learning:` → `select_data`, `parquet_files`, `plot_data`,
  `ml_preprocess`, `ml_model` (drop `explore_select_data`, `inspect_dataset`).
- `_data/navigation.yml`, three places: line ~25 (main "Machine learning" → `/lucas_2009/machine_learning/select_data/`),
  the `lucas_2009` "Machine learning" children (~62-71: Select data, Parquet files, Plot data, ML
  preprocessing, ML modeling), and ~91.
- `_data/story_order.yml`: replace the explore_select_data and inspect_dataset lines with the three new
  URLs in order, before `ml_preprocess`.
- Internal links to update (grep `explore_select_data` and `inspect_dataset`). Anchor mapping:
  - `_lucas_2009/load_lucas_2009.md:97` → page 1
  - `_lucas_2009/foreign_key_explained.md:79,127` (Indicator units) → page 2 `#indicator-units`
  - `_lucas_2009/prepare_data.md:207` (`#invalid-values`) → page 2 `#invalid-values`
  - `_lucas_2009/observations_explained.md:570` (`#covariates-and-ranks`) → page 1 `#covariates-and-ranks`
  - `_lucas_2009/synopsis.md:93,134,153` → page 1. Also update the pipeline sketch at line 93: notebook
    `select_indicator_only.ipynb` etc.
  - `_lucas_2015/load_lucas_2015.md:196,219,220` → page 1, and page 2 `#indicator-units`
  - `_machine_learning/ml_preprocess.md:10,38,386` → page 2 (compile); also mention there that an empty
    `dataframe` reads `.previous_dataframe`, whose last record is now the selection.
- Build locally (`notes/local_serve_cmd.txt`) and click through prev/next and every changed link.

## Don't

- Don't document `select_indicator`/`dataset_name` as current. It no longer exists in the code.
- Don't invent plot parameters for class colouring.
- Don't copy the "rank" question from `for_claude_advanced_select.md` as open. It is decided (no rank
  parameter for filters).
- Keep LUCAS sample counts marked "current DB". They change with each import.

## Session summary (2026-10-19, Claude)

Done:
- New `_machine_learning/select_data.md`, `parquet_files.md`, `plot_data.md`; deleted
  `explore_select_data.md` and `inspect_dataset.md` (folded into Parquet files).
- `_config.yml` nav_order, `_data/navigation.yml` (3 places), `_data/story_order.yml` updated.
- Links updated: load_lucas_2009, foreign_key_explained (→ #indicator-units), prepare_data
  (→ #invalid-values), observations_explained (→ #covariates-and-ranks), synopsis (pipeline sketch +
  links), load_lucas_2015, ml_preprocess (compile link; `.previous_dataframe` section now describes the
  history and `"dataframe": "previous"` vs default `"raw"`).
- Correction vs this note: ML preprocessing reads `.previous_dataframe` with `"dataframe": "previous"`,
  not with an empty value (empty/none/previous only applies to inspect's `parquet_file`). Pages follow
  the code.
- xspatula_lucas prerequisite 1 done (uncommitted): job file paths fixed in `explore_select_data.ipynb`
  and `select_landscape_covariate_only.ipynb`.
- Local build OK; no links to the old pages remain, all internal anchors resolve, prev/next verified.

Open:
- Prerequisite 2 (`select_indicator_only.ipynb` markdown: `???`, old job_file labels,
  `add_link_to_documentation`) not done: the notebook has uncommitted user edits.
- Prerequisite 3: run `setup/setup_processes.ipynb`.
- Counts are from this note (current DB), not re-run.
