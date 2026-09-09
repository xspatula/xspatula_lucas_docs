---
title: "Foreign Key Explained"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/foreign_key_explained/
author_profile: false
---

Reference page — not required reading to complete any of the LUCAS 2009 steps. Explains
`utility.foreign_key`, a catch-all table Xspatula falls back to for resolving a foreign key by
name when the normal `xxx_id__yyy` column convention can't work directly — and the one place this
project actually needs it: `unit_translate`.

## How foreign keys normally resolve

A process parameter named `xxx_id__yyy` (e.g. `dataset_id__dataset_name`) is read as: write the
result into the `xxx_id` column, and find it by searching column `yyy` of the table literally
named `xxx` for the value given. This works as long as a table named `xxx` actually exists.

## When it can't — and how `utility.foreign_key` steps in

Some parameters don't have a real table matching their `xxx` prefix. `unit_translate`'s two foreign keys are the case in point: its parameters include `src_unit_id__unit_name` and
`dst_unit_id__unit_name`, but there's no table called `src_unit` or `dst_unit` — both should really resolve against `observation_utility.unit`, just via two differently-named columns on the same `unit_translate` row.

When the resolver (`_Check_get_foreign_key()` in `src/postgres/pg_common.py`) can't find a table
named after the `xxx` prefix, it falls back to querying `utility.foreign_key` for a row whose `foreign_key` column equals `xxx_id` — that row tells it which schema, table, and column to search instead.

## The `utility.foreign_key` table

**Source**: `lucas/import_data/utility/general/excel/foreign_key.xlsx`, inserted by cell 1 of
[Insert utility][insert_utility] (`general/insert_process/foreign_key.json`, via the
`manage_foreign_key` process) — see [Utility explained][utility_explained].

| Column | Description |
|---|---|
| `foreign_key` | The parameter name being resolved, e.g. `src_unit_id` (primary key — one row per resolvable name) |
| `dst_schema` | Schema of the table to actually search |
| `dst_table` | Table to actually search |
| `dst_search_column` | Primary column to match the given value against |
| `dst_alt_search_column` | Fallback/alias column to match against if the primary column misses |

Current content — two rows, both needed for `unit_translate` (see below):

| `foreign_key` | `dst_schema` | `dst_table` | `dst_search_column` | `dst_alt_search_column` |
|---|---|---|---|---|
| `dst_unit_id` | `observation_utility` | `unit` | `name` | `alias` |
| `src_unit_id` | `observation_utility` | `unit` | `name` | `alias` |

## Unit translate

`observation_utility.unit_translate` defines the mathematical conversion between two units — for
example millimetres to metres. Its two foreign keys, `src_unit_id__unit_name` and
`dst_unit_id__unit_name`, are exactly the case above: both resolve against
`observation_utility.unit`, via `utility.foreign_key`'s two rows shown above. `src_unit_id__unit_name
= "mm"` looks up `name = 'mm'` (or `alias = 'mm'`) in `observation_utility.unit` and substitutes
its id — same for the destination unit.

**Source**: `lucas/import_data/utility/observation/excel/unit_translate.xlsx`

| `src_unit_id__unit_name` | `dst_unit_id__unit_name` | `factor` | `addon` | `exponent` |
|---|---|---|---|---|
| `mm` | `m` | `1000` | `0` | `1` |
| `cm` | `m` | `100` | `0` | `1` |
| `dm` | `m` | `10` | `0` | `1` |
| `km` | `m` | `0.001` | `0` | `1` |
| `g/Kg` | `percent` | `0.1` | `0` | `1` |
| `percent` | `g/Kg` | `10` | `0` | `1` |

The conversion formula is `dst_value = (src_value * factor + addon) ** exponent`.

Loaded as part of [Insert utility][insert_utility]'s cell 2, in the dependent tier — after `unit`
and after `foreign_key` (cell 1) has already populated the two rows above. See [Utility explained][utility_explained] for the full cell sequence.

### Adding a new unit translation

1. Add a row to `unit_translate.xlsx` with the source unit, destination unit, and conversion
   `factor`/`addon`/`exponent`.
2. If either unit doesn't already have a `utility.foreign_key` row (only needed the first time —
   the two rows above cover every `unit_translate` row, since they're keyed by parameter name,
   `src_unit_id`/`dst_unit_id`, not by which units are being converted), add it to
   `foreign_key.xlsx` first.
3. Re-run `insert_utility.ipynb` in full, or add a new notebook cell that just inserts the new translation into `unit_translate`:

   ```python
   process_file = 'import_data/utility/observation/insert_process/unit_translate.json'

   structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

   if structured_process_D is not None:
       Run_process(structured_process_D, scheme_params_D)
   ```

   **WARNING** Do **not** set `"overwrite: true"` in the JSON process file as this will cause irreparable changes; you are inserting a new translation and it will be added with `"overwrite: false"` while all existing rows will remain unchanged.

   If you added a new `foreign_key.xlsx` row in step 2, re-run cell 1 (`foreign_key`) first — it must exist before `unit_translate` can resolve.

[insert_utility]: /lucas_2009/insert_utility/
[utility_explained]: /lucas_2009/utility_explained/
