# Notes for updating `xspatula_lucas`

Written while building `xspatula_lucas_docs` (2026-09-04). Nothing in `xspatula_lucas` was
touched by this session except `README.md` (rewritten in LUCAS terms, explicitly requested by
Thomas — see git history). Everything else below is a recommendation, not a record of changes
made there.

## Real bug candidate: duplicate/conflicting `lucas_eu_2009` campaign record

Documented in the docs site at `/dataset_meta/manage_campaign/` and
`/lucas_2009/prepare_data/#known-quirk-duplicate-campaign-sampling-log`, but worth flagging here
directly since it's a genuine pipeline inconsistency, not just a docs gap:

- `lucas/import_data/dataset_meta/excel/campaign.xlsx`, loaded via
  `insert_lucas_dataset_meta.ipynb`, defines one `lucas_eu_2009` campaign row with
  `provision_id__provision_name_array` = `"foss xds rca,lucas-wetlab-2009"` (**both** provisions).
- `lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`'s `step1_campaign_and_sampling_log()`
  independently generates a `manage_campaign` call for the **same name** `lucas_eu_2009`, but with
  `provision_id__provision_name_array` set to only `lucas-wetlab-2009` (the copy under
  `process_lab/campaign/`, which `load_LUCAS.ipynb` actually runs — the `process_spectra/campaign/`
  copy is generated but never loaded by any notebook, so it's dead output either way).

If `manage_campaign` is UPDATE-capable (matches by name, replaces fields), running
`insert_lucas_dataset_meta.ipynb` and then `load_LUCAS.ipynb`'s campaign cell against the same
database could overwrite the campaign's provision array down to one value, silently dropping
`foss xds rca`. **This was not tested end-to-end** — I read the script and the Excel source, I
didn't run the pipeline against a live database. Worth either:

1. Testing directly (run both, inspect `observation.campaign.provision_id__provision_name_array`
   afterward), or
2. Fixing at the source — either drop `step1_campaign_and_sampling_log()`'s campaign generation
   from the script (rely on the Excel route as the single source of truth for campaigns) and only
   keep the sampling-log half, or make the script write the full provision array to match.

Also worth deciding: the `process_spectra/campaign/` and `process_spectra/sampling_log/` copies
the script generates are pure dead output today (nothing loads them). Either wire them into a
spectra-only load path, or stop generating them.

## Deep AI4SH branding beyond the README

The README rewrite (this session) only touched prose and doc links. These are still literally
named after AI4SH, unchanged:

- `src/ai4sh/` — the Python package `load_LUCAS.ipynb` and friends import (`from src.ai4sh import
  Run_process`)
- `src/postgres/pg_ai4sh.py`
- `anaconda/xspatula_ai4sh_py_3.12.yml` and the `xspatula_ai4sh_py_3.12` conda env/kernel name
- `setup/zzz/scheme_ai4sh_local_setup.json`, `_delete.json`, `_use.json`, `_use_pswd.json`
- `setup/zzz/lucas/setup_db/json_ai4sh/` and `setup/zzz/lucas/setup_processes/json_ai4sh/`
  directory names

None of this is a docs problem — it's a real rename/refactor (Python imports, a conda env name,
JSON file references throughout `setup/zzz/`) that touches working code, not just text. Flagged
for a separate task if Thomas wants LUCAS fully de-branded; the current README calls out the
scheme/anaconda naming inline so newcomers don't read it as a mistake.

## Notebook markdown cells reference the wrong docs URL

`load_LUCAS.ipynb`, `insert_lucas_dataset_meta.ipynb`, and `insert_utility.ipynb` all link to
`https://xspatula.github.io/setup_core_db_docs/framework/scheme_file/` in their "Scheme file"
markdown cell. That's the old pre-`xspatula_core_docs` URL (see the main `xspatula_lucas_docs`
CLAUDE.md for why `xspatula_core_docs` supersedes `setup_core_db_docs`). Worth updating to
`https://xspatula.github.io/xspatula_core_docs/framework/scheme_file/` next time these notebooks
are touched.
