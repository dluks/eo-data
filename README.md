Geosense EO Data Registry
==============================

A data registry of Earth observation data commonly used by the Geosense group at the University of Freiburg.

Project Organization
------------

    ├── LICENSE
    ├── README.md          <- The top-level README for developers using this project.
    ├── data               <- Data available for download with `dvc get` or `dvc import`
    │
    ├── models             <- Trained and serialized models, model predictions, or model summaries
    │
    ├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering),
    │                         the creator's initials, and a short `-` delimited description, e.g.
    │                         `1.0-jqp-initial-data-exploration`.
    │
    ├── references         <- Data dictionaries, manuals, and all other explanatory materials.
    │
    ├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
    │   └── figures        <- Generated graphics and figures to be used in reporting
    │
    ├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
    │                         generated with `pip freeze > requirements.txt`
    │
    ├── setup.py           <- makes project pip installable (pip install -e .) so src can be imported
    ├── src                <- Source code for use in this project.
    │   ├── __init__.py    <- Makes src a Python module
    │   │
    │   ├── data           <- Scripts to download or generate data
    │   │   └── download_gbif.py
    │   │
    │   ├── features       <- Scripts to turn raw data into features for modeling (placeholder for now)
    │   │   └── build_features.py
    │   │
    │   ├── models         <- Scripts to train models and then use trained models to make
    │   │   │                 predictions (placeholders for now)
    │   │   ├── predict_model.py
    │   │   └── train_model.py
    │   │
    │   └── visualization  <- Scripts to create exploratory and results oriented visualizations
    │       └── visualize.py
    │
    ├── tests              <- Pytest test scripts
    ├── .dvc               <- Contains DVC data cache and config
    ├── dvc.yaml           <- Defines DVC pipeline stages
    ├── params.yaml        <- Define parameters used in DVC pipeline stages
    ├── pyproject.toml     <- Human-readable project dependencies managed with Poetry
    ├── poetry.lock        <- File used by Poetry to install dependencies
    └── .pre-commit-config.yaml <- pre-commit Git hooks


--------

<p><small>Project based on the <a target="_blank" href="https://drivendata.github.io/cookiecutter-data-science/">cookiecutter data science project template</a>. #cookiecutterdatascience</small></p>

## Usage
### Downloading data
Downloading data from this registry is simple—that's the point!

To see what data is available, use `dvc list`, for example:
```bash
dvc list -R --dvc-only path/to/eo-data data
```
The above command lists all files available in the `data` directory of the `eo-data` registry (located in this case at `path/to/eo-data`). The `-R` flag tells DVC to list all directory contents recursively, and `--dvc-only` only shows the actual data, excluding system files like `__pycache__` and `.gitignore`.

Download the data you want with `dvc get` or `dvc import`. Use `dvc get` if you don't want to automatically track your local copy of the data in your own DVC project. Using `dvc import` checks the files into DVC versioning.
```bash
dvc import path/to/eo-data data/raw/gbif/all_tracheophyta.zip -o myproject/data/raw/gbif
```

> [!NOTE]
> DVC supports remote storage, just like Git. In this case, you would **not** want to set this registry as a DVC remote for your project, since it should be considered *read-only*. Configure a DVC remote for your project only if you want to push your data (and its intermediate transformations and features) to a project-specific remote.

## Development
### Setup

#### 1. Clone the project repository

```bash
git clone https://github.com/dluks/eo-data
cd eo-data
```

#### 2. Install poetry and DVC.

```bash
pipx install poetry==~1.7
pipx install dvc
```
> [!TIP]
> If you don't have root access, you can use `condax`, a conda-flavored implementation of `pipx`.
> ```bash
> pip install --user condax
> ```

#### 3. Create a virtual environment

```bash
conda create -n eo-data -c conda-forge python=3.10
conda activate eo-data
```

#### 4. Install dependencies
Install dependencies with Poetry.
```bash
poetry install
```

#### 5. Install pre-commit Git hooks (optional)

```bash
pre-commit install
```

> [!NOTE]
> If you'd prefer not to use `pre-commit` for managing Git hooks, simply remove `pre-commit` with
> ```bash
> poetry remove pre-commit
> ```
> If you've already installed the hooks with `pre-commit install`, you'll need to run `pre-commit uninstall`.

### Running pipelines
> [!WARNING]
> This section is for **reproducing core data registry pipelines**. In other words, reproducing pipeline stages may result in modifcations or even removal of data, which could result in breaking changes for other parties who are simply pulling data for their own projects. Only run pipelines (e.g. `dvc repro` or `dvc exp run`) if you are sure you need to! If you only want to download data, see the above section on [Downloading data](#downloading-data).

The entire data creation and transformation process can be reproduced using DVC pipelines. You can reproduce the entire pipeline with
```bash
dvc repro
```
but (more common) you may want to simply run a single stage, with
```bash
dvc repro my_stage
```

#### `download_gbif` DVC pipeline stage
To reproduce the GBIF download of all non-cultivated plant occurrences in Tracheophyta, first check to see if any of the data files (outs) have changed.
```bash
dvc status download_gbif
```
If there have been changes, or if you simply want to get the most up-to-date query from GBIF, take the following steps:

1. Create (or reuse) a JSON query in `references/gbif`
2. View `dvc.yaml` to familiarize yourself with what the stage does.
3. Update `params.yaml` accordingly (e.g. you will most likely at least want to update `gbif.query_date` for clear versioning).
4. Run the pipeline stage:
```bash
dvc repro download_gbif
```