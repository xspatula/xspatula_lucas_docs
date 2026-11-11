# For Claude: the synthetic project tutorial (collection `_synthetic`)

Written 2026-11-10 from Thomas's sketch `_synthetic/for_claude_20261108.md`, checked against the code repo
(`../xspatula_lucas`, folder `synthetic_dataset/`). Updated the same day with Thomas's decisions: notebook 4
gets a cleanup cell (4.3) before predicting with the registered model (4.4), and notebooks 5–8 get **one**
"Covariates" page. The whole collection goes online now, under **one** main navigation entry that Thomas
can switch off (section 2).

## 1. Aim and hard rules

A complete step-by-step tutorial: set up a database, seed it with the synthetic dataset, and run the
machine-learning chain from compiling the data to registering a model. The reader is a newcomer who
has the software installed.

- **No software setup.** Link to the existing installation page instead (Prerequisites).
- **Disclaimer, prominent, at the top of the first page** (use a `notice--warning` block). Keep the
  meaning of Thomas's text:
  > The data are synthetic, derived from the LUCAS 2015 topsoil survey. Neither the laboratory values
  > nor the spectra are real measurements of any soil sample. The data cannot be used to model real
  > soil conditions; they are only for learning how to model soil physical and chemical properties
  > from NIR spectra with Xspatula.
- Figures are in `/assets/media/synthetic/` (file names in section 6). Do not invent figures. Where a
  figure is missing, leave an HTML comment `<!-- figure: ... -->` and list it in your report.
- Follow the style of `_machine_learning/ml_predict.md`: front matter, short paragraphs, parameter
  tables, JSON in fenced blocks, link to the `_machine_learning` pages for details instead of
  repeating them.
- **Never show a real password.** Use the placeholders below.

## 2. Collection setup: one switchable entry

Thomas is asking the LUCAS data owner for permission to distribute the synthetic data. Until then the
data can only be run locally, but the docs go online. **Everything about the synthetic project must
hang on one main navigation entry**, so it can be removed in one place:

- `_config.yml`: add `synthetic: output: true` under `collections`, and a `nav_order: synthetic:` list.
- `_data/navigation.yml`: **one** main entry "Synthetic tutorial" (url `/synthetic/`) and a `synthetic:`
  sidebar listing all pages, as was done for `lucas_2015`. Also one line in the landing-page list (the
  block with `description:`). No links to `/synthetic/` from other collections.
- Add a comment above each of these three spots: `# synthetic tutorial: remove this block (and set
  output: false) to take it offline`. Write the same instructions in `README.md`.
- On the synopsis page, add a `notice--info`: the synthetic dataset is not yet distributed; the pages
  show how the project is run.
- Permalinks `/synthetic/<page>/`, `sidebar: nav: "synthetic"`.
- Update `README.md`.
- Delete `_synthetic/for_claude_20261108.md` from the collection folder when done (it would be rendered
  as a page). Move it to `notes/` if Thomas wants it kept.

Suggested pages (one notebook per page, except setup):

| Page | Content | Sections below |
|---|---|---|
| `synopsis` | aim, disclaimer, what the data are, prerequisites, overview of the run order | 3 |
| `setup_database` | steps 1–7 | 4.1–4.7 |
| `import_data` | step 8 | 4.8 |
| `compile` | notebook 1 | 5.1 |
| `preprocess` | notebook 2 | 5.2 |
| `model` | notebook 3 | 5.3 |
| `predict_register` | notebook 4, cells 4.1–4.5 | 5.4 |
| `covariates` | notebooks 5–8 on one page | 5.5 |

## 3. Synopsis content (facts to use)

- 2000 calibration + 300 independent validation samples. Each sample is a random-weight mix of 3
  LUCAS 2015 points that share land cover family, land use family and biogeographic region and lie
  within 50 km of each other. The same weights apply to spectra, lab values and coordinates. No LUCAS
  point is used in both sets.
- Lab indicators: organic carbon `c-org`, total nitrogen `n-tot`, electrical conductivity `ec`,
  pH in water `ph-water`, `clay`, `silt`, `sand`. Texture is missing for about 23% of samples.
- Spectra: 400–2500 nm in 10 nm bands (211 bands), from the FOSS XDS scans of LUCAS 2015.
- Nominal covariates: biogeographic region (BIOGEO16), land cover, land use, USDA soil texture class.
- Sample names: `cal0001@0-20` …, `val0001@0-20` …; database name `synthetic`.

