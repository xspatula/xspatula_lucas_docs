---
title: "Dataset Metadata Explained"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/dataset_meta_explained/
author_profile: false
---

Reference page — not required reading to complete the [Insert dataset metadata][insert_dataset_meta]
step, only if you want the detail behind the data source, person, dataset, and campaign records
it inserts.

Dataset metadata describes the provenance of the LUCAS soil observations: who published them, who
the named contact is, and under what campaign they were collected. This chain of records must be
in place before any sample or observation data can be entered — and before it, the
[utility catalogues][utility_explained] must already exist, since these records reference several
of them by foreign key (territory, license, spatial reference).

## The hierarchy

```
data_source              (ESDAC-JRC, the publisher)
    └── dataset          (LUCAS topsoil data — one row for all survey rounds)
          └── campaign   (the place, e.g. lucas european union)
                └── sampling_log   (one survey round, e.g. lucas european union 2009 — written by the prepare script)
```

Persons are registered independently, linked to a data source, and referenced from other tables
by contact name/email. The persons registered here do not need to be registered as users.

| Table | Description |
|---|---|
| `observation.data_source` | The organisation providing the data — here, ESDAC-JRC |
| `observation.person` | Named contacts responsible for the data |
| `observation.dataset` | Top-level grouping — the LUCAS topsoil dataset as a whole |
| `observation.campaign` | The place sampled — LUCAS in the EU; each survey round (2009, 2015, ...) is a sampling log under it |

All four are inserted in one step each via `insert_tabular_data` — translate and insert in a
single call, no separate translate/manage split. Every process file follows the same shape, e.g.
data source's:

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_data_source",
        "tabular_data_path": "import_data/dataset_meta/excel/data_source.xlsx",
        "dst_path": "import_data/dataset_meta/insert_process/staging"
      }
    }
  ]
}
```

## Data source

**Notebook cell**: 2.1 Insert data source in `insert_lucas_data.ipynb`

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/data_source.xlsx` — one row:

| Column | Value |
|---|---|
| `name` | `european soil data centre (esdac), european commission, joint research centre jrc)` |
| `alias` | `esdac-jrc` |
| `display_name` | `ESDAC-JRC` |
| `url` | `https://esdac.jrc.ec.europa.eu` |
| `territory_id__territory_name` | `eu` |
| `contact_name` | `ESDAC - European Commission` |
| `contact_email` | `ec-esdac@jrc.ec.europa.eu` |

`alias` (`esdac-jrc`) is the value later referenced by `data_source_id__data_source_name` foreign
keys in `person.xlsx` and `dataset.xlsx`.

## Person

**Notebook cell**: 2.2 Insert persons in `insert_lucas_data.ipynb`

LUCAS ships one placeholder contact tied to ESDAC-JRC — edit the Excel row before running this step for a production import. The person to register does not need to be registered as a user, it is a contact person for dataset details. If there is no person responsible you can also add _anonymous_.

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/person.xlsx`

| Column | Value (shipped placeholder) |
|---|---|
| `data_source_id__data_source_name` | `esdac-jrc` (FK to data source, above) |
| `first_name` / `middle_name` / `last_name` | `fn` / `mn` / `ln` — placeholder, edit before real use |
| `email` | `ec-esdac@jrc.ec.europa.eu` |
| `territory_id__territory_name` | `eu` |

## Dataset

**Notebook cell**: 2.3 Insert dataset in `insert_lucas_data.ipynb`

The dataset record is the umbrella for all LUCAS topsoil data, spanning every survey round. One
row covers the whole dataset. Underneath it, the campaign is the place (the EU) and each survey
round (2009, 2015, ...) is a sampling log of that campaign.

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/dataset.xlsx` — one row:

| Column | Value |
|---|---|
| `name` | `land use and coverage area frame survey (lucas) topsoil data` |
| `alias` | `lucas` |
| `tag` | `lucas` — short machine key, used to compose the campaign tag (`lucas_eu`) |
| `data_source_id__data_source_name` | `esdac-jrc` (FK to data source, above) |
| `begun_at` / `ended_at` | `20090101` / `20221231` — spans every survey round shipped so far |
| `species_id__species_name` | `soil` |
| `license_id__license_name` | `jrc-lucas` |
| `territory_id__territory_name` | `EU` |
| `spatial_reference_id__spatial_reference_name` | `geographic` |

