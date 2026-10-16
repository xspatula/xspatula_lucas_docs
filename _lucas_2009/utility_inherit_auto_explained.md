---
title: "Utility: Inherit and Auto Explained"
layout: single
sidebar:
  nav: "lucas_2009"
permalink: /lucas_2009/utility_inherit_auto_explained/
author_profile: false
---

Reference page — not required reading to complete any of the LUCAS 2009 steps. Explains two
special values you can put in an Excel/CSV cell, `inherit` and `auto`, that a few of this
project's spreadsheets already use.

## Inherit

**Generic mechanism, documented once for the whole Xspatula framework**:
[xspatula_core_docs → Setup processes → Process options → Inherit][core_inherit]. Read that page
for how it works — this section only covers where LUCAS actually uses it.

In short: set a parameter's value to the literal string `inherit` and, instead of requiring you
to type it, the framework looks it up from a related record already on the same row (via a
foreign key) — the exact source table/column is registered once in the database, not per-row in
the spreadsheet.

### Where LUCAS uses it

`lucas/import_data/dataset_meta/excel/campaign.xlsx` sets `contact_name` and `contact_email` to
`inherit` for both LUCAS campaigns:

| Column | Value |
|---|---|
| `contact_name` | `inherit` |
| `contact_email` | `inherit` |

Since every campaign row already carries `dataset_id__dataset_name`, the framework resolves both
fields from the parent [dataset record][dataset_meta_explained] instead of repeating ESDAC-JRC's
contact details on every campaign row. Edit them on the dataset once; every campaign that inherits
picks up the change automatically next time it's (re-)inserted.

There are also other processes that use the `inherit` mechanism, but these are not explicitly obvious. All process arguments have a `default` value, and if a user entered value for this argument is not required and the `default` value states `inherit`, the mechanism is automatically called. To get a detailed grasp on this, see the core documentation on [inherit][core_inherit].

## Auto / auto_name

Also documented generically at
[xspatula_core_docs → Setup processes → Process options][core_auto_name]. Set a parameter to the literal string `auto` or
`auto_name` and the framework builds it by concatenating other parameter values already on the same row, per a `%`-style format string registered once in the database for that process/parameter — rather than you typing the concatenated string out by hand.

### Where LUCAS uses it

The campaign, sampling log and observation log records are auto named. None of them gives a
`name` or `tag`; the framework composes both from the parent records:

- Campaign (the place) — `lucas european union` / `lucas_eu` (dataset alias or tag + territory
  name or code)
- Sampling log (the time) — `lucas european union 2009` / `lucas_eu_2009` (campaign name or tag +
  `datetime_tag`, written by the prepare script)
- Observation log — `lucas european union 2009@lucas wetlab 2009` / `lucas_eu_2009@lucas-lab-2009`
  (sampling log name or tag `@` provision name or tag), see [Load LUCAS 2009][load_lucas_2009]

Samples were already built by an `auto_name` rule: `<POINT_ID>@0-20` (`'%s@%s-%s'
%(tag,profile_min,profile_max)`, point ID `@` depth profile), see
[Samples explained][samples_explained].

The utility catalogues (including `foreign_key.xlsx` and `unit_translate.xlsx`, see
[Foreign key explained][foreign_key_explained]) don't use `auto`.

The full LUCAS chain, the Excel `tag` columns and the rules for `_` and `@` are on
[Auto naming and tags explained][auto_naming_explained].

[core_inherit]: https://xspatula.github.io/xspatula_core_docs/setup_processes/process_options/#inherit
[core_auto_name]: https://xspatula.github.io/xspatula_core_docs/setup_processes/process_options/#automatic-naming
[dataset_meta_explained]: {{ site.baseurl }}/lucas_2009/dataset_meta_explained/
[load_lucas_2009]: {{ site.baseurl }}/lucas_2009/load_lucas_2009/
[samples_explained]: {{ site.baseurl }}/lucas_2009/samples_explained/
[foreign_key_explained]: {{ site.baseurl }}/lucas_2009/foreign_key_explained/
[auto_naming_explained]: {{ site.baseurl }}/lucas_2009/auto_naming_explained/