## 4. Database setup and data import (correct the sketch as noted)

### 4.1 Setup scheme `setup/zzz/scheme_local_setup_synthetic.json`

Use the JSON from the sketch. Add a table explaining the keys:

| Key | Meaning |
|---|---|
| `project_path` | `./lucas`, relative to the scheme file: the folder `setup/zzz/lucas` holding the database definitions. Keep it. |
| `postgresdb.db` | Name of the new database (`synthetic`) |
| `postgresdb.host_netrc_id` | `.netrc` machine name of your PostgreSQL superuser (the sketch uses `karttur`: tell the reader to use their own) |
| `postgresdb.dump_path` | Where `dump_restore_db.ipynb` writes the dump |
| `postgresdb.db_users` | The database roles and their `.netrc` machine names. Keep the role names. |
| `initial_user` | The first application user, stratum 5. Login and password come from `.netrc` (`user_netrc_id`). `organisation_id__organisation_name` and `territory_id__territory_name` must exist in the seed tables (`xspectre`, `sweden` do). |

### 4.2 `.netrc`

Fix the sketch: **every line starts with `machine`** (the host line in the sketch lacks it).

```
machine your_host_netrc_id login your_pg_superuser password your_pg_superuser_password
machine ini_user_synthetic login ini_user password change-me-1
machine community_admin login community_admin password change-me-2
machine login_evaluation login login_evaluation password change-me-3
machine user_cat_1 login user_cat_1 password change-me-4
...
machine user_cat_5 login user_cat_5 password change-me-8
```

Add:

- `chmod 600 ~/.netrc`: only you may read it.
- Database roles belong to the PostgreSQL server, not to one database. If you already set up another
  Xspatula database on the same server, the role lines exist already: keep them as they are. Setup
  gives the existing roles their rights in the new database.
- Passwords must not be written in the scheme files. The repos have a pre-commit hook that refuses
  JSON files with plaintext passwords.

### 4.3 Adding more initial users (optional), the section Thomas asked for

Facts from the code:

- `setup/zzz/lucas/setup_db/json/community/user_records_v10_sql.json` seeds `community.user`. Its only
  row has `"{initial_user}"` as password: setup replaces that row with the scheme's `initial_user`
  (login from `.netrc`, password bcrypt-hashed in memory, other keys such as email override the row).
- More users = more rows in `values`, same columns. `password` must be a **bcrypt hash**, never
  plaintext. Make one in Python:
  ```python
  from src.community import Hash_password
  print(Hash_password('the-new-users-password'))
  ```
- Seeds are insert-only: rows that exist are skipped. A user added after setup gets in by running
  `setup_db.ipynb` again (nothing existing is changed).
- **Recommend against this route** and say why: this seed file is shared by every database set up from
  `setup/zzz/lucas` (the LUCAS database too), so a user added there appears in all of them. For users of
  one database, use step 4.7 (`register_users.ipynb`).

### 4.4 Run `setup/setup_db.ipynb`

- Scheme cell: `scheme_file = './zzz/scheme_local_setup_synthetic.json'`.
- Answer `y` when asked whether to set up the database.
- Then run the audit cell (optional; it turns on the audit trail of inserts, updates and deletes).
- Setup rewrites the env files in `src/postgres/environment/` to point at `synthetic`. One checkout works
  with one database at a time; `python synthetic_dataset/switch_db.py lucas` switches back
  (`switch_db.py` without an argument shows the current one).
- Rebuilding from scratch: first `setup/delete_db.ipynb` with
  `scheme_file = './zzz/scheme_local_delete_synthetic.json'`. It drops the database `synthetic` and
  deletes the env files. Roles that other databases use are kept.

### 4.5 Use scheme `setup/zzz/scheme_local_use_synthetic.json`

The sketch's JSON is correct. `user_netrc_id` = the `.netrc` machine of the initial user.

### 4.6 Run `setup/setup_processes.ipynb`

The scheme cell is `scheme_file = './zzz/scheme_local_use_synthetic.json'`. It registers about 120
processes. If it stops with "permission denied" (the login role has no rights on `community`), rerun
`setup_db.ipynb`: it re-grants role rights every run. Since 2026-11-10 the message tells you this
itself, without a traceback.

