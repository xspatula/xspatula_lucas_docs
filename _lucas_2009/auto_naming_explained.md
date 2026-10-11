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

- `name` is free text for people, e.g. `lucas 2009 eu`.
- `tag` is a short key for machines, e.g. `lucas_2009_eu`. Tags are used in composed names and
  later in ML file names.

Full description: [Name and tag][core_name_and_tag].

## Excel columns

Three Excel files gained a column for this ([Tag columns][core_tag_columns]):

| File | New column | LUCAS values |
|---|---|---|
| `lucas/import_data/dataset_meta/excel/dataset.xlsx` | `tag` | `lucas`, `biogeo16` |
| `lucas/import_data/dataset_meta/excel/campaign.xlsx` | `datetime_tag` | `2009`, `2015`, `2016` (biogeo16) |
| `lucas/import_data/utility/observation/excel/provision.xlsx` | `tag` | see table below |

`campaign.xlsx` has no `name` or `tag` columns: both are auto named from the dataset alias/tag
and `datetime_tag`. The provision tags:

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
| campaign | `lucas 2009` | `lucas_2009` |
| sampling_log | `lucas 2009 eu` | `lucas_2009_eu` |
| observation_log (lab) | `lucas 2009 eu@lucas wetlab 2009` | `lucas_2009_eu@lucas-lab-2009` |
| observation_log (spectra) | `lucas 2009 eu@foss xds rapid content analyzer` | `lucas_2009_eu@foss-xds-rca` |
| observation_log (landscape) | `lucas 2009 eu@human interpretation` | `lucas_2009_eu@hum-sens` |

LUCAS 2015 follows the same pattern with `2015` (`lucas 2015`, `lucas 2015 eu`,
`lucas 2015 eu@lucas wetlab 2015`, …; tags `lucas_2015`, `lucas_2015_eu`,
`lucas_2015_eu@lucas-lab-2015`, …).

The BIOGEO16 regions have their own dataset and campaign:

| Level | name | tag |
|---|---|---|
| dataset | `eu-eea biogeographical regions 2016` (alias `biogeo16`) | `biogeo16` |
| campaign | `biogeo16 2016` | `biogeo16_2016` |
| sampling_log | `biogeo16 2016 eu` | `biogeo16_2016_eu` |
| observation_log | `biogeo16 2016 eu@compilation` | `biogeo16_2016_eu@compilat` |

Names join parts with a blank, tags with `_`. Both use `@` between sampling log and provision.

## How each level is built

Each auto-named parameter has a `concat` rule in the process definition
(`setup/zzz/lucas/setup_processes/json/observation/*_v10_sql.json`):

| Level | `name` | `tag` |
|---|---|---|
| campaign | `'%s %s' %(dataset_id__dataset_alias, datetime_tag)` | `'%s_%s' %(dataset_id__dataset_tag, datetime_tag)` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, campaign_territory_id)` | `'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` | `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` |

`campaign_territory_id` is the campaign's territory code (`eu`), read from
`observation.campaign_location`. Arguments like `dataset_id__dataset_alias` read a column from the
parent record; plain arguments like `datetime_tag` take the value from the same row
([Concat arguments][core_concat_arguments]).

The `name` and `tag` parameters of `manage_sampling_log` (`sampling_log_v10_sql.json`):

```json
{
  "parameter": "name",
  "parameter_type": "text",
  "required": false,
  "default_value": "auto",
  "hint": "The name of the sampling log",
  "schema_table": { "schema": "observation", "table": "sampling_log", "write": true },
  "permission": { "update": true, "delete": false },
  "auto_name": {
    "concat": "'%s %s' %(campaign_id__campaign_name, campaign_territory_id)"
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
    "concat": "'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)"
  },
  "format": { "at": false, "capital": false, "blank": false }
}
```

Both default to `auto`, so a record that leaves them out gets them composed.

## A record without a name

`lucas_2009_to_xspatula.py` writes the sampling log record with no `name` or `tag`
(`.../process_lab/sampling_log/manage_process/lucas 2009_sampling_log.json`):

```json
{
  "process": [
    {
      "process": "manage_sampling_log",
      "parameters": {
        "campaign_id__campaign_name": "lucas 2009",
        "contact_name": "inherit",
        "contact_email": "inherit",
        "abstract": "Sampling log for lucas eu 2009"
      }
    }
  ]
}
```

The framework looks up campaign `lucas 2009` (tag `lucas_2009`) and its territory `eu`, and stores
the sampling log as `lucas 2009 eu` / `lucas_2009_eu`. The observation log records work the same
way: they give `sampling_log_id__sampling_log_name` and `provision_id__provision_name` and leave
`name` out. The prepare script still builds the same names as constants (`CAMPAIGN_NAME`,
`SAMPLING_LOG_NAME`, `*_OBSERVATION_LOG_NAME`) because the sample and observation records refer to
them; see [Prepare data][prepare_data].

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
