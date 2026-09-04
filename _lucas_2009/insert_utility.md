---
title: "Insert Utility"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/insert_utility/
author_profile: false
---

Populates the lookup catalogues — territories, units, methods, provisions, and more — that every
other insert in this pipeline depends on via foreign key. **Must** be loaded before
[Load campaign][load_campaign], since that step references utility values like
`territory_id__territory_name`, `provision_id__provision_name`, and
`wavelength_unit_id__wavelength_unit_name`.

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/insert_utility.ipynb`

Four cells, each a `job_file` (not a single process file — these run a whole pilot of process
files each), run **in this order**:

1. **Insert general utilities** — `import_data/utility/job_insert_general_utility.json`
   (territory codes and other shared lookups)
2. **Insert observation utilities** — `import_data/utility/job_insert_observation_utility.json`
   (indicators, units, methods, provisions, preparation, preservation, storage — in dependency
   order: utilities without foreign keys first)
3. **Insert observation utilities with inheritance** —
   `import_data/utility/job_insert_observation_utility_inherit.json` (utility tables that
   optionally inherit values from the tables loaded in step 2 — must run after them)
4. **Insert landscape utility** — `import_data/utility/job_insert_landscape_utility.json`
   (land use, land cover, land form, soil texture, biogeography catalogues)

Each cell's `insert_tabular_data` process translates and inserts its source tables from
`import_data/utility/general/`, `import_data/utility/observation/`, and
`import_data/utility/landscape/` in one step — no separate translate/manage cells.

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default), then run all
four cells top to bottom.

This is a one-time bootstrap: once these catalogues exist in your database, you don't need to
re-run this notebook for later campaigns or additional LUCAS survey rounds — only when a new
utility value (a new provision, a new unit) needs registering.

## Next step

Proceed to [Load campaign][load_campaign].

[load_campaign]: /lucas_2009/load_campaign/
