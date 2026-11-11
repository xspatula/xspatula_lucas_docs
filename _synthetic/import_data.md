---
title: "Import the data"
layout: single
sidebar:
  nav: "synthetic"
permalink: /synthetic/import_data/
author_profile: false
excerpt: "One notebook inserts everything: utilities, dataset metadata, samples, observation logs, lab values, landscape classes and spectra."
date: 2026-11-11
last_modified_at: 2026-11-11
---

**Notebook**: `xspatula_lucas/synthetic_dataset/import_data/insert_synthetic_data.ipynb`
**Scheme**: `scheme_file = '../scheme_synthetic.json'`

```json
{
  "project_path": "../synthetic_dataset",
  "user_project": {
    "user_netrc_id": "ini_user_synthetic"
  }
}
```

Run the cells in order. Each inserts one kind of record; later cells depend on the earlier ones (a sample needs its sampling log, a lab value needs its sample and observation log).

| Cell | Inserts |
|---|---|
| 1.1 | General utilities (territories, units). Skip it if you ran [Register an extra user][extra_user] |
| 1.2 | Observation utilities: the lab methods and indicators |
| 1.3 | Observation utilities with inheritance: the provisions, including the spectral provision `foss xds 10nm` |
| 1.4 | Landscape utilities: land cover, land use and BIOGEO16 classes |
| 2 | Dataset metadata: data source, person, dataset, the 2 campaigns and their sampling logs |
| 3.1–3.2 | Geolocations and samples, calibration and validation |
| 4 | 8 observation logs (lab, spectra, landscape and BIOGEO16, for each set) |
| 5.1–5.2 | Lab values. The USDA texture class is derived from clay, silt and sand on insert |
| 6.1–6.2 | Land cover, land use and BIOGEO16 per sample |
| 7 | The spectrometer, with its 10 nm wavelength array |
| 7.2–7.3 | 2000 calibration and 300 validation spectra, one JSON file per sample. This takes a while |

## Two routes into the database

Everything except the spectra goes from excel sheets into the database through `insert_tabular_data`: each sheet is translated to JSON and inserted in one step. A 211-band spectrum does not fit in an excel cell, so the spectra come as one JSON file per sample, inserted with `manage_observation`.

**The spectra are stored as diffuse reflectance.** They become absorbance only when a dataset is compiled (`output_spectra_signal: absorb`, next page).

## What the tables hold

The synthetic data use the same tables as LUCAS. For what each table stores, see the LUCAS reference pages: [Dataset metadata explained][lucas_meta], [Utility explained][lucas_utility], [Samples explained][lucas_samples] and [Observations explained][lucas_obs].

## Next step

[Compile and inspect][compile].

[extra_user]: {{ "/synthetic/setup_database/#7-register-an-extra-user-optional" | relative_url }}
[compile]: {{ "/synthetic/compile/" | relative_url }}
[lucas_meta]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/dataset_meta_explained/
[lucas_utility]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/utility_explained/
[lucas_samples]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/samples_explained/
[lucas_obs]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/observations_explained/
