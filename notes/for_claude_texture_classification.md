# TASK: document `@`-coded observation values and on-the-fly texture classification

Written 2026-10-19. Source of truth is the sibling code repo `xspatula_lucas` (commit `f4263d8`,
"Report missing database entries in texture classification hook; register major groups"). The
code is byte-identical with `xspatula_ai4sh`, so the same facts are documented there too, with
ai4sh examples (`xspatula_ai4sh_docs/notes/for_claude_texture_classification.md`). Keep the two
explanations consistent.

## Where to write

| Page | Permalink | What to add |
|---|---|---|
| `_lucas_2009/observations_explained.md` | `/lucas_2009/observations_explained/` | Full explanation: a new subsection on `@` values (expand the existing "Two kinds of observation" table and the paragraph under "Laboratory observations"), and a new `## Derived soil texture classes` section after "Laboratory observations", before "Spectral observations". Add a third row to the "Two kinds of observation" table (or rename it) for derived classes → `observation.observation_classification`. |
| `_lucas_2015/load_lucas_2015.md` | `/lucas_2015/` | Short: one bullet in "What the script does differently" (or right after the texture-backfill bullet) saying texture classes are derived the same way as 2009, link to the 2009 section. Mention that for revisited points with backfilled 2009 texture, the 2015 texture class is also derived from those 2009 values. |
| `_lucas_2009/utility_explained.md` | | Check the `nominal_classes` entry (added 2026-10-07) and the indicator/provision_indicator description agree with the facts below: quantity `class`, unit `class`, analysis method `derived`, four texture indicators, 32 nominal classes. |

Also point readers to the general ai4sh explanation of the prefix:
<https://xspatula.github.io/xspatula_ai4sh_docs/observation/translate_agrolab/#the--prefix-for-observation-values>.
That page does **not** cover texture classification, so this repo must explain it itself.

## Facts: `@`-coded observation values

- In a `manage_observation` process (and in the source Excel/CSV), any parameter whose name starts
  with `@` is an observation value, not a column of `observation.observation`.
- The text after `@` must be the **name or alias** of an indicator registered for the
  observation's provision in `observation_utility.provision_indicator`
  (`lucas/import_data/utility/observation/excel/provision_indicator.xlsx`). Lookup:
  `_Retrieve_quantity_indicator_id_from_provision` in `src/postgres/pg_ai4sh.py`.
- An `@` key that is not registered for the provision **stops the import** with
  `No indicator found for name/alias: <key>`.
- Value handling, per `@` key:
  - number → stored as is;
  - string starting with `<` (below detection limit, e.g. `"<6"`) → stored as **half the limit**
    (`3.0`). For LUCAS 2009 the prepare script already does this; the database loader does it
    again for any `<` value it still receives;
  - other non-numeric string → warning `⚠️ Value for indicator … is not a number`, value skipped;
  - missing/empty → nothing stored for that indicator.
- Where it is stored:
  - scalar value → `observation.observation_measurement` (one row per observation × indicator);
  - list value (spectra) → `observation.observation_measurement_array`.
- The unit is not in the observation record; it comes from the provision's `provision_indicator`
  row.

## Facts: on-the-fly texture classification

### What happens

After each laboratory observation's `@` values are written to `observation_measurement`, the
loader (`_Insert_derived_classifications` in `src/ai4sh/import_data/import_data.py`) derives soil
texture classes from the same `@` values and stores them. No extra notebook cell, no extra process
file: it happens inside the normal lab-observation load (LUCAS 2009 cell 3.4.2, LUCAS 2015 cell
4.4.2 of `insert_lucas_data.ipynb`).

Classifier code: `src/ai4sh/import_data/nominal_classification.py`.

### Four derived indicators

| Indicator (alias) | Classes | Meaning |
|---|---|---|
| `soil texture usda` | 12 | USDA texture triangle |
| `soil texture usda major group` | 4 | aggregation of the 12 USDA classes |
| `soil texture iuss` | 12 | IUSS (ISSS) texture triangle |
| `soil texture iuss major group` | 4 | aggregation of the 12 IUSS classes |

Major groups: heavy clayey soils, light clayey soils, loamy soils, sandy soils. Take the exact
class names (and their sand/silt/clay ranges, kept as text in `abstract`) from
`lucas/import_data/utility/observation/excel/nominal_classes.xlsx` — do not retype them from
memory.

### Inputs and formulas

Inputs are the `@` values of the same observation: `@clay`, `@silt`, `@sand`, or the split fractions
`@fine silt`, `@coarse silt`, `@fine sand`, `@coarse sand` (ai4sh/Agrolab style). LUCAS has only
`@clay`, `@silt`, `@sand`.

- **USDA**: sand = `@sand` (else `@fine sand` + `@coarse sand`), silt = `@silt` (else fine + coarse
  silt), clay = `@clay`.
- **IUSS**: the IUSS silt/sand boundary is 0.02 mm, not 0.05/0.06 mm, so coarse silt counts as sand.
  - All four split fractions present: sand = coarse silt + fine sand + coarse sand, silt = fine silt.
  - Otherwise (**LUCAS**): approximation sand = SAND + SILT/2, silt = SILT/2. Half the measured
    silt is assumed to be coarse silt.
- **Sanity check**: if any of sand/silt/clay is missing, or sand + silt + clay (before the IUSS
  transform) is outside **98–102 %**, no class is derived for that observation. Not an error;
  that observation simply gets no texture class.
