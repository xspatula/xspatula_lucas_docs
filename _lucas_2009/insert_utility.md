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

The arguments with double underscore denote database foreign key linkages. These are translated and verified by Xspatula before the data is inserted.

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_data.ipynb`, block **1. Utilities**

Four cells, each calling a `job_file`, run **in this order**:

1.1 **Insert general utilities** — `import_data/utility/job_insert_general_utility.json`
   (territory codes and other shared lookups)
1.2 **Insert observation utilities** — `import_data/utility/job_insert_observation_utility.json`
   (indicators, units, methods, provisions, preparation, preservation, storage etc. — in dependency
   order: utilities without foreign keys first)
1.3 **Insert observation utilities with inheritance** —
   `import_data/utility/job_insert_observation_utility_inherit.json` (utility tables that
   optionally inherit values from the tables loaded in 1.2 — must run after them)
1.4 **Insert landscape utility** — `import_data/utility/job_insert_landscape_utility.json`
   (`biogeo16`, then `landuse` and `landcover`: one insert per scheme, each holding all ranks
   order > family > genus in one table; the ranks themselves, `class_rank`, are created by the
   database setup)

Each cell's `insert_tabular_data` process translates and inserts its source tables from
`import_data/utility/general/`, `import_data/utility/observation/`, and
`import_data/utility/landscape/` in one step.

Set `scheme_file = '../scheme_lucas.json'` in the setup cell (already the default), then run the
four cells top to bottom.

This is a one-time bootstrap: once these catalogues exist in your database, you don't need to
re-run block 1 for later campaigns or additional LUCAS survey rounds — only when a new utility
value (a new provision, a new unit etc.) needs registering. To skip the block on a later run,
switch its cells off with `%%script false --no-raise-error` (remove the `#` in front of the first
line of each cell; see [the synopsis][synopsis_partial]).

For exactly which tables each cell inserts, and which ones ship disabled, see
[Utility explained][utility_explained]. For the `inherit`/`auto` special values a couple of this
project's spreadsheets use, see [Utility → inherit and auto][utility_inherit_auto].

## Next step

Proceed to [Insert dataset metadata][insert_dataset_meta].

[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[utility_explained]: {{ site.baseurl }}/lucas_2009/utility_explained/
[utility_inherit_auto]: {{ site.baseurl }}/lucas_2009/utility_inherit_auto_explained/
[synopsis_partial]: {{ site.baseurl }}/lucas_2009/#data-loading-pipeline
