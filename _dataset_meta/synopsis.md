---
title: "Dataset Metadata"
layout: single
sidebar:
  nav: "dataset_meta"
permalink: /dataset_meta/
author_profile: false
---

Dataset metadata describes the provenance of the LUCAS soil observations: who published them,
who is the named contact, and under what campaign they were collected. This chain of records
must be in place before any sample or observation data can be entered.

Loaded by a single notebook:

```
xspatula_lucas/lucas/import_data/insert_lucas_dataset_meta.ipynb
```

## The dataset hierarchy

```
data_source              (ESDAC-JRC, the publisher)
    └── dataset          (LUCAS topsoil data — one row per LUCAS survey round)
          └── campaign   (one bounded round, e.g. lucas_eu_2009)
```

Persons are registered independently, linked to a data source, and referenced from other tables
by contact name/email.

| Table | Description |
|---|---|
| `observation.data_source` | The organisation providing the data — here, ESDAC-JRC |
| `observation.person` | Named contacts responsible for the data |
| `observation.dataset` | Top-level grouping — the LUCAS topsoil dataset as a whole |
| `observation.campaign` | A single bounded LUCAS survey round (2009, 2015, ...) |

## Source files

All four Excel files are in `xspatula_lucas/lucas/import_data/dataset_meta/excel/`:

| File | Database table |
|---|---|
| `data_source.xlsx` | `observation.data_source` |
| `person.xlsx` | `observation.person` |
| `dataset.xlsx` | `observation.dataset` |
| `campaign.xlsx` | `observation.campaign` |

Each is translated and inserted in one step via the `insert_tabular_data` process.

## Required loading sequence

Utility catalogues must already exist (see [LUCAS 2009 → Insert utility][insert_utility] —
`campaign.xlsx` and `dataset.xlsx` reference them by foreign key). Then, data source and person
before dataset; dataset before campaign:

1. [Manage data source] — insert the ESDAC-JRC data source record
2. [Manage person] — insert person record(s) (requires data source)
3. [Manage dataset] — insert the LUCAS topsoil dataset record (requires data source)
4. [Manage campaign] — insert campaign record(s), e.g. `lucas_eu_2009` (requires dataset)

After this, continue with [Load LUCAS 2009][load_lucas_2009] for the campaign-specific sampling
data, or [Samples explained] and [Observations explained] for what the resulting tables look
like.

[Manage data source]: /dataset_meta/manage_data_source/
[Manage person]: /dataset_meta/manage_person/
[Manage dataset]: /dataset_meta/manage_dataset/
[Manage campaign]: /dataset_meta/manage_campaign/
[insert_utility]: /lucas_2009/insert_utility/
[load_lucas_2009]: /lucas_2009/load_lucas_2009/
[Samples explained]: /lucas_2009/samples_explained/
[Observations explained]: /lucas_2009/observations_explained/
