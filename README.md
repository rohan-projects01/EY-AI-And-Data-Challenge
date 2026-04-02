# EY AI and Data Challenge

[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-experimental-yellow.svg)](#)

## Table of Contents

- [What the project does](#what-the-project-does)
- [Why the project is useful](#why-the-project-is-useful)
- [How to get started](#how-to-get-started)
  - [Prerequisites](#prerequisites)
  - [Install dependencies](#install-dependencies)
  - [Prepare data](#prepare-data)
  - [Run model training](#run-model-training)
  - [Generate submission](#generate-submission)
- [Project structure](#project-structure)
- [Where to get help](#where-to-get-help)
- [Who maintains and contributes](#who-maintains-and-contributes)

## What the project does

EY-AI-And-Data-Challenge is a machine learning repository for modeling three water quality targets via geospatial and remote sensing information.

Targets:
- Total Alkalinity
- Electrical Conductance
- Dissolved Reactive Phosphorus

Input data sources:
- TerraClimate, MODIS (vegetation indices + LST), Landsat
- CHIRPS precipitation, ERA5 reanalysis, JRC surface water
- NASADEM elevation, HydroSHEDS terrain/hydrology products
- iSDA agricultural demand estimates

Core workflow:
- Data extraction and alignment in `extraction/`
- EDA and missing-value analysis in `notes.ipynb`
- Feature engineering pipeline output in `data/feature_engineered_training_set.csv`
- Model training in `models/` notebooks
- Submission generation in `submission.csv`

## EDA and feature engineering

The analysis path is documented in `notes.ipynb` and follows these steps:

- Confirm coordinate and time alignment across datasets
- Impute/handle missing data (median fill, iterative imputation, missing flags)
- Replace noisy means with median summaries for robustness
- Introduce derived features from raw variables, e.g., rolling statistics, ratios, terrain metrics
- Scale each data source with dataset-specific factors and normalization
- calculate feature importance with shap/RFECV to reduce dimensionality

Feature engineering output files:
- `data/feature_engineered_training_set.csv`
- `data/*_features_training.csv` and `data/*_features_validation.csv`

## Modeling approach

Models in `models/` contain the following baseline and ensemble experiments:

- `models/catboost.ipynb`: CatBoostRegressor with tuned parameters (iterations, depth, learning_rate, l2_leaf_reg, bagging_temperature, random_strength)
  - Reason: handles heterogeneous, partially missing data and nonlinear interactions with minimal preprocessing; strong gradient-boosted tree baseline for tabular features.
- `models/ridge.ipynb`: Ridge regression and linear baseline
  - Reason: provides a simple regularized linear benchmark, interpretable coefficients, and robust behaviour for multicollinearity in high-dimensional geo-features.
- `models/svm.ipynb`: SVR with RBF kernel
  - Reason: models complex non-linear relationships with compact support and tolerance to outliers via epsilon-insensitive loss, useful for continuous water quality targets.
- `models/super_learner.ipynb`: StackingRegressor combining Ridge, CatBoost, SVR with RandomForestRegressor final estimator
  - Reason: ensembles diverse base learners to capture both linear and non-linear structure and reduce generalization error.

Training strategy:
- Read `data/feature_engineered_training_set.csv` and drop non-predictor columns
- Split into train/test holdout (e.g., 70/30)
- Preprocessing pipeline: `SimpleImputer(strategy='median')` + `StandardScaler`
- Multi-target approach: `MultiOutputRegressor` and optionally `RegressorChain`
- Target transformation: `TransformedTargetRegressor` using `np.log1p` and `np.expm1` for skewed distributions
- Evaluate with cross-validation scores and holdout `.score()` (R²) with RMSE metrics

## Development cycle

1. Data extraction: use notebooks in `extraction/` to refresh source features
2. Preprocessing: join feature slices into master training + validation CSVs
3. Feature engineering: create derived features and remove correlated/noisy features
4. Model selection and hyperparameter tuning: BayesSearchCV/Optuna in notebooks with explicit candidate ranges
5. Ensemble: build stacking and multi-output wrappers for robust final predictions
6. Validation and submission: predict on `validation_set.csv` and write `submission.csv`

## Why the project is useful

- Enables reproducible geospatial water quality modeling for research and environmental monitoring
- Supports modular feature-engineering and dataset experiments
- Provides baseline methods for benchmarking in the EY AI and Data Challenge
- Facilitates fast prototyping with Jupyter notebooks and CSV training/validation artifacts

## How to get started

### Prerequisites

- macOS/Linux/Windows
- Python 3.10+
- `git` (to clone repository)

### Install dependencies

1. Clone repository:

```bash
git clone https://github.com/<your-org>/EY-AI-And-Data-Challenge.git
cd EY-AI-And-Data-Challenge
```

2. Create and activate virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate   # macOS/Linux
.venv\Scripts\activate     # Windows
```

3. Install required packages:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Prepare data

The repository includes prepared data under `data/`:

- `training_set.csv`
- `validation_set.csv`
- Feature-specific subsets (e.g., `terraclimate_features_training.csv`)
- Upstream source extractions in `extraction/` notebooks

If you need to refresh or rebuild features, run the corresponding notebook in `extraction/` (e.g., `extraction/terraclimate_Data_Extraction.ipynb`).

### Run model training

Use model notebooks in `models/` for experimentation.

Example with `models/ridge.ipynb`:

```bash
jupyter notebook models/ridge.ipynb
```

Or run Python script from notebook logic (if separated):

```bash
python -m models.ridge
```

### Generate submission

1. Create predictions on `validation_set.csv`.
2. Export to `submission.csv` (see `submission_template.csv` for format).
3. Validate with challenge scoring.

## Project structure

- `data/`: training, validation, engineered features, and templates
- `extraction/`: data extraction notebooks for each source
- `models/`: modeling notebooks (CatBoost, Ridge, SVR, super learner)
- `README.md`: this overview
- `requirements.txt`: Python dependencies

## Where to get help

- Start with this README and the notebooks in `models/` and `extraction/`
- Open issues in repository
- Ask teammates or community in challenge channels
- See upstream docs for major dependencies:
  - scikit-learn: https://scikit-learn.org/
  - CatBoost: https://catboost.ai/
  - xarray/rasterio/geopandas for geospatial data handling

## Licensing

See `LICENSE` for license terms.