- A below-detection string (`"<x"`) in a texture fraction is not halved by the classifier: it
  counts as missing, so no class is derived for that observation.

### Known small error (document it plainly)

- **USDA, sand ≥ 70 % corner**: the USDA decision tree uses box limits there (sand > 85 and
  clay < 8 → sand; clay < 12 → loamy sand; else sandy loam) instead of the true diagonal lines of
  the triangle. A small number of samples near the sand / loamy sand / sandy loam boundaries can get
  the neighbouring class. Elsewhere the USDA tree matches the reference table.
- **IUSS** is exact against the IUSS polygon vertices (tested on all 5151 integer sand+silt+clay=100
  points). For LUCAS the input itself is approximate (the SILT/2 split above), which is a larger
  source of uncertainty than the classifier.

### Opt-in per provision

A derived class is stored only if its indicator is registered for the observation's provision in
`provision_indicator.xlsx`, with analysis method `derived` and unit `class`. In LUCAS all four
indicators are registered for both `lucas wetlab 2009` and `lucas wetlab 2015`. Removing a row
(e.g. a major group) switches that indicator off for that provision.

Database prerequisites (all inserted by the observation utility insert, `insert_observation_utility.txt`):

1. quantity `class` (`quantity.xlsx`) and unit `class` (`unit.xlsx`);
2. the four indicators, quantity `class` (`indicator.xlsx`);
3. analysis method `derived` (`analysis_method.xlsx`);
4. the 32 classes in `observation_utility.nominal_classification` (`nominal_classes.xlsx`,
   via `nominal_classes.json`);
5. the `provision_indicator` rows.

### Where it is stored

`observation.observation_classification` (note: **not** `observation_measurement`):

| Column | Content |
|---|---|
| `id` | BIGSERIAL |
| `observation_id` | → `observation.observation(id)`, the lab observation |
| `indicator_id` | → `observation_utility.indicator(id)`, e.g. `soil texture usda` |
| `value_id` | → `observation_utility.nominal_classification(id)`, the class, e.g. `silt loam` |

`UNIQUE (observation_id, indicator_id)`: one class per observation and texture system. Audited for
UPDATE/DELETE. So a lab observation with valid texture gets up to 4 rows. Include a short example
SQL joining `observation_classification` → `indicator` and `nominal_classification` to list
`observation_id, indicator, class`.

### Re-runs and errors

- Re-running the lab-observation cell on observations that already exist derives and stores missing
  classes (existing rows are left alone unless `overwrite` is set). No delete needed after fixing
  the utility data.
- If texture can be derived but none of the four indicators is registered for the provision, the
  loader prints an ERROR at every verbosity level, **once per provision per run**, listing what is
  missing (quantity `class`, indicator, provision_indicator row), and continues. Same for a
  `provision_indicator` row without analysis method and for a derived class missing in
  `nominal_classes.xlsx`. Show the error text as an example (copy from the ai4sh notes file).

### Selection

Texture classes are used in class-filtered selection: `texture_class_array` in the select process,
classification given by `texture_classification` (default `soil texture usda`). Mention this with a
link to the select documentation if a page for it exists; otherwise one sentence.

## Style

Follow the existing page style: reference tone, tables for mappings, notebook cell numbers in bold,
reference-style links at the bottom. Keep the 2015 addition short and link back to 2009. Update
`last_modified_at` where the front matter has it.

## After the work

Append a dated summary of what was changed to this file, as with the other TASK files.

## Done 2026-10-19

- `_lucas_2009/observations_explained.md`: "Two kinds of observation" renamed "Three kinds of
  observation value", third row for `observation_classification`; new subsection "The `@` prefix"
  (registration lookup, stop-on-unregistered, value-handling table, link to ai4sh page); new
  `## Derived soil texture classes` (indicators, major-group table, formulas, sanity check,
  accuracy, opt-in, prerequisites table with cells 1.2/1.3, storage table + SQL, re-runs/errors with
  example, selection link).
- `_lucas_2015/load_lucas_2015.md`: bullet after texture backfill, links to 2009 section; backfilled
  2009 texture → 2015 class derived from 2009 values.
- `_lucas_2009/utility_explained.md`: was only listing `nominal_classes`; added short paragraph on
  the 32 classes / four indicators / quantity+unit `class` / method `derived`, and a paragraph on
  the derived rows in `provision_indicator` (cell 1.3).
- `_machine_learning/select_data.md`: one bullet linking texture filter to the derivation section.
- No page has `last_modified_at`; nothing to update. Jekyll build clean, anchors verified.

Deviations from the notes above (checked against code at `xspatula_lucas` HEAD `3959dff`):

- `provision_indicator` is inserted by cell 1.3 (`insert_observation_utility_inherit.txt`), not by
  `insert_observation_utility.txt`.
- `"<x"` values are NOT treated as missing by the classifier: `_Retrieve_quantity_indicator_id_from_provision`
  halves them in place in `at_columns_D`, the same dict then passed to
  `_Insert_derived_classifications`. Documented as "class derived from the halved value".
- Class sand/silt/clay ranges in `nominal_classes.xlsx` `abstract` were not copied: the IUSS
  abstracts contradict `iuss_by_tg` (e.g. sandy clay loam clay 25–45 in xlsx vs 15–25 in code;
  loam clay 0–25 vs <15; silty clay/light clay clay 45–65 vs 25–45). Page lists class names and
  major groups only. Fix the xlsx abstracts in the code repo, then ranges can be added.
