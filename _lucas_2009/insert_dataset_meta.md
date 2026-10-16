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

**Path**: `xspatula_lucas/lucas/import_data/insert_lucas_data.ipynb`, block **2. Dataset metadata**

Four cells, each an `insert_tabular_data` process file — translate and insert in one step:

| Cell | Process file | Detail |
|---|---|---|
| 2.1 Insert data source | `import_data/dataset_meta/insert_process/data_source.json` | [Data source][dm_data_source] |
| 2.2 Insert persons | `import_data/dataset_meta/insert_process/person.json` | [Person][dm_person] |
| 2.3 Insert dataset | `import_data/dataset_meta/insert_process/dataset.json` | [Dataset][dm_dataset] |
| 2.4 Insert campaigns | `import_data/dataset_meta/insert_process/campaign.json` | [Campaign][dm_campaign] |

Run the four cells in order — each depends on the one before it. Block 2 covers both LUCAS 2009
and 2015: `campaign.xlsx` holds the LUCAS campaign and the BIOGEO16 campaign.

## Full detail

Each step's process file, source Excel columns, and parameter table is documented in
[Dataset metadata explained][dataset_meta_explained] — including the `inherit` value
`campaign.xlsx` uses for `contact_name`/`contact_email`, explained further in
[Utility → inherit and auto][utility_inherit_auto].

## Next step

Proceed to [Load LUCAS 2009][load_lucas_2009].

[dataset_meta_explained]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/
[utility_inherit_auto]: {{ site.baseurl }}/lucas_2009/utility_inherit_auto_explained/
[insert_utility]: {{ site.baseurl }}/lucas_2009/insert_utility/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[dm_data_source]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/#data-source
[dm_person]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/#person
[dm_dataset]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/#dataset
[dm_campaign]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/#campaign