### 4.7 Register an extra user, `synthetic_dataset/user_management/register_users.ipynb` (optional)

The project schemes (`synthetic_dataset/scheme_synthetic.json`, `scheme_synthetic_admin.json`) log in as
`ini_user_synthetic`, so this step is **not required**. It shows how to add an organisation
(`synthetic soil lab`) and a user (`dummy.user`, stratum 5) without e-mail:

- Cells 1–3: general utilities (territories), organisation, translate the user excel to JSON.
- Cell 4 writes a **bcrypt hash** of `dummy_password` into `user/manage_process/manage_user.json`; the
  plaintext is never stored. Tell the reader to change `dummy_password` before running.
- To work as that user, add `machine dummy_synthetic login dummy.user password <that password>` to
  `.netrc` and set `"user_netrc_id": "dummy_synthetic"` in `synthetic_dataset/scheme_synthetic.json`.

### 4.8 Import data, `synthetic_dataset/import_data/insert_synthetic_data.ipynb`

Scheme `../scheme_synthetic.json`. Cells:

| Cell | Inserts |
|---|---|
| 1.1–1.4 | utilities: general, observation, observation with inheritance (provisions, incl. `foss xds 10nm`), landscape (land cover, land use, BIOGEO16). 1.1 can be skipped if 4.7 ran. |
| 2 | dataset metadata: data source, person, dataset, the 2 campaigns and their sampling logs |
| 3.1–3.2 | geolocations and samples, calibration and validation |
| 4 | 8 observation logs |
| 5.1–5.2 | lab values (the USDA texture class is derived from clay/silt/sand on insert) |
| 6.1–6.2 | land cover, land use, BIOGEO16 |
| 7.1–7.3 | spectrometer (the 10 nm wavelength array), then 2000 + 300 spectra (one JSON per sample; this takes a while) |

Correct the sketch: spectra are **stored as diffuse reflectance**. They become absorbance at
compilation (`output_spectra_signal: absorb`, notebook 1). Everything except spectra goes from excel
via `insert_tabular_data`. A 211-band spectrum does not fit in an excel cell, so spectra use JSON files.

## 5. The ML chain (`synthetic_dataset/project_synthetic/`)

All notebooks use `scheme_file = '../scheme_synthetic.json'`. Process files are in `process/`, data go to
`data/`, models to `models/` (kept apart on purpose: see 5.4).

### 5.1 Notebook 1, compile and inspect

- 1.1: `compile_dataset_from_db` for the calibration set (`process/compile/compile_calibration.json`):
  7 indicators, absorbance 400–2500 nm at 10 nm, covariates `landcover`, `landuse`, `biogeo16`,
  `soil texture usda major group`, `covariate_rank: family`. Then `inspect` prints a summary.
  Output `data/compiled/synthetic_calibration_10nm/`.
- 1.2: the same for the validation set. It is **not preprocessed**: models carry their own preprocessing.
- 1.3: `plot_indicators` (box plots and histograms) and `plot_spectra` (raw, SNV and meancentred,
  100 spectra).
- **Correction to the sketch**: `transformation: standard` now **log-transforms c-org, n-tot and ec**
  (c-org was added 2026-11-10; it is right-skewed, and outlier removal does not remove the skew).
  Thomas is re-running cell 1.3; **ask him whether `1.3 Look at the calibration data box.png` and
  `1.3 Look at the calibration data.png` show the new transformation before using them.**
- Explain `covariate_rank: family`: a land cover code is reported at family level (e.g. sunflower → its
  crop family), which gives fewer, larger classes.

### 5.2 Notebook 2, preprocessing (calibration set only)

| Cell | Process | Output dataframe |
|---|---|---|
| 2.1 | `detect_outliers`, isolation forest on c-org, n-tot, ec, threshold 0.05 | `synthetic_calibration_10nm.olif` |
| 2.2 | `spectra_scatter_correction`, SNV | `….olif.snv` |
| 2.3 | `spectra_scaling`, meancentring | `….olif.snv.mc` |

- **Interactive prompts, the newcomer trap. Make this a `notice--info` box.** The cells wait for keyboard
  input. In VS Code the input box appears at the **top of the window**, not in the cell, and the cell
  looks hung until you answer. In order:
  1. 2.1: the outlier fraction (Enter keeps 0.05; the example used `0.025`),
  2. 2.1: which indicators' outliers to remove (`all`, or names),
  3. 2.2: save the SNV result? `y`,
  4. 2.3: save the meancentred result? `y`.
