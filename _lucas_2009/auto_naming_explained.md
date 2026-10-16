---
title: "Auto naming and tags explained"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/auto_naming_explained/
author_profile: false
---

Reference page with LUCAS examples. The campaign, sampling log and observation log records are
not named by hand: the framework composes their `name` and `tag` from the parent records. This
page shows what that looks like for LUCAS. The full rules are on the core page
[Automatic naming and tags][core_auto_naming].

## Name and tag

Every record in the dataset → campaign → sampling log → observation log chain carries both a
`name` and a `tag`:

- `name` is free text for people, e.g. `lucas european union 2009`.
- `tag` is a short key for machines, e.g. `lucas_eu_2009`. Tags are used in composed names and
  later in ML file names.

Full description: [Name and tag][core_name_and_tag].

## Excel columns

Two Excel files carry a `tag` column ([Tag columns][core_tag_columns]):

| File | Column | LUCAS values |
|---|---|---|
| `lucas/import_data/dataset_meta/excel/dataset.xlsx` | `tag` | `lucas`, `biogeo16` |
| `lucas/import_data/utility/observation/excel/provision.xlsx` | `tag` | see table below |

`campaign.xlsx` has no `name`, `tag` or `datetime_tag` columns. A campaign is a **place**: its
name and tag are composed from the dataset alias/tag and the territory (plus an optional `site`).
The **time** part sits on the sampling log: `lucas_2009_to_xspatula.py` writes `datetime_tag`
(`2009`, `2016` for biogeo16) into each sampling log record, and the 2015 script writes `2015`.
The provision tags:

| Provision name | Tag |
|---|---|
| `lucas wetlab 2009` | `lucas-lab-2009` |
| `lucas wetlab 2015` | `lucas-lab-2015` |
| `foss xds rapid content analyzer` | `foss-xds-rca` |
| `human interpretation` | `hum-sens` |
| `compilation` | `compilat` |

## The LUCAS chain

LUCAS 2009:

| Level | name | tag |
|---|---|---|
| dataset | `land use and coverage area frame survey (lucas) topsoil data` (alias `lucas`) | `lucas` |
| campaign | `lucas european union` | `lucas_eu` |
| sampling_log | `lucas european union 2009` | `lucas_eu_2009` |
| observation_log (lab) | `lucas european union 2009@lucas wetlab 2009` | `lucas_eu_2009@lucas-lab-2009` |
| observation_log (spectra) | `lucas european union 2009@foss xds rapid content analyzer` | `lucas_eu_2009@foss-xds-rca` |
| observation_log (landscape) | `lucas european union 2009@human interpretation` | `lucas_eu_2009@hum-sens` |

LUCAS 2015 is the same place sampled again: it reuses campaign `lucas european union` /
`lucas_eu` and adds sampling log `lucas european union 2015` / `lucas_eu_2015`, with observation
logs `lucas european union 2015@lucas wetlab 2015` / `lucas_eu_2015@lucas-lab-2015`, and so on.

The BIOGEO16 regions have their own dataset and campaign:

| Level | name | tag |
|---|---|---|
| dataset | `eu-eea biogeographical regions 2016` (alias `biogeo16`) | `biogeo16` |
| campaign | `biogeo16 european union` | `biogeo16_eu` |
| sampling_log | `biogeo16 european union 2016` | `biogeo16_eu_2016` |
| observation_log | `biogeo16 european union 2016@compilation` | `biogeo16_eu_2016@compilat` |

Names join parts with a blank, tags with `_`. Both use `@` between sampling log and provision.

## How each level is built

Each auto-named parameter has a `concat` rule in the process definition
(`setup/zzz/lucas/setup_processes/json/observation/*_v10_sql.json`):

