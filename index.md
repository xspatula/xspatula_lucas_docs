---
layout: home
sidebar:
  nav: "main_navigation"
author_profile: true
excerpt: "Loading the published LUCAS soil sampling campaigns into a database with the Xspatula framework."
---

# LUCAS soil data with Xspatula

`xspatula_lucas` builds a PostgreSQL database seeded with the published [LUCAS](https://esdac.jrc.ec.europa.eu/projects/lucas) (Land Use/Cover Area frame Survey) soil sampling campaigns, using the [Xspatula](https://github.com/xspatula) framework.

This site documents the full walkthrough for creating, downloading and seeding an Xspatula framework for the LUCAS 2009 soil sampling and analysis campaign.

The generic Xspatula framework — how to set up the database itself, define processes, add users, and audit changes — is documented once, for every Xspatula project, at **[xspatula_core_docs](https://xspatula.github.io/xspatula_core_docs/)**. Start there if you haven't set up an Xspatula database yet.

## Where to start

- New to Xspatula? Start with [Setup DB][setup_db] on the core docs site.
- Already have a database running? Jump straight to [LUCAS 2009][lucas_2009] to download and load the campaign.
- Looking for what a specific table stores? See [Dataset metadata][dataset_meta], [Sample][sample], or [Spectra][spectra].

## Access and license

- Data License: Creative Commons Attribution license (**CC-BY**)
- Code License: Massachusetts Institute of Technology License (**MIT License**)

[setup_db]: https://xspatula.github.io/xspatula_core_docs/setup_db/
[lucas_2009]: ./lucas_2009
[dataset_meta]: ./dataset_meta
[sample]: ./sample
[spectra]: ./spectra
