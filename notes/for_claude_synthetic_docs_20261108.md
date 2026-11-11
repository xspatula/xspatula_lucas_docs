# For Claude

## This is inteded as a full covered step-step guide for setting up, seeding and running the "synthetic" project.

Do not include the software setup.

Figures to use are under `/assets/media/synthetic/``

## Aim and scope

The aim of this tutorial is a to go through the how the Xspatula framework can be used for modleing soil properties using Near Infra-Red (NIR) spectral. The data is synthetically derived from the LUCAS 2015 soil sampling campaign. Neither the wet laboratory results nor the spectra are actual data points, they have all been generated synthetically and do not represent any actual soil sample.

THIS DATA CAN NOT BE USED FOR MODELING REAL SOIL CONDITIONS, IT IS A ONLY INTENDED FOR EXPLORING THE POSSIBILITIES FOR MODELING SOIL PHYSICAL AND CHEMICAL PROPERTIES FROM NIR SPECTRAL DATA.

## Prerequisits

Link to the software setup.

## Setup_db

### 1 scheme_local_setup_synthetic.json
Create a new JSON scheme file for setting up a new database with an initial database user for the synthetic dataset, e.g.  `setup/zzz/scheme_local_setup_synthetic.json`, change the database name, host_netrc_id, the user name and credentials, dump_path and default verbosity:

```
{
  "project_path": "./lucas",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "synthetic",
    "host_netrc_id": "karttur",
    "dump_path": "~/xspatula/db_dump/xspatula_synthetic.dump",
    "db_users": [
      {
        "user_netrc_id": "community_admin",
        "role": "community_admin"
      },
      {
        "user_netrc_id": "login_evaluation",
        "role": "login_evaluation"
      },
      {
        "user_netrc_id": "user_cat_1",
        "role": "user_cat_1"
      },
      {
        "user_netrc_id": "user_cat_2",
        "role": "user_cat_2"
      },
      {
        "user_netrc_id": "user_cat_3",
        "role": "user_cat_3"
      },
      {
        "user_netrc_id": "user_cat_4",
        "role": "user_cat_4"
      },
      {
        "user_netrc_id": "user_cat_5",
        "role": "user_cat_5"
      }
    ]
  },
  "initial_user": {
    "user_netrc_id": "ini_user_synthetic",
    "email": "initial_user@example.com",
    "email_alt": "initial_user_alt@example.com",
    "first_name": "Initial",
    "last_name": "User",
    "organisation_id__organisation_name": "xspectre",
    "territory_id__territory_name": "sweden",
    "stratum_code": "5"
  },
  "process": [
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false
    }
  ]
}
```

### 2 .netrc

For this to work you **must** create a hidden `.netrc` file in your home directory that specifies both the host_netrc_id user (for your postgres database cluster), postgres users (pg_users, for the new database to create) and the initial user (initial_user, for the new database). These are the required lines to put in the `.netrc` file, but you should edit all passwords. And you must put your real credentials for the postgres cluster where you are setting up the new database.

```
host_netrc_id login your_pg_cluster_user password your_pg_cluster_user_password
machine ini_user_synthetic login ini_user password pswd-ini-user
machine community_admin login community_admin password pswd_admin
machine login_evaluation login login_evaluation password pswd-login
machine user_cat_1 login user_cat_1 password pswd-cat-1
machine user_cat_2 login user_cat_2 password pswd-cat-2
machine user_cat_3 login user_cat_3 password pswd-cat-3
machine user_cat_4 login user_cat_4 password pswd-cat-4
machine user_cat_5 login user_cat_5 password pswd-cat-5
```

You can give user name and password in plain text but that is risky.

### 3 Adding more initial users - optional

for Claude - can you please add this optional section,

How to do if also other initial users are setup via `setup/zzz/lucas/setup_db/json/community/user_records_v10_sql.json`



### 4 open and run notebook setup_db

Open the notebook `setup/setup_db.ipynb` and edit the Scheme file cell to point to your JSON setup scheme file (3).

```
# Set the path to the scheme_file

scheme_file = './zzz/scheme_local_setup_synthetic.json'
```

If you set everything correct the database (synthetic) will be be complete setup. The setup includes process definitions for defining all process.

### 5 scheme_local_use_synthetic.json

Create a new JSON scheme file for using the new database with your initial user defined under (4), e.g.  `setup/zzz/scheme_local_use_synthetic.json`, change `user_netrc_id` to the `.netrc` code of your initial user, the project path must still be "./lucas":

```
{
  "project_path": "./lucas",
  "user_project": {
    "user_netrc_id": "ini_user_synthetic"
  },
  "process": [
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false
    }
  ]
}
```
## setup processes

### 6 open and run notebook setup_processes

Open the notebook `setup/setup_processes.ipynb` and edit the Scheme file cell to point to your JSON user scheme file (5).

```
# Set the path to the scheme_file