| Level | `name` | `tag` |
|---|---|---|
| campaign | `'%s %s %s' %(dataset_id__dataset_alias, territory_id__territory_name, site?)` | `'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, datetime_tag)` | `'%s_%s' %(campaign_id__campaign_tag, datetime_tag)` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` | `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` |

`territory_id__territory_name` (`european union`) and `territory_id__territory_iso_code_a2_ext`
(`eu`) are read from the campaign's territory (`EU` in `campaign.xlsx`). `site?` is optional: a
campaign without a `site` drops that part, so the LUCAS campaign is `lucas european union` /
`lucas_eu`. Arguments like `dataset_id__dataset_alias` read a column from the parent record;
plain arguments like `datetime_tag` take the value from the same record
([Concat arguments][core_concat_arguments]).

The `name` and `tag` parameters of `manage_sampling_log` (`sampling_log_v10_sql.json`):

```json
{
  "parameter": "name",
  "parameter_type": "text",
  "required": false,
  "default_value": "auto",
  "hint": "Name of the sampling log, auto = campaign name + datetime tag ('no date' if no datetime tag)",
  "schema_table": { "schema": "observation", "table": "sampling_log", "write": true },
  "permission": { "update": true, "delete": false },
  "auto_name": {
    "concat": "'%s %s' %(campaign_id__campaign_name, datetime_tag)"
  }
},
{
  "parameter": "tag",
  "parameter_type": "text",
  "required": false,
  "default_value": "auto",
  "schema_table": { "schema": "observation", "table": "sampling_log", "write": true },
  "permission": { "update": false, "delete": false },
  "auto_name": {
    "concat": "'%s_%s' %(campaign_id__campaign_tag, datetime_tag)"
  },
  "format": { "at": false, "capital": false, "blank": false }
}
```

Both default to `auto`, so a record that leaves them out gets them composed.

## A record without a name

`lucas_2009_to_xspatula.py` writes the sampling log record with no `name` or `tag`
(`.../process_lab/sampling_log/manage_process/lucas_eu_2009_sampling_log.json`):

```json
{
  "process": [
    {
      "process": "manage_sampling_log",
      "parameters": {
        "campaign_id__campaign_name": "lucas_eu",
        "datetime_tag": "2009",
        "begun_at": 20090501,
        "ended_at": 20091031,
        "url": "https://esdac.jrc.ec.europa.eu/content/lucas-2009-topsoil-data",
        "contact_name": "inherit",
        "contact_email": "inherit",
        "abstract": "Sampling log for lucas eu 2009"
      }
    }
  ]
}
```

The parent is given by its tag (`lucas_eu`); the framework finds campaign
`lucas european union` and stores the sampling log as `lucas european union 2009` /
`lucas_eu_2009`. The observation log records work the same way: they give
`sampling_log_id__sampling_log_name` as the tag (`lucas_eu_2009`) plus
`provision_id__provision_name`, and leave `name` out. The prepare script still holds the names and
tags as constants (`CAMPAIGN_NAME`/`CAMPAIGN_TAG`, `SAMPLING_LOG_NAME`/`SAMPLING_LOG_TAG`,
`*_OBSERVATION_LOG_NAME`) because the sample and observation records refer to them; see
[Prepare data][prepare_data].

## Rules

- **Asking for an auto name.** Leave the cell or parameter out, or write `auto`.
  [Concat arguments][core_concat_arguments]
- **Referring to a parent.** In a parameter like `campaign_id__campaign_name` you may give the
  parent's name, tag or alias; the framework finds it either way. That is why `campaign.xlsx` can
  give `dataset_id__dataset_name` as `lucas` (the alias) instead of the full dataset name.
  [Record lookup][core_record_lookup]
- **Reserved separators.** `_` and `@` aren't allowed in hand-written names, aliases or tags,
  because they're reserved for composed names. A composed name you type yourself is accepted only
  if it exactly matches the auto name. This is why the sample name `<POINT_ID>@0-20` written by the
  prepare script passes: it equals what the sample's `auto_name` rule composes. Use `-` instead
  (e.g. provision tag `foss-xds-rca`). [Reserved separators][core_reserved_separators]
- **Geolocation names are the exception.** Names like `fr_lucas@38622598` are set before the
  sample and act as a unique key, so `manage_geolocation` may use `_` and `@`.
  [Reserved separators][core_reserved_separators]
- **Load order matters.** Load dataset → campaign → sampling_log → observation_log. Each level
  reads its parent's name and tag from the database, so the parent must exist first.
  [Requirements][core_requirements]

If composition fails (parent missing, ambiguous lookup, no territory), the process logs an
`ERROR auto_name` line and stops; see [Errors][core_errors].

## Full description

[xspatula_core_docs → Automatic naming and tags][core_auto_naming]:

- [Name and tag][core_name_and_tag]
- [Tag columns][core_tag_columns]
- [Reserved separators][core_reserved_separators]
- [Concat arguments][core_concat_arguments]
- [Record lookup][core_record_lookup]
- [Worked example][core_worked_example]
- [Errors][core_errors]
- [Requirements][core_requirements]

Related LUCAS pages: [Dataset metadata explained][dataset_meta_explained],
[Utility: inherit and auto][utility_inherit_auto_explained],
[Observations explained][observations_explained].

[core_auto_naming]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/
[core_name_and_tag]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#name-and-tag
[core_tag_columns]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#tag-columns
[core_reserved_separators]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#reserved-separators
[core_concat_arguments]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#concat-arguments
[core_record_lookup]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#record-lookup
[core_worked_example]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#worked-example
[core_errors]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#errors
[core_requirements]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#requirements
[prepare_data]: {{ site.baseurl }}/lucas_2009/prepare_data/
[dataset_meta_explained]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/
[utility_inherit_auto_explained]: {{ site.baseurl }}/lucas_2009/utility_inherit_auto_explained/
[observations_explained]: {{ site.baseurl }}/lucas_2009/observations_explained/
