---
title: "Spectra"
layout: single
sidebar:
  nav: "spectra"
permalink: /spectra/
author_profile: false
---

LUCAS 2009 soil samples were scanned on a FOSS XDS Rapid Content Analyzer. Spectral data is
stored as a diffuse-reflectance array per sample scan, generated from the source CSV's `spc.*`
columns by `lucas_2009_to_xspatula.py` — see [LUCAS 2009 → Prepare data][prepare_data].

| Instrument | Provision | Provision ID |
|---|---|---|
| FOSS XDS Rapid Content Analyzer | `foss xds rca` | `foss-xds-rca` |

## Three loading steps

As with any Xspatula instrument, spectral data requires three steps in order:

1. [Manage spectrometer] — register the instrument and its wavelength axis
2. [Manage observation log] — link the campaign's sampling log to the instrument's provision
3. [Manage observation] — insert the spectral arrays, one per sample scan

All three run from `xspatula_lucas/lucas/import_data/load_LUCAS.ipynb`, after
[sample][sample] loading.

## Prerequisites

[Sample][sample] loading must be complete. The campaign and sampling log must already exist —
see [LUCAS 2009 → Load campaign][load_campaign].

[prepare_data]: /lucas_2009/prepare_data/
[load_campaign]: /lucas_2009/load_campaign/
[sample]: /sample/
[Manage spectrometer]: /spectra/manage_spectrometer/
[Manage observation log]: /spectra/manage_observation_log/
[Manage observation]: /spectra/manage_observation/