scheme_file = './zzz/scheme_local_use_synthetic.json'
```

All the processes required for running the synthetic LUCAS project should be inserted in the database.

### 7 Add users via notebook

or the notebook
`synthetic_dataset/user_management/register_users.ipynb`
And what the user needs to do to fix the hash crypted password.

## Import data

### 8 Import data to the database

open and run the notebook `synthetic_dataset/import_data/insert_synthetic_data.ipynb` to insert all the required data:
- utilities
- dataset metadata
- geolocations
- wetlab
- land cover
- land use
- biogeo16
- spectra

The following physicochemical properties are inserted:


The spectra is reduced from its original resolution to 10 nm. It is imported as absorbance.

There are four nominal (classified) covariates included in the dataset:
- biogeographic regions
- land cover
- land use
- soil textural classes

## ML model

### 9 Compile and inspect the dataset

Open and run the notebook `synthetic_dataset/project_synthetic/1_compile_dataset.ipynb`. It compiles the selected indicators, the spectra and the covariates into a single dataframe.

Use `1.3 Look at the calibration data` to first inspect the indicators; tot-n and ec are log transformed.


And then the spectra.

### 10 preprocess the indicators and spectra

Open and start the notebook `synthetic_dataset/project_synthetic/2_ml_preprocess.ipynb` to apply-preprocessing to both the indicators and the spectra.

For the docs: notebook 2 waits for keyboard answers (threshold, which
  outliers to remove, accept SNV, accept mean-centring). That's the most
  likely place for a newcomer to get stuck in VS Code, so it's worth a
  clear note in the docs.

#### Remove indicator outliers

Cell `2.1 Indicator outliers` is set to remove 5% of the data points for c-org, n-tot and ec using the isolation forest outlier detector. When you run the the cell you can interactively change the fraction of samples to remove and then also decide for each indicator (c-org, n-tot and ec) if you want to remove the identified indicators or not.

In the example below the threshold was changed to 0.025

#### Pre-process spectra

The spectra pre-processing is setup to a chain with two consecutive steps:
- SNV
- Meancentring

If you save the SNV output, the Meancentring will use that as input, otherwise it will use the origincal data (it is set to use whatever dataframe was the previous as input). You will see that in the plot.

First run Cell 2.2, you must save the snv dataframe for it to be available for...

... Cell 2.3 meancentring. Save the meancentring. The dataframe from your pre-process chain will not have the name

`synthetic_calibration_10nm.olif.snv.mc` with one dataframe (.parquet) and one file describing the complete pre-processing chain (.json). The dot separated suffix indicate the process history:

- olif: indicator outlier detection (isolation forest)
- snv: spectral scatter correction
- mc: spectral rescaling by meancentring

The next step (Creating a model) expects a file with this name as input.

### 11 Create a model

 Open and start the notebook `synthetic_dataset/project_synthetic/3_ml_model.ipynb` to create models for predicting soil properties from spectral data.

The soil indicator properties that are modeled include:
- c-org
- n-tot
- ph-water
- clay

 The models applied are:
- plsr
- svr
- randfor
- cubist

To Claude: Explain CEll 3.1 a bit

Run the model from Cell 3.2. and you will get both numerical results and plots of all combinations between indicators and regressors.

The model is set to only run a quicker train-test evaluation mode and not the slower but more comprehensive kfold evaluation mode. To also run a kfold model calibration, open the JSON file `synthetic_dataset/project_synthetic/process/ml_model/regression_modeling.json` and set

```
"kfold": true,
```

### 12 Independent prediction

Open and start the notebook `synthetic_dataset/project_synthetic/4_ml_predict.ipynb`. It uses an independent dataset of 300 samples.

#### Validate using parquet + JSON
Cell 4.1 validates all four indicators against all 4 regressors.


#### Register model
Cell 4.2 Registers the model to the database, this should only be done after throrougly testing different pre-processing and hyper-parameterization. But is used here to show how it works.

To Claude: Here it would b better to delete the whole data folder (`synthetic_dataset/project_synthetic/data`)

With the model you only need the independent dataframe to rerun the predictions.
To Claude: is this correctt?

#### Delete dataframes

Delete all dataframes that are no longer required - which I guess means that all data can be deleted and then just a rerun of cell 1.2 Validation dataset in notebook "1_compile"

#### Predict with registered model

To Claude - can we make a new Cell 4.3 that deletes the not needed data and then the user can re-run the old Cell 4.3 (relabelled to 4.4)?

or we delete the whole folder `synthetic_dataset/project_synthetic/data` (requires a new process) and then make a copy of cell 1.2 (compile validation dataset), and then run "Predict with registered model" - would be the most illustrating.
