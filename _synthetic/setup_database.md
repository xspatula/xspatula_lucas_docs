---
title: "Set up the database"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/setup_database/
author_profile: false
excerpt: "Create the database synthetic, its roles and the first user, and register the processes. All credentials come from ~/.netrc."
date: 2026-11-11
last_modified_at: 2026-11-11
---

You create the database `synthetic`, its database roles and a first application user, then register the processes the rest of the tutorial uses. No password is written in any file except `~/.netrc`.

## 1. The setup scheme

**File**: `xspatula_lucas/setup/zzz/scheme_local_setup_synthetic.json`

```json
{
  "project_path": "./lucas",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "synthetic",
    "host_netrc_id": "my_pg_superuser",
    "dump_path": "~/xspatula/db_dump/xspatula_synthetic.dump",
    "db_users": [
      { "user_netrc_id": "community_admin",  "role": "community_admin" },
      { "user_netrc_id": "login_evaluation", "role": "login_evaluation" },
      { "user_netrc_id": "user_cat_1",       "role": "user_cat_1" },
      { "user_netrc_id": "user_cat_2",       "role": "user_cat_2" },
      { "user_netrc_id": "user_cat_3",       "role": "user_cat_3" },
      { "user_netrc_id": "user_cat_4",       "role": "user_cat_4" },
      { "user_netrc_id": "user_cat_5",       "role": "user_cat_5" }
    ]
  },
  "initial_user": {
    "user_netrc_id": "ini_user_synthetic",
    "email": "initial_user@example.com",
    "email_alt": "initial_user_alt@example.com",
    "first_name": "Initial",
    "last_name": "User",
    "organisation_id__organisation_name": "xspectre",
    "territory_id__territory_name": "sweden",
    "stratum_code": "5"
  },
  "process": [
    { "execute": true, "verbose": 1, "overwrite": false, "delete": false }
  ]
}
```

| Key | Meaning |
|---|---|
| `project_path` | `./lucas`, relative to the scheme file: the folder `setup/zzz/lucas` that holds the database definitions. Keep it |
| `postgresdb.db` | Name of the new database: `synthetic` |
| `postgresdb.host_netrc_id` | `.netrc` machine name of **your** PostgreSQL superuser. Change it to your own |
| `postgresdb.dump_path` | Where `dump_restore_db.ipynb` writes a dump of the database |
| `postgresdb.db_users` | The database roles and their `.netrc` machine names. Keep the role names |
| `initial_user` | The first application user, stratum 5. Login and password come from `.netrc` (`user_netrc_id`). `organisation_id__organisation_name` and `territory_id__territory_name` must exist in the seed tables (`xspectre` and `sweden` do) |

## 2. The .netrc entries

Add one line per credential to `~/.netrc` in your home directory. **Every line starts with `machine`** and has all three keywords:

```
machine my_pg_superuser    login your_pg_superuser password <long-random-1>
machine ini_user_synthetic login ini_user          password <long-random-2>
machine community_admin    login community_admin   password <long-random-3>
machine login_evaluation   login login_evaluation  password <long-random-4>
machine user_cat_1         login user_cat_1        password <long-random-5>
machine user_cat_2         login user_cat_2        password <long-random-6>
machine user_cat_3         login user_cat_3        password <long-random-7>
machine user_cat_4         login user_cat_4        password <long-random-8>
machine user_cat_5         login user_cat_5        password <long-random-9>
```

- Generate each password with `python3 -c "import secrets; print(secrets.token_urlsafe(18))"`.
- Restrict the file to yourself: `chmod 600 ~/.netrc`.
- **Database roles belong to the PostgreSQL server, not to one database.** If you already set up another Xspatula database (LUCAS, for example) on the same server, the role lines exist already: keep them as they are. Setup gives the existing roles their rights in the new database.
- Never write a password in a scheme file. The repositories ship a pre-commit hook that refuses JSON files with plaintext passwords.

The rules, pitfalls and the full list of netrc ids are on [.netrc and credentials][core_netrc].

## 3. Run setup_db.ipynb

