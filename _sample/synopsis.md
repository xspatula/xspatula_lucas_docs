---
title: "Sample"
layout: single
sidebar:
  nav: "sample"
permalink: /sample/
author_profile: false
---

A sample in the LUCAS database is a soil specimen collected at a known geolocation, with a
0–20 cm depth profile. Each sample belongs to a sampling log (see [Dataset metadata]), which
belongs to a campaign. Samples must exist before any lab or spectral observation can be entered.

Unlike [dataset metadata][Dataset metadata] — a handful of hand-authored Excel rows — geolocation
and sample records are generated per-point from the source CSV by
`lucas_2009_to_xspatula.py` (one record per LUCAS sampling point, thousands of them). See
[LUCAS 2009 → Prepare data][prepare_data] for how they're generated, and
[LUCAS 2009 → Load campaign][load_campaign] for how they're inserted.

## Two tables

| Table | Populated from | Records |
|---|---|---|
| `observation.geolocation` | `GPS_LONG` / `GPS_LAT` columns | One per unique `POINT_ID` |
| `observation.geolocated_profile_sample` | `POINT_ID`, `SURV_DATE`, `iso.country` columns | One per unique `POINT_ID`, depth 0–20 cm |

## Required loading sequence

1. [Manage geolocation] — insert the GPS coordinate for each sampling point
2. [Manage sample] — insert the sample record, linked to its geolocation and sampling log

Both run from `xspatula_lucas/lucas/import_data/load_LUCAS.ipynb`, immediately after the campaign
and sampling log steps.

[Dataset metadata]: /dataset_meta/
[prepare_data]: /lucas_2009/prepare_data/
[load_campaign]: /lucas_2009/load_campaign/
[Manage geolocation]: /sample/manage_geolocation/
[Manage sample]: /sample/manage_sample/
