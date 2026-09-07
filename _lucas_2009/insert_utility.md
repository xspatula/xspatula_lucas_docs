---
title: "Insert Utility"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/insert_utility/
author_profile: false
---

Populates the lookup catalogues — territories, units, methods, provisions, and more — that every
other insert in this pipeline depends on via foreign keys. **Must** be loaded first, before
[Insert dataset metadata][insert_dataset_meta] and [Load LUCAS 2009][load_lucas_2009]: even the dataset metadata step references utility values like `territory_id__territory_name`, `license_id__license_name`, and `spatial_reference_id__spatial_reference_name`, and the load step references more (for instance `provision_id__provision_name`, `wavelength_unit_id__wavelength_unit_name`).

The arguments (excel columns) with double underscore denote database foreign key linkages. These are translated and verified before the data is inserted.

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/insert_utility.ipynb`

Four cells, each calling a `job_file`, run **in this order**:

1. **Insert general utilities** — `import_data/utility/job_insert_general_utility.json`
   (territory codes and other shared lookups)
2. **Insert observation utilities** — `import_data/utility/job_insert_observation_utility.json`
   (indicators, units, methods, provisions, preparation, preservation, storage etc. — in dependency
   order: utilities without foreign keys first)
3. **Insert observation utilities with inheritance** —
   `import_data/utility/job_insert_observation_utility_inherit.json` (utility tables that
   optionally inherit values from the tables loaded in step 2 — must run after them)
4. **Insert landscape utility** — `import_data/utility/job_insert_landscape_utility.json`
   (land use, land cover, land form, soil texture, biogeography etc.)

Each cell's `insert_tabular_data` process translates and inserts its source tables from
`import_data/utility/general/`, `import_data/utility/observation/`, and
`import_data/utility/landscape/` in one step.

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default), then run all
four cells top to bottom.

This is a one-time bootstrap: once these catalogues exist in your database, you don't need to
re-run this notebook for later campaigns or additional LUCAS survey rounds — only when a new
utility value (a new provision, a new unit etc.) needs registering.

For exactly which tables each cell inserts, and which ones ship disabled, see
[Utility explained][utility_explained]. For the `inherit`/`auto` special values a couple of this
project's spreadsheets use, see [Utility → inherit and auto][utility_inherit_auto].

## Next step

Proceed to [Insert dataset metadata][insert_dataset_meta].

[insert_dataset_meta]: /lucas_2009/insert_dataset_meta/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
[utility_explained]: /lucas_2009/utility_explained/
[utility_inherit_auto]: /lucas_2009/utility_inherit_auto_explained/
