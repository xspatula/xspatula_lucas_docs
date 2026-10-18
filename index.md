---
layout: home
sidebar:
  nav: "main_navigation"
author_profile: true
excerpt: "Loading the published LUCAS soil sampling campaigns into a database with the Xspatula framework."
---

# LUCAS soil data with Xspatula

The open source `xspatula_lucas` project available on [GitHub](https://github.com/xspatula/xspatula_lucas) builds a PostgreSQL database seeded with the published [LUCAS](https://esdac.jrc.ec.europa.eu/projects/lucas) (Land Use/Cover Area frame Survey) soil sampling campaigns, using the [Xspatula](https://github.com/xspatula) framework. Once the database is completed you can apply Machine Learning for modeling soil properties from spectral data.

The generic Xspatula framework — how to set up the database itself, define processes, add users, and audit changes — is documented once, for every Xspatula project, at **[xspatula_core_docs](https://xspatula.github.io/xspatula_core_docs/)**. Start there if you haven't set up an Xspatula database yet.

## Where to start

- New to Xspatula? Start with [Setup DB][setup_db] on the core docs site or link to a specific topic under **SETUP XSPATULA** in the menu to the left.
- Already have a database running? Jump straight to [LUCAS 2009][lucas_2009] to download and load the campaign.
- LUCAS 2009 done? [LUCAS 2015][lucas_2015] follows the same steps. Its page only covers what differs.
- Both loaded? [Dump the database][lucas_dump] once. A restore takes minutes instead of hours ([Dump and restore][dump_restore] on the core docs site).
- Looking for what a specific table stores? See [Dataset metadata explained][dataset_meta], [Utility explained][utility], [Samples explained][samples], or [Observations explained][observations] — all reachable from the **Reference** heading under LUCAS 2009 in the menu to the left.

## Access and license

- Documentation License: Creative Commons Attribution license (**CC-BY**)
- Code License: Massachusetts Institute of Technology License (**MIT License**)
- LUCAS soil data: **not** covered by the licenses above. The LUCAS 2009 and LUCAS 2015 data are released by ESDAC under the [LUCAS_SOIL licence agreement][lucas_license], which you accept when you request the download. The licence is personal and does not permit sharing the data, a database dump with the data, or files derived from it outside your own organisation. See [LUCAS license][lucas_license_page].

[setup_db]: https://xspatula.github.io/xspatula_core_docs/setup_db/
[lucas_2009]: ./lucas_2009
[lucas_2015]: ./lucas_2015
[lucas_dump]: ./lucas_2009/#dump-once-restore-in-minutes
[lucas_license_page]: ./lucas_2009/#lucas-license-the-data-may-not-be-shared
[dump_restore]: https://xspatula.github.io/xspatula_core_docs/setup_db/dump_restore/
[lucas_license]: https://esdac.jrc.ec.europa.eu/projects/LUCAS/Documents/LUCAS_SOIL_LIC_AGR_final_for_web.pdf
[dataset_meta]: ./lucas_2009/dataset_meta_explained
[utility]: ./lucas_2009/utility_explained
[samples]: ./lucas_2009/samples_explained
[observations]: ./lucas_2009/observations_explained/