Open `xspatula_lucas/setup/setup_db.ipynb`.

1. In the scheme cell, set
   ```python
   scheme_file = './zzz/scheme_local_setup_synthetic.json'
   ```
2. Run the setup cell. Answer `y` when asked whether to set up the database `synthetic`.
3. Run the audit cell (optional). It turns on the audit trail of inserts, updates and deletes.

**One checkout works with one database at a time.** Setup rewrites the environment files in `src/postgres/environment/` so that they point at `synthetic`. After this, the LUCAS notebooks in the same checkout also talk to `synthetic`. Switch with:

```bash
python synthetic_dataset/switch_db.py lucas       # back to LUCAS
python synthetic_dataset/switch_db.py synthetic   # to the synthetic database
python synthetic_dataset/switch_db.py             # show the current one
```

### Rebuilding from scratch

Run `setup/delete_db.ipynb` with

```python
scheme_file = './zzz/scheme_local_delete_synthetic.json'
```

It drops the database `synthetic` and deletes the environment files. Roles that other databases use are kept. Also delete `synthetic_dataset/project_synthetic/data/` and `…/models/` before you start over.

## 4. Adding more initial users (optional, not recommended)

`setup/zzz/lucas/setup_db/json/community/user_records_v10_sql.json` seeds the table `community.user`. Its only row has the password placeholder `"{initial_user}"`: setup fills that row from the scheme's `initial_user` (login from `.netrc`, password bcrypt-hashed in memory, the other keys override the row).

You can add more users as further rows in `values`, with the same columns. The `password` must then be a **bcrypt hash**, never plaintext. Make one from the root of `xspatula_lucas`:

```python
from src.community import Hash_password
print(Hash_password('the-new-users-password'))
```

Seeds are insert-only: existing rows are skipped, so a user added after setup gets in by running `setup_db.ipynb` again.

**Use step 7 instead.** This seed file is shared by every database set up from `setup/zzz/lucas`, the LUCAS database included, so a user added here appears in all of them. For users of one database, use [Register an extra user](#7-register-an-extra-user-optional).

## 5. The use scheme

**File**: `xspatula_lucas/setup/zzz/scheme_local_use_synthetic.json`

```json
{
  "project_path": "./lucas",
  "user_project": {
    "user_netrc_id": "ini_user_synthetic"
  },
  "process": [
    { "execute": true, "verbose": 1, "overwrite": false, "delete": false }
  ]
}
```

`user_netrc_id` is the `.netrc` machine of the initial user: the same entry that created the user at setup now logs it in.

## 6. Run setup_processes.ipynb

Open `xspatula_lucas/setup/setup_processes.ipynb`, set

```python
scheme_file = './zzz/scheme_local_use_synthetic.json'
```

and run it. It registers about 120 processes.

If it stops with *permission denied* (the login role has no rights on the schema `community`), rerun the setup cell of `setup_db.ipynb`. It re-grants the role rights on every run, and the error message tells you so.

## 7. Register an extra user (optional)

**Notebook**: `synthetic_dataset/user_management/register_users.ipynb`

The project schemes log in as `ini_user_synthetic`, so this step is **not required**. It shows how to add an organisation (`synthetic soil lab`) and a user (`dummy.user`, stratum 5) without sending an e-mail:

| Cell | Does |
|---|---|
| 1 | General utilities (territories), needed by organisation and user |
| 2 | Inserts the organisation |
| 3 | Translates the user excel sheet to JSON |
| 4 | Writes a **bcrypt hash** of `dummy_password` into `user/manage_process/manage_user.json`; the plaintext is never stored. **Change `dummy_password` before you run it** |
| 5 | Inserts the user |

To work as that user, add a line to `.netrc`

```
machine dummy_synthetic login dummy.user password <the password you set in cell 4>
```

and set `"user_netrc_id": "dummy_synthetic"` in `synthetic_dataset/scheme_synthetic.json`.

## Next step

[Import the data][import].

[core_netrc]: https://xspatula.github.io/xspatula_core_docs/setup_db/netrc/
[import]: {{ "/synthetic/import_data/" | relative_url }}