`alias` (`lucas`) is the value referenced by `dataset_id__dataset_name` in `campaign.xlsx`. The
alias also forms the campaign name (`lucas european union`) and the `tag` forms the campaign tag
(`lucas_eu`). Neither `alias` nor `tag` may contain `_` or `@`; see
[Auto naming and tags explained][auto_naming_explained].

## Campaign

**Notebook cell**: 2.4 Insert campaigns in `insert_lucas_data.ipynb`

A campaign is a **place**: dataset + territory (+ an optional `site`). The time part is not on
the campaign but on the sampling log: each LUCAS survey round is one sampling log of the same
campaign (`lucas european union 2009`, `lucas european union 2015`). Each row can carry more than
one provision (lab and spectrometer) in a single array field. The campaign must exist before
[Load LUCAS 2009][load_lucas_2009] runs; `lucas_2009_to_xspatula.py` (used in
[Prepare data][prepare_data]) generates the sampling log, geolocation, sample and observation
records under it.

**Path**: `xspatula_lucas/lucas/import_data/dataset_meta/excel/campaign.xlsx`. There is no `name`,
`tag` or `datetime_tag` column: name and tag are composed from the dataset and the territory (see
[Auto naming and tags explained][auto_naming_explained]). The LUCAS 2009 row:

| Column | Value |
|---|---|
| `dataset_id__dataset_name` | `lucas` (FK to dataset, above) |
| `territory_id__territory_name` | `EU` — composes name `lucas european union` (dataset alias + territory name) and tag `lucas_eu` (dataset tag + `_` + territory code) |
| `site` | empty — an optional part of the name and tag, dropped when empty |
| `display_name` | `LUCAS EU` |
| `contact_name` / `contact_email` | `inherit` — pulled from the parent dataset record, see [Utility → inherit and auto][utility_inherit_auto] |
| `begun_at` / `ended_at` | `20090501` / `20091031` |
| `location_method_id__location_method_name` | `gps` |
| `location_error` / `location_error_unit_id__unit_name` | `1000` / `m` |
| `provision_id__provision_name_array` | `foss xds rca,lucas wetlab 2009` — **both** provisions, one array |

The year is given per sampling log instead: the prepare script writes `datetime_tag: "2009"` into
the sampling log record, which becomes `lucas european union 2009` / `lucas_eu_2009`.

The same spreadsheet also holds the other rows this project loads:

| Auto name / tag | `dataset_id__dataset_name` | Territory | `begun_at` / `ended_at` | `provision_id__provision_name_array` | Loaded in |
|---|---|---|---|---|---|
| `lucas european union` / `lucas_eu` | `lucas` | `EU` | `20150501` / `20151031` | `foss xds rca,lucas wetlab 2015` | [LUCAS 2015][lucas_2015] |
| `biogeo16 european union` / `biogeo16_eu` | `biogeo16` | `EU` | `20160331` / `20261231` | `compilation` | BIOGEO16 cells in [Load LUCAS 2009][load_lucas_2009] and [LUCAS 2015][lucas_2015] |

The LUCAS 2015 row composes the same campaign as 2009, `lucas european union`. LUCAS 2015 does
not get a campaign of its own; its prepare script adds the sampling log
`lucas european union 2015` / `lucas_eu_2015` to the existing campaign.

`biogeo16` is not a LUCAS survey round but the EEA biogeographical regions dataset. It has its own
row in `dataset.xlsx` (alias `biogeo16`) and its own publisher in `data_source.xlsx` (`eu-eea`,
the European Environment Agency), so all three levels of the hierarchy above get a second branch.

[insert_dataset_meta]: {{ site.baseurl }}/lucas_2009/insert_dataset_meta/
[utility_explained]: {{ site.baseurl }}/lucas_2009/utility_explained/
[utility_inherit_auto]: {{ site.baseurl }}/lucas_2009/utility_inherit_auto_explained/
[prepare_data]: {{ site.baseurl }}/lucas_2009/prepare_data/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[lucas_2015]: {{ site.baseurl }}/lucas_2015/
[auto_naming_explained]: {{ site.baseurl }}/lucas_2009/auto_naming_explained/
