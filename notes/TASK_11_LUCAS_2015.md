# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. The previous CLAUDE.md files are saved as `.claude/CLAUDE_version1_20260904.md` through `.claude/CLAUDE_version10_unit_setting.md`. Please look at these to understand where we are and for reference to the code repo for which this is the documentation.

## New tasks

We finished the adding of the BIOGE16 dataset since the previous update (`.claude/CLAUDE_version10_unit_setting.md`), so please add that to _lucas_2009 `load_lucas_2009.md`.


Together we created the complete chain for also loading LUCAS 2015 data, starting with the script `lucas/prepare_lucas_data/lucas_2015_to_xspatula.py`. The steps are identical to LUCAS 2009, except that the downloaded and stored files are differently organised. And we wrote a function that takes LUCAS 2009 texture data (coarse, sand, silt, clay) and attached it to points without texture but revisited in 2015.

My suggestion is to create one more "main" entry in `_data/navigation.yml` "LUCAS 2015", with a single page. This page should just explain the difference in files and processing and then show each 2015 notebook (`lucas/import_data/load_LUCAS_2015.ipynb`) cell (step) for importing the LUCAS 2015 data to the database and refer back to the relevant (equal) page in the _lucas_2009 collection for details. No use repeating the content of the collection LUCAS 2009 as it is the same sequence. The alternative would be to expand the content of the _lucas_2009 collection too also cover LUCAS 2015, and rename it LUCAS 2009/2015. But the next LUCAS dataset (2018), that I did not start with yet, does not include spectra but instead have eDNA data. So I think it better to have them one at a time.

Talking about eDNA, I also started the adding of eDNA in the code repo (`xspatual_lucas`) but you should ignore that completely for the documentation for now.

Also remember to update the README.md.
