# xspatula_lucas_docs

Documentation site for `xspatula_lucas` — loading the published LUCAS soil sampling campaigns
into a PostgreSQL database with the Xspatula framework.

**Live site**: [xspatula.github.io/xspatula_lucas_docs](https://xspatula.github.io/xspatula_lucas_docs)

---

## Scope

This site documents only the LUCAS-specific parts of the build: dataset metadata, sample and
spectra records, and the full download → prepare → insert walkthrough for the LUCAS 2009
campaign. The generic Xspatula framework — database setup, process definitions, auditing,
community/user management — is documented once for every Xspatula project at
[xspatula_core_docs](https://xspatula.github.io/xspatula_core_docs/), and this site links out to
it rather than duplicating it. See `.claude/CLAUDE.md` for the full reasoning behind that split.

## Content sections

| Section | URL | Covers |
|---|---|---|
| Dataset metadata | `/dataset_meta/` | Data source, persons, dataset, and campaign records |
| Sample | `/sample/` | Geolocations and sample records for LUCAS sampling points |
| Spectra | `/spectra/` | FOSS XDS RCA spectrometer, observation logs, and spectral observations |
| LUCAS 2009 | `/lucas_2009/` | Download the source CSV, run the prep script, and load the full campaign |

## Site architecture

- **Engine**: Jekyll with the Minimal Mistakes theme (v4.27.3, local install via `Gemfile`).
- **Serve locally**: `bundle exec jekyll serve --config _config.yml,_config_local.yml`
- **Build**: `bundle exec jekyll build`
- **Deploy**: GitHub Actions (`.github/workflows/jekyll.yml`) on push to `main`. Requires
  repo Settings → Pages → Build and deployment → Source = "GitHub Actions".
- **Content**: one Jekyll collection per LUCAS-specific section (`_dataset_meta/`, `_sample/`,
  `_spectra/`, `_lucas_2009/`), each with `output: true` in `_config.yml` and a page order under
  `nav_order:`.
- **Navigation**: hand-maintained in `_data/navigation.yml`. Entries for the generic framework
  sections are external links to `xspatula_core_docs`, not local pages.

## Source repositories

- [xspatula/xspatula_lucas](https://github.com/xspatula/xspatula_lucas) — the Python package this
  site documents.
- [xspatula/xspatula_core_docs](https://github.com/xspatula/xspatula_core_docs) — the generic
  framework documentation this site links out to.

## Licenses

- **Code**: [MIT License](LICENSE)
- **Data**: [Creative Commons Attribution (CC-BY)](https://creativecommons.org/licenses/by/4.0/)