- Outliers are **set to missing for that indicator only**. The sample stays, with its spectrum and its
  other indicators.
- Each step reads `dataframe: previous`. If you don't save SNV, meancentring runs on the un-corrected
  data, and the plot shows it.
- Fix the sketch's typo: the final dataframe **will** be named `synthetic_calibration_10nm.olif.snv.mc`
  (parquet + JSON with the whole chain). The dot suffixes are its history: `olif` = indicator outliers
  (isolation forest), `snv` = scatter correction, `mc` = meancentring. Notebook 3 expects this name.
- Figures: use `2.1 Indicator outliers 0.025.png` (it matches the example). Mention that
  `0.05.png` is the default for comparison, or show both side by side.

### 5.3 Notebook 3, model

**Cell 3.1 (optional) `list_parameter_sets`**, which the sketch asked to explain. It lists the
parameter sets stored in the database (schema `ml_setting`): family (e.g. regressor hyperparameters,
indicator transformation, plot symbols), name, version and category (`raw` / `preset` / `custom`).
`family: ""` lists all families. The names you see are the values you can give in 3.2: `"default"` =
the preset `standard`, `name@version` = a specific version. It only reads; nothing changes. Link to
`/machine_learning/ml_settings/`.

**Cell 3.2 `regression_modeling`** (`process/ml_model/regression_modeling.json`). Show the JSON and a
parameter table:

| Parameter | Value | Meaning |
|---|---|---|
| `dataframe` | `synthetic_calibration_10nm.olif.snv.mc` | output of notebook 2 |
| `indicator_array` | c-org, n-tot, ph-water, clay | |
| `regressor_array` | plsr, svr, randfor, cubist | 16 models |
| `model_parameters_fp` | `default` | hyperparameters: preset `standard` |
| `indicator_transformation` | `standard` | log for c-org, n-tot, ec; predictions are back-transformed |
| `refit_preprocessing` | `true` | SNV + meancentring are refitted on the training samples and stored in the model |
| `test_size` | 0.3 | holdout by sample-id hash: the same samples every run |
| `traintest` / `kfold` | `true` / `false` | k-fold is slower; set `"kfold": true` to add it |
| `model_fp` | `./project_synthetic/models` | models are kept outside `data/` |

Key point for the reader: a saved model is a **complete pipeline**, raw spectra in and indicator values
out. That is why the validation set stays unprocessed. Output: `.joblib` + `.manifest.json` per model in
`models/synthetic_calibration_10nm.olif.snv.mc_tt/`, plus plots. There is no figure for 3.2 yet: leave a
placeholder.

### 5.4 Notebook 4, cells 4.1–4.2

**4.1 `predict_independent`** (`process/predict/predict_validation.json`): all 16 models on the 300
unprocessed validation spectra. The validation set has lab values, so the run reports independent
metrics and plots observed vs predicted (figure `4.1 Independent validation, all models.png`). Output in
`data/predictions/synthetic_validation_10nm/`: `_pred.parquet` (one column `<indicator>__<regressor>_tt`
per model) and `_pred.json` (models, sha256, metrics). Link to `/machine_learning/ml_predict/`.

**4.2 `register_model`** (`process/ml_model/register_model.json`) registers c-org PLSR as
`synthetic_c-org_plsr`. Say plainly: in real work you register only after testing preprocessing and
hyperparameters; it is done here to show how. Choose the model from the 4.1 metrics and change
`model_array`/`name` for another. Registration:

- stores path, sha256, manifest, preprocessing steps, metrics and spectral range in schema `ml_model`;
- **freezes** the parameter sets the model used (they can no longer be edited or deleted);
- is permanent: a model can only be retired, not deleted or registered twice.

**Thomas's question ("with the model you only need the independent dataframe to rerun the
predictions — correct?")**: yes, with two conditions. Put both in the docs:

1. The model **files** must stay. The registry holds the path and sha256 of the `.joblib`, not the file.
   Prediction needs the `.joblib` and its `.manifest.json` in `models/`. It does not need the calibration
   data or the preprocessed dataframes. So deleting `data/` is safe; deleting `models/` breaks the
   registered model.
