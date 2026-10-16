# For Claude: miscellaneous fixes

## 1 land use/cover utility table

https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/observations_explained/#land-cover-and-land-use

Refers to the old 3-file hierarchical table structure of land use/cover classification that we changed to a single table with internal foreign keys and a catalogue of the ranks.

## 2 Sampling log for BIOGEO16

The sampling log for the dataaset BIOGEO16 dataset and campaign are missing in
https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/observations_explained/#land-cover-and-land-use
Insert it as it appears in in the repo `lucas/import_data/insert_lucas_all_data.ipynb` and explain why it needs its own sampling log - this is not observed as part of LUCAS, but is a map product from EEA (European Environment Agency).

## 3 Observation log for BIOGE16
Same as above 2, but for the observation log.

## 4 List of LUCAS 2009 data download

Create a list of the LUCAS 2009 data to download, and URL if possible, for running the CLI `lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`

## 5 List of LUCAS 2015 data download

Create a list of the LUCAS 2015 data to download, and URL if possible, for running the CLI `lucas/prepare_lucas_data/lucas_2009_to_xspatula.py`

## 6 Complete LUCAS import in a single notebook

Instead of using 4 separate notebooks for importing the LUCAS 2009 and 2015 data, I created a single notebook for the complete pipeline covering both 2009 and 2015. It is very long, but more efficient. The whole notebook runs clean and loads the entire database in a single run. It is also possible to "turn of" the data not required - but uncommenting the first rows of each code cell (`%%script false --no-raise-error`). I want to switch to only using this single notebook and thus update the documentation accordingly. All pages should remain as they are, but instead of referring to specific notebooks, refer to the single notebook and its numbered and named blocks. The block numbering should also improve the findability from the docs by giving the cell number + name.

## 7 Check import pipeline consistency

Once you have gone through points 1 to 6, check that the pipeline described for both 2009 and 2015 are consistent with the cells in the notebook `lucas/import_data/insert_lucas_data.ipynb`.
