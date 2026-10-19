# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. The initial CLAUDE.md is saved as `.claude/CLAUDE_version1_20260904.md`. The second CLAUDE.md that is also completed is saved as `.claude/CLAUDE_version2_20260904.md`. The third CLAUDE.md, also completed, is saved as `.claude/CLAUDE_version3_20260904.md`. And so are the fourth `.claude/CLAUDE_version4_20260905.md`, fifth `.claude/CLAUDE_version5_20260905.md` and sixth `CLAUDE_version6_20260906.md`, seventh `CLAUDE_version7_20260907.md` and eighth `CLAUDE_version8_20260909.md`.

## New tasks

The way to set the indicators unit and edit unit_translate.

## Task 1 - Indicator unit setting


To Harmonise different data recorded with different units, users can set the required unit(s) to use when saving a database selection to a local .parquet file (format used by python pandas). The simplest alternative is to set
```
"unit": "native",
```

that will use the database recorded unit. But if you want to translate from one unit to another (say "g/Kg" to "percent") the unit translation must first be defined in the observation_utility.unit_translate table (e.g. to go from g/Kg to percent requires multiplying g/Kg with a factor of 0.1). Thus if you have two different datasets that you want to combine at a later stage, and one is in percent and the other in g/Kg you must translate one of them. When extracting the data (process: ) make sure you set the same unit for both extractions. 