2. The path is stored as an **absolute path**. Moving or renaming the repository folder breaks the
   registered model as well.

Re-creating the validation dataframe after deleting `data/` = re-run notebook 1, cell 1.2 (its compile
has `overwrite: false`; that is fine on an empty folder).

**4.3 Clean up the intermediate dataframes** (`process/cleanup/delete_dataframes_calibration.json`, the
existing process `delete_dataframes`, added to the notebook 2026-11-10):

```json
{
  "process": [
    {
      "process": "delete_dataframes",
      "parameters": {
        "project_root_fp": "./project_synthetic/data/compiled/synthetic_calibration_10nm",
        "keep_array": ["raw", "synthetic_calibration_10nm.olif.snv.mc"]
      }
    }
  ]
}
```

- It lists what it keeps and what it deletes, then asks `y/n` (another interactive prompt). Answer `y`.
- Deleted: `synthetic_calibration_10nm.olif` and `.olif.snv` (parquet + companion JSON, about 8 MB), and
  their entries in `.previous_dataframe`.
- Kept: the raw compiled set (selected datasets are never deleted) and `.olif.snv.mc`.
- The lesson: a model holds its own preprocessing, so intermediate steps are not needed once the chain is
  final. Their history stays in the companion JSON of `.olif.snv.mc` and in each model's manifest.
- Why `.olif.snv.mc` is kept: it is the modelling input. It carries the indicator values with outliers
  set to missing, and notebooks 5–8 train on it again. A new `regression_modeling` run needs it plus the
  raw set; the intermediate frames are not needed (tested 2026-11-10).
- Link `delete_dataframes` to the `_machine_learning` page that documents it, if one does; otherwise
  give its `keep_array` options: `raw`, `previous`, dataframe names.

**4.4 Predict with the registered model** (`process/predict/predict_validation_registered.json`):
`model_array: ["synthetic_c-org_plsr"]`, the registry name instead of a file path. It runs after the
cleanup, which shows the model does not need the calibration data. The model is checked against the
sha256 recorded at registration. The independent metrics are written to the registry (dataset
`independent:synthetic_validation_10nm`; a rerun replaces them). Output in
`data/predictions/synthetic_validation_10nm_registered/`. Show the SQL query from `ml_predict.md` (find
models by indicator and spectral range) as a "what you can do now" example, adapted to c-org.

**4.5 Retire the model** is disabled (`%%script false`). Say what it does (the name can no longer be
used for predictions; the record, metrics and frozen sets stay) and how to enable it (remove the first
line of the cell).

### 5.5 Notebooks 5–8, covariates (one page)

Each notebook repeats the chain of notebook 3 with **one nominal covariate**, predicts the validation set
with both model sets (spectra only, spectra + covariate), and compares them on the same samples. Write
the page in this order:

1. **The idea.** A covariate is a class per sample (region, land cover, …), one-hot encoded next to the
   spectra. Question: does it improve the predictions? Field relevance: in the app, region, land cover or
   a soil map class come from the location (Earth observation, maps), not from the lab.
2. **How it is set** (show `process/covariate_landcover/regression_modeling_landcover.json`, only the
   parameters that differ from notebook 3):
   ```json
   "covariate_array": ["landcover"],
   "class_grouping": [{"covariate": "landcover",
                       "groups": [["cultivated"], ["grass-l-no-trees", "grass-l-trees"],
                                  ["conif-woodl", "broadl-woodl", "mixed-woodl"]],
                       "other": "exclude"}]
   ```
   `groups` pools classes; `"other": "exclude"` leaves the samples of all unlisted classes out of training
   and prediction; `"other": "<name>"` would pool them into one group instead. Samples without a class,
   or with a class the model never saw, are not predicted. Link to the Covariates section of
   `/machine_learning/ml_model/`.
3. **The four groupings** (one table):

   | Notebook | Covariate | Groups | Left out |
   |---|---|---|---|
   | 5 | `biogeo16` | boreal+alpine; atlantic; continental+pannonian+steppic+black-sea; mediterranean | none (rare regions pooled: steppic 16, black-sea 3 calibration samples) |
   | 6 | `landcover` | cultivated; grassland (with/without trees); forest (coniferous, broadleaved, mixed) | agroforest, lichens, spontaneous vegetation, shrubland, swamp |
   | 7 | `landuse` | agriculture; forestry | natural/semi-natural, recreation, unused, mining |
   | 8 | `soil texture usda major group` | sandy, loamy, light clayey, heavy clayey (no grouping) | samples without texture (about 23%) |

