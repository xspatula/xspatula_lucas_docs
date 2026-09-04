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

## Notebook

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_dataset_meta.ipynb`

Four cells, each an `insert_tabular_data` process file — translate and insert in one step, no
separate translate/manage split:

1. Insert data source (`import_data/dataset_meta/insert_process/data_source.json`)
2. Insert persons (`import_data/dataset_meta/insert_process/person.json`)
3. Insert dataset meta (`import_data/dataset_meta/insert_process/dataset.json`)
4. Insert campaigns (`import_data/dataset_meta/insert_process/campaign.json`)

Set `scheme_file = '../scheme_lucas.json'` in the notebook's setup cell (already the default),
then run all four cells in order — each depends on the one before it.

## Full detail

Each step's process file, source Excel columns, and parameter table is documented in
[Dataset metadata][dataset_meta]:

- [Manage data source]
- [Manage person]
- [Manage dataset]
- [Manage campaign] — **read this one before continuing**: it flags an overlap with the campaign
  record `lucas_2009_to_xspatula.py` also generates.

## Next step

Proceed to [Insert utility][insert_utility].

[dataset_meta]: /dataset_meta/
[Manage data source]: /dataset_meta/manage_data_source/
[Manage person]: /dataset_meta/manage_person/
[Manage dataset]: /dataset_meta/manage_dataset/
[Manage campaign]: /dataset_meta/manage_campaign/
[insert_utility]: /lucas_2009/insert_utility/
