---
title: "Try Xspatula with synthetic soil data"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/
author_profile: false
excerpt: "A complete run from an empty database to a registered machine-learning model, on a small synthetic soil spectral library. The quickest way to see how Xspatula works."
date: 2026-11-11
last_modified_at: 2026-11-11
---

<div class="notice--warning" markdown="1">
**The data are synthetic.** They are derived from the LUCAS 2015 topsoil survey, but neither the laboratory values nor the spectra are real measurements of any soil sample. The data cannot be used to model real soil conditions. They are only for learning how to model soil physical and chemical properties from NIR spectra with Xspatula.
</div>

<div class="notice--info" markdown="1">
**The synthetic dataset is not yet distributed.** Permission to share data derived from LUCAS 2015 has been requested from the data owner. Until it is granted, these pages show how the project is run, with the outputs of a complete run.
</div>

This tutorial takes you from an empty PostgreSQL database to a registered model that predicts soil organic carbon from spectra. Every step is a Jupyter notebook cell you run in order. The dataset is small (2300 samples), so a full run takes an afternoon instead of the days a full LUCAS campaign needs.

## What the data are

- **2000 calibration samples** (model formulation) and **300 independent validation samples**.
- Each sample is a random-weight mix of **3 LUCAS 2015 points** that share land cover family, land use family and biogeographic region, and lie within 50 km of each other. The same weights apply to the spectra, the lab values and the coordinates.
- No LUCAS point is used in both sets, so the validation set is truly independent.

| | Content |
|---|---|
| Lab indicators | organic carbon `c-org`, total nitrogen `n-tot`, electrical conductivity `ec`, pH in water `ph-water`, `clay`, `silt`, `sand`. Texture is missing for about 23% of the samples |
| Spectra | 400–2500 nm in 10 nm bands (211 bands), from the FOSS XDS scans of LUCAS 2015 |
| Nominal covariates | biogeographic region (BIOGEO16), land cover, land use, USDA soil texture class |
| Sample names | `cal0001@0-20` … `cal2000@0-20`, `val0001@0-20` … `val0300@0-20` |
| Database | `synthetic` |

## Prerequisites

You need the software installed and running: PostgreSQL, a Python environment with the `xspatula_lucas` code, and VS Code (or another Jupyter front end). Installation is covered once for all Xspatula projects in [Setup DB, prerequisites][core_prereq] on the core documentation site. This tutorial does not repeat it.

Before you start, make sure PostgreSQL actually checks passwords: see [Securing authentication][core_hba].

## Run order

| Step | Page | Notebook |
|---|---|---|
| 1 | [Set up the database][setup] | `setup/setup_db.ipynb`, `setup/setup_processes.ipynb` |
| 2 | [Import the data][import] | `synthetic_dataset/import_data/insert_synthetic_data.ipynb` |
| 3 | [Compile and inspect][compile] | `project_synthetic/1_compile_dataset.ipynb` |
| 4 | [Preprocess][preprocess] | `project_synthetic/2_ml_preprocess.ipynb` |
| 5 | [Model][model] | `project_synthetic/3_ml_model.ipynb` |
| 6 | [Predict and register][predict] | `project_synthetic/4_ml_predict.ipynb` |
| 7 | [Covariates][covariates] | `project_synthetic/5_…` to `8_…` |

Steps 1–2 build the database. Steps 3–7 are the machine-learning chain: compile a dataset from the database, preprocess the spectra, train models, validate them on independent samples, register the best one, and test whether location-based information (region, land cover, land use, texture) improves the predictions.

Each page links to the general machine-learning pages of the LUCAS documentation for the full parameter reference.

[core_prereq]: https://xspatula.github.io/xspatula_core_docs/setup_db/#prerequisits
[core_hba]: https://xspatula.github.io/xspatula_core_docs/setup_db/postgres/#securing-authentication-pg_hbaconf
[setup]: {{ "/synthetic/setup_database/" | relative_url }}
[import]: {{ "/synthetic/import_data/" | relative_url }}
[compile]: {{ "/synthetic/compile/" | relative_url }}
[preprocess]: {{ "/synthetic/preprocess/" | relative_url }}
[model]: {{ "/synthetic/model/" | relative_url }}
[predict]: {{ "/synthetic/predict_register/" | relative_url }}
[covariates]: {{ "/synthetic/covariates/" | relative_url }}
