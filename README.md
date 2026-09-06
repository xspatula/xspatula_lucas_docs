# xspatula_lucas_docs

Documentation site for `xspatula_lucas` — loading the published LUCAS soil sampling campaigns
into a PostgreSQL database with the Xspatula framework.

**Live site**: [xspatula.github.io/xspatula_lucas_docs](https://xspatula.github.io/xspatula_lucas_docs)

---

## Scope

This site documents only the LUCAS-specific parts of the build, all under a single `lucas_2009`
collection: the full download → prepare → insert walkthrough for the LUCAS 2009 campaign, its
downstream explore/preprocess/model machine learning pipeline, and reference pages on the dataset
metadata, utility, sample, and observation tables involved. There is deliberately no separate
top-level collection for any of that reference material — it's reachable only from within
`lucas_2009`, since it's detail, not a parallel entry point. The generic Xspatula framework —
database setup, process definitions, auditing, community/user management — is documented once for
every Xspatula project at [xspatula_core_docs](https://xspatula.github.io/xspatula_core_docs/),
and this site links out to it rather than duplicating it. See `.claude/CLAUDE.md` for the full
reasoning behind that split, including why an earlier `_dataset_meta` collection was folded back
into `lucas_2009`.

## Content sections

Everything lives under `/lucas_2009/`, split into three groups in its own sidebar:

| Group | Pages | Covers |
|---|---|---|
| Seed LUCAS data (required, in order) | synopsis, prepare data, insert utility, insert dataset metadata, load LUCAS 2009 | The actual runbook for loading the campaign |
| Reference (optional) | dataset metadata explained, utility explained, utility inherit/auto explained, samples explained, observations explained | Table/parameter detail behind the runbook steps |
| Machine learning | explore & select data, ML preprocessing, ML modeling | `/lucas_2009/machine_learning/...` — a separate Jekyll collection, nested under the same URL prefix |

## Site architecture

- **Engine**: Jekyll with the Minimal Mistakes theme (v4.27.3, local install via `Gemfile`).
- **Serve locally**: `bundle exec jekyll serve --config _config.yml,_config_local.yml`
- **Build**: `bundle exec jekyll build`
- **Deploy**: GitHub Actions (`.github/workflows/jekyll.yml`) on push to `main`. Requires
  repo Settings → Pages → Build and deployment → Source = "GitHub Actions".
- **Content**: two Jekyll collections, `_lucas_2009/` and `_machine_learning/` (permalinks nested
  under `/lucas_2009/machine_learning/...` even though it's a separate collection), each with
  `output: true` in `_config.yml` and a page order under `nav_order:`.
- **Navigation**: hand-maintained in `_data/navigation.yml`. Entries for the generic framework
  sections are external links to `xspatula_core_docs`, not local pages. The `lucas_2009:` sidebar
  key is shared by pages in both collections (both set `sidebar: nav: "lucas_2009"`), which is
  what makes its three-group split show up everywhere.
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
