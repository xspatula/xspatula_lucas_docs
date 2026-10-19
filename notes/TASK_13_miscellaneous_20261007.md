# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. The previous CLAUDE.md files are saved as `.claude/TASK_1_20260904.md` through `.claude/TASK_11_LUCAS_2015.md`. Please look at these to understand where we are and for reference to the code repo for which this is the documentation.

## Reducing setup_db content (resolved 2026-10-07)

The existing documentation included database schemas and tables that were outside what was required by the LUCAS datasets (the dataset for which this project is set up). I removed all schemas and tables not required by LUCAS and edited the pilot file setting up the database in the sibling code repo (`xspatula_lucas`) pilot file `setup/zzz/lucas/setup_db/db_setup.txt`.

Please check that the documentation in this repo is not containing any reference to any of the deleted tables.

## Reducing setup_processes content (resolved 2026-10-07)

The same issue as in "## Reducing setup_db content" above, but for the pilot file `setup/zzz/lucas/setup_processes/setup_processes.txt`.

Please check that the documentation in this repo is not containing any reference to any of the deleted tables.

## Excel files for loading utility data (resolved 2026-10-07)

The excel files for loading the utility data also changed (were reduced) to only accomodate the utility catalogues relevant for LUCAS data. These excel files are under `lucas/import_data/utility`. Please also check that records removed from these files are not mentioned in the documentation.

## Outcome of the reduction check (2026-10-07)

Docs (commit `a827bb4`): only two pages referenced removed content.
- `_lucas_2009/utility_explained.md` — dropped `setting_system`/`juxtaposition` and stale TODO placeholders (monolith, sound, taxa, reference proprietor); added `nominal_classes`.
- `_lucas_2009/samples_explained.md` — removed `juxtaposition_id__juxtaposition_name` (now commented out in `lucas_2009_to_xspatula.py`).

Code repo `xspatula_lucas` (commit `e7cac53`): removed unused providers `soil cylinder` and `bruker`, stripped macrofauna/monolith/taxa/eDNA TODO blocks from `insert_observation_utility.txt`, untracked LibreOffice lock files (`.~lock.*#` now in `.gitignore`).

Decision: `setting_system` and `juxtaposition` tables are kept in `db_setup.txt` (empty for LUCAS) — they are wired into the dataset → campaign → sampling_log → sample inherit chain and the core framework.
