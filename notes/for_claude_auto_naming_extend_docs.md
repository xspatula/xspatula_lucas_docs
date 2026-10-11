# For Claude: document tag-based automatic naming (xspatula_lucas_docs)

Written 2026-10-11 from a session in `xspatula_lucas`. Work through the tasks below from within this
repo. This repo gets **one new summary page with LUCAS examples** that links to the full
description in `xspatula_core_docs`, plus corrections to pages that still show the old LUCAS names.

## Background

The automatic naming (`auto_name`) was extended in `xspatula_lucas`:

- branch `feature/tag-auto-naming`, PR https://github.com/xspatula/xspatula_lucas/pull/1
- all LUCAS processes run clean on the live DB

The full description is being written in `xspatula_core_docs` as the page **"Automatic naming and
tags"**:

- URL: `https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/`
- anchors: `#name-and-tag`, `#tag-columns`, `#reserved-separators`, `#concat-arguments`,
  `#record-lookup`, `#worked-example`, `#errors`, `#requirements`
- Link to these rather than repeating the full rules.
- If the core page isn't published yet, link anyway; the URL is fixed.

**Sources of truth.** Verify every example against these files; the code wins over this note:

- `../xspatula_lucas/setup/zzz/lucas/setup_processes/json/observation/campaign_v10_sql.json`,
  `sampling_log_v10_sql.json` and `observation_log_v10_sql.json` (the `auto_name` concats)
- `../xspatula_lucas/lucas/prepare_lucas_data/lucas_2009_to_xspatula.py` and
  `lucas_2015_to_xspatula.py` (constants `CAMPAIGN_NAME`, `SAMPLING_LOG_NAME`,
  `*_OBSERVATION_LOG_NAME`; explicit `name` entries are now commented out so `auto` applies)
- `../xspatula_lucas/lucas/import_data/dataset_meta/excel/dataset.xlsx` and `campaign.xlsx`, and
  `../xspatula_lucas/lucas/import_data/utility/observation/excel/provision.xlsx` (the new `tag` and
  `datetime_tag` columns). The staging JSON next to them shows the same content in readable form.
- `../xspatula_lucas/src/lib/structure.py` (`compose_auto_name` and helpers, `separator_exempt_D`)

## What changed for LUCAS (summary for the new page)

**Name versus tag.** `name` is free text for people; `tag` is a short key for machines, used in
composed names and later in ML file names.

**Excel columns:**
- `dataset.xlsx` gains `tag` (LUCAS: `lucas`).
- `campaign.xlsx` gains `datetime_tag` (`2009`, `2015`). The campaign name and tag are auto named,
  so leave them out.
- `provision.xlsx` gains `tag`.

**The auto-named chain for LUCAS 2009** (verify the provision values):

| Level | name | tag |
|---|---|---|
| dataset | `land use and coverage area frame survey (lucas) topsoil data` (alias `lucas`) | `lucas` |
| campaign | `lucas 2009` | `lucas_2009` |
| sampling_log | `lucas 2009 eu` | `lucas_2009_eu` |
| observation_log | `lucas 2009 eu@<provision name>` | `lucas_2009_eu@<provision tag>` |

For LUCAS 2015 the same pattern applies with `2015`. Check the biogeo16 chain as well
(`biogeo16 2016`, `biogeo16 2016 eu`, from `BIOGEO_*` in the prepare script).

**How each level is built** (show the real `auto_name` block for at least sampling_log):
- campaign: `'%s %s' %(dataset_id__dataset_alias, datetime_tag)` / `'%s_%s' %(dataset_id__dataset_tag, datetime_tag)`
- sampling_log: `'%s %s' %(campaign_id__campaign_name, campaign_territory_id)` /
  `'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)`. `campaign_territory_id` is the
  campaign's territory code (`eu`) from `observation.campaign_location`.
- observation_log: `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` /
  `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)`

**User-facing rules to summarise** (one or two sentences each, linking to the core anchors):
- To get an automatic name, leave the cell or parameter out, or write `auto`. ( `#concat-arguments` )
- When referring to a parent (e.g. `campaign_id__campaign_name`), you may give its name, tag or
  alias; the framework finds it either way. ( `#record-lookup` )
- `_` and `@` aren't allowed in hand-written names, aliases or tags, because they're reserved for
  composed names. A composed name you type yourself is accepted only if it exactly matches the
  auto name. ( `#reserved-separators` )
- Geolocation names (e.g. `fr_lucas@38622598`) are the one exception.
- Load order matters: dataset → campaign → sampling_log → observation_log. ( `#requirements` )

## Task 1: new page

- Create `_lucas_2009/auto_naming_explained.md`, using the front matter pattern of
  `utility_inherit_auto_explained.md`:
  - `title: "Auto naming and tags explained"`
  - `permalink: /lucas_2009/auto_naming_explained/`
  - `sidebar: nav: "lucas_2009"`
- Add it to `_data/navigation.yml` under `lucas_2009` → `Reference`, directly after "Utility:
  inherit and auto". If `_lucas_2015` has its own reference section, link the same page from there
  rather than duplicating it.
- Content: a short intro, the name/tag idea, the Excel columns, the LUCAS chain table, the
  sampling_log `auto_name` JSON block as an example, a record JSON example showing `name` omitted
  (taken from the generated `manage_process` sampling_log JSON or the prepare script), the
  user-facing rules above, and "Full description" links to the core page anchors.

## Task 2: fix `utility_inherit_auto_explained.md`

The section `### Where LUCAS uses it` under `## Auto / auto_name` is now wrong. It says LUCAS
builds its composite names in Python and that no Excel file uses `auto`.

- Rewrite it: campaign, sampling_log and observation_log are now auto named, and samples
  (`<POINT_ID>@0-20`) were already built by an `auto_name` rule.
- Link to the new page.
- The examples `lucas_eu_2009@lucas-wetlab-2009` are outdated.
- Also add the new page to the reference links at the bottom.

## Task 3: replace old names across the site

The old names `lucas_eu_2009` and `lucas_eu_2015` (campaign, sampling_log, and the prefix of
observation_log names) no longer exist.

- Run `grep -rn 'lucas_eu_20' --include='*.md' . | grep -v _site`. Known hits:
  - `_lucas_2009/observations_explained.md`
  - `_lucas_2009/samples_explained.md`
  - `_lucas_2009/load_lucas_2009.md`
  - `_lucas_2009/dataset_meta_explained.md`
  - `_lucas_2009/prepare_data.md`
  - `_machine_learning/explore_select_data.md`
  - pages under `_lucas_2015/`
- Replace each with the new name or tag as appropriate: names in prose, tags where a key or
  identifier is meant.
- `notes/xspatula_lucas.md` is a working note; leave it.
- In `dataset_meta_explained.md`, also document the new `tag` (dataset) and `datetime_tag`
  (campaign) columns where the Excel columns are described.
- In `prepare_data.md`, update any description of the prepare script's naming constants.

## Done when

- The new page renders locally (`notes/local_serve_cmd.txt`) and sits after "Utility: inherit and
  auto" in the sidebar.
- `grep -rn 'lucas_eu_20' --include='*.md' . | grep -v -e _site -e notes/` returns nothing.
- Every rule on the new page links to the matching core anchor.