4. **Cells**, written once, using biogeo16 as the example: X.1 `regression_modeling` (models in
   `models/synthetic_calibration_10nm.olif.snv.mc_cv-biogeo16_tt/`); X.2 `predict_independent` with both
   model sets (columns `<indicator>__<regressor>_tt` and `<indicator>__<regressor>+biogeo16_tt`, one plot
   row per model set); X.3 the comparison table: r² and RMSE on the common samples, `change` = with
   covariate − spectra only (negative RMSE change = better), then RMSE per group. Notebooks 6–8 are the
   same apart from the covariate: say so in one sentence and list their notebook and process-file names.
5. **Results**: one table of the r² change per indicator × regressor × covariate. **Take the numbers from
   the outputs of Thomas's current run** (ask him for them, or for the X.3 output cells). Do not use older
   numbers: c-org became log-transformed on 2026-11-10, which changes them.
6. **What it shows** (these hold across runs, from the 2026-11-09/10 runs with 2000 samples):
   - Region, land cover and land use add about nothing overall. The per-group RMSE can still show where a
     spectra-only model is biased.
   - Texture improves clay, but that is **leakage**: the texture class is computed from the measured clay,
     silt and sand. In the field it would come from a soil map, and the gain would be much smaller. Use
     this as the teaching point: check where a covariate comes from before trusting a gain.
   - Run-to-run differences of about ±0.02 r² are noise: treat smaller changes as no change.
   - The data are synthetic composites. Real LUCAS covariate effects may differ.

Figures: none in `/assets/media/synthetic/` yet. Leave placeholders for the X.2 grid plot (saved in
`data/predictions/synthetic_validation_10nm_<covariate>/plot/grid/`) and ask Thomas for them.

## 6. Figures available (`/assets/media/synthetic/`)

```
1.3 Look at the calibration data.png             (check: new c-org log transformation?)
1.3 Look at the calibration data box.png         (check: same)
1.3 Look at the calibration data spectra raw.png
1.3 Look at the calibration data spectra snv.png
1.3 Look at the calibration data spectra mc.png
2.1 Indicator outliers 0.025.png
2.1 Indicator outliers 0.05.png
2.2 SNV scatter correction.png
2.3 Meancentring.png
4.1 Independent validation, all models.png
```

The file names contain spaces: URL-encode them (`%20`) in the image links. Missing: 3.2 model plots,
4.2 registration output (a code block with the printed output is enough for that one).

## 7. Open for Thomas (put these in your final report, don't decide them)

- Whether the 1.3 figures show the c-org log transformation.
- Figures for 3.2, 4.4 and the covariate grids; the numbers for the results table in 5.5.
- How readers will get the dataset once permission is granted (section 2's notice changes then).

## Work log

### 2026-11-11 (done, not committed)
- Decisions (Thomas): nav title "Try Xspatula"; links out to lucas/core docs as full URLs (collection movable); nav entry + landing line.
- `_synthetic/`: synopsis, setup_database, import_data, compile, preprocess, model, predict_register, covariates (permalinks `/synthetic/<page>/`).
- `_config.yml`: collection + nav_order; `_data/navigation.yml`: main_navigation entry, `synthetic` sidebar, `main` (top menu) entry. All 5 blocks carry the "remove this block" comment; README has a "Synthetic tutorial" section with the table.
- Open items from §7 resolved from the code repo, not by asking: 1.3 figures do show the log transform (all figures postdate commit 7f5eb00); results table from the 5.3–8.3 outputs of the 2026-11-10 run; 3.2, 4.4 and covariate figures exist now, so no placeholders were needed.
- Deviations from this note: setup scheme shown with `my_pg_superuser` (file has a personal id); 4.3 seed users section kept, with the recommendation against it; "what it shows" reworded to "add little" (landcover reaches ±0.06–0.07 for single models); n-tot gain with texture noted.
- Model folder suffix for texture is `_cv-soiltextureusdamajorgroup_tt`.
- Verified: production build, 0 broken internal links, 20/20 figures, 15 external links + anchors live.
- Code feedback: `xspatula_lucas/notes/comments_from_xspatula_lucas_docs_synthetic_20261111.md`.
