# Database dump and restore — LUCAS

Added 2026-10-18. Full instructions, including command line alternatives: [xspatula_core_docs/notes/for_lucas_db_dump_restore.md](https://github.com/xspatula/xspatula_core_docs/blob/main/notes/for_lucas_db_dump_restore.md).

## Why

Loading the LUCAS data (`lucas/import_data/insert_lucas_data.ipynb`) takes hours. After a complete load, dump the database once; restoring it later takes minutes and replaces re-running setup + import.

## Scheme file

`setup/zzz/scheme_local_setup.json`, under `postgresdb`:

```json
"dump_path": "~/xspatula/db_dump/xspatula_lucas.dump"
```

Default if `dump_path` is missing: `~/xspatula/db_dump/<db>.dump` (here `~/xspatula/db_dump/lucas.dump`). Relative paths resolve against the project root; keep the dump outside the project.

## Steps

1. Set up the database and import all LUCAS data as usual.
2. Open `setup/dump_restore_db.ipynb`, run the import and scheme cells, then **Dump database**.
3. To get the loaded database back (same or another machine): run **Restore database**. ⚠️ An existing `lucas` database is dropped after confirmation and replaced by the dump. Roles in `db_users` are created/granted afterwards.

Re-dump after every change to the import — a restore is only as current as its dump.

## Limitation

The dump restores the whole database only. LUCAS data can **not** be restored on top of another project's database (e.g. AI4SH): `SERIAL` ids collide. See the core note for details.

## Docs work done (2026-10-18)

Linked the dump/restore option (core docs `setup_db/dump_restore/`) and the ESDAC LUCAS_SOIL licence agreement (no sharing: art. 3 non-transferable/no sub-licence, art. 8 own organisation only, art. 23 confidentiality; art. 9 acknowledgement text) into:
- `_lucas_2009/synopsis.md`: new sections "Dump once, restore in minutes" and "LUCAS license: the data may not be shared" (the anchors other pages link to).
- `_lucas_2015/load_lucas_2015.md`: new section "Dump and restore" with a no-sharing statement, licence link in the download text, and a dump pointer in "After block 4".
- `_lucas_2009/prepare_data.md`: licence link and no-sharing note at the ESDAC request form.
- `_lucas_2009/load_lucas_2009.md`: dump pointer in "After block 3".
- `_machine_learning/explore_select_data.md`: extracted files fall under the licence (art. 5 aggregation/maps).
- `index.md`: "Where to start" dump bullet; "Data License: CC-BY" renamed to "Documentation License" and a LUCAS data licence bullet added.
