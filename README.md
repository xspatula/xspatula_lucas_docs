# xspatula_lucas_docs

Documentation site for `xspatula_lucas` — loading the published LUCAS soil sampling campaigns
into a PostgreSQL database with the Xspatula framework.

**Live site**: [xspatula.github.io/xspatula_lucas_docs](https://xspatula.github.io/xspatula_lucas_docs)

---

## Scope

This site documents only the LUCAS-specific parts of the build. The main content is the
`lucas_2009` collection: the full download → prepare → insert walkthrough for the LUCAS 2009 campaign, its
downstream explore/preprocess/model machine learning pipeline, and reference pages on the dataset
metadata, utility, sample, and observation tables involved. There is deliberately no separate
top-level collection for any of that reference material — it's reachable only from within
`lucas_2009`, since it's detail, not a parallel entry point. The generic Xspatula framework —
database setup, process definitions, auditing, community/user management — is documented once for
every Xspatula project at [xspatula_core_docs](https://xspatula.github.io/xspatula_core_docs/),
and this site links out to it rather than duplicating it. See `.claude/CLAUDE.md` for the full
reasoning behind that split, including why an earlier `_dataset_meta` collection was folded back
into `lucas_2009`.

Each later LUCAS campaign gets its own small collection that documents only what differs from
2009 and links back to the 2009 pages for the shared steps. So far that is `lucas_2015`: a single
page covering the different source files, `lucas_2015_to_xspatula.py` (including the texture
backfill from 2009 for revisited points) and the `load_LUCAS_2015.ipynb` cells. Campaigns are kept
separate rather than merged into one "LUCAS 2009/2015" collection because they diverge: LUCAS 2018
has eDNA instead of spectra. eDNA is not documented yet.

BIOGEO16 (EEA biogeographic regions, joined from `LUCAS-Master-Grid.csv`) is a separate dataset and
campaign (`biogeo16`), but it is loaded by the last three cells of both the 2009 and 2015 load
notebooks. It is documented in the `lucas_2009` pages (Load LUCAS 2009, Observations explained).

## Content sections

Most content lives under `/lucas_2009/`, split into three groups in its own sidebar:

| Group | Pages | Covers |
|---|---|---|
| Seed LUCAS data (required, in order) | synopsis, prepare data, insert utility, insert dataset metadata, load LUCAS 2009 | The actual runbook for loading the campaign |
| Reference (optional) | dataset metadata explained, utility explained, utility inherit/auto explained, foreign key explained, samples explained, observations explained | Table/parameter detail behind the runbook steps |
| Machine learning | explore & select data, ML preprocessing, ML modeling, inspect dataset | `/lucas_2009/machine_learning/...` — a separate Jekyll collection, nested under the same URL prefix |

LUCAS 2015 is a single page at `/lucas_2015/` (collection `_lucas_2015/`, sidebar `lucas_2015`).
It covers the differences from 2009 plus the load notebook's cells, and its sidebar links back
to the shared 2009 pages.

## Site architecture

- **Engine**: Jekyll with the Minimal Mistakes theme (v4.27.3, local install via `Gemfile`).
- **Serve locally**: `bundle exec jekyll serve --config _config.yml,_config_local.yml`
- **Build**: `bundle exec jekyll build`
- **Deploy**: GitHub Actions (`.github/workflows/jekyll.yml`) on push to `main`. Requires
  repo Settings → Pages → Build and deployment → Source = "GitHub Actions".
- **Content**: three Jekyll collections, `_lucas_2009/`, `_machine_learning/` (permalinks nested
  under `/lucas_2009/machine_learning/...` even though it's a separate collection) and
  `_lucas_2015/`, each with
  `output: true` in `_config.yml` and a page order under `nav_order:`.
- **Navigation**: hand-maintained in `_data/navigation.yml`. Entries for the generic framework
  sections are external links to `xspatula_core_docs`, not local pages. The `lucas_2009:` sidebar
  key is shared by pages in both collections (both set `sidebar: nav: "lucas_2009"`), which is
  what makes its three-group split show up everywhere. `_lucas_2015/` uses its own `lucas_2015`
  sidebar key.
- **Page order**: previous/next pagination follows `_data/story_order.yml`, not Jekyll's default
  per-collection order — see `_includes/post_pagination.html`.

## Source repositories

- [xspatula/xspatula_lucas](https://github.com/xspatula/xspatula_lucas) — the Python package this
  site documents.
- [xspatula/xspatula_core_docs](https://github.com/xspatula/xspatula_core_docs) — the generic
  framework documentation this site links out to.

## Licenses

- **Code**: [MIT License](LICENSE)
- **Data**: [Creative Commons Attribution (CC-BY)](https://creativecommons.org/licenses/by/4.0/)
