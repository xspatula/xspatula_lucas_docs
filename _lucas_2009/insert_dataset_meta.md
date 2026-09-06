---
title: "Insert Dataset Metadata"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/insert_dataset_meta/
author_profile: false
---

Registers who published the data, the top-level LUCAS dataset, and the campaign record — all
from hand-authored Excel files, not from the generated CSV output.

## Prerequisites

- [Insert utility][insert_utility] must be complete — `campaign.xlsx` and `dataset.xlsx` both
  reference utility values (territory, license, spatial reference) by foreign keys.

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_dataset_meta.ipynb`

Four cells, each an `insert_tabular_data` process file — translate and insert in one step:

1. Insert data source (`import_data/dataset_meta/insert_process/data_source.json`)
2. Insert persons (`import_data/dataset_meta/insert_process/person.json`)
3. Insert dataset meta (`import_data/dataset_meta/insert_process/dataset.json`)
4. Insert campaigns (`import_data/dataset_meta/insert_process/campaign.json`)

Set `scheme_file = '../scheme_lucas.json'` in the notebook's setup cell (already the default),
then run all four cells in order — each depends on the one before it.

## Full detail

Each step's process file, source Excel columns, and parameter table is documented in
[Dataset metadata explained][dataset_meta_explained] — including the `inherit` value
`campaign.xlsx` uses for `contact_name`/`contact_email`, explained further in
[Utility → inherit and auto][utility_inherit_auto].

## Next step

Proceed to [Load LUCAS 2009][load_lucas_2009].

[dataset_meta_explained]: /lucas_2009/dataset_meta_explained/
[utility_inherit_auto]: /lucas_2009/utility_inherit_auto_explained/
[insert_utility]: /lucas_2009/insert_utility/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
