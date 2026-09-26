# Kcat
# CatPred-kcat: Machine Learning Prediction of Enzyme Catalytic Activity

## Overview

This repository contains the computational workflow for predicting enzyme catalytic turnover rates (**kcat**) using machine learning.

The project uses the **CatPred-DB** dataset and develops an interpretable regression framework based on substrate molecular descriptors and experimental conditions. The workflow includes data preprocessing, molecular feature engineering, model development, feature-importance analysis, ablation studies, protein-disjoint evaluation, applicability-domain analysis, and final held-out testing.

The target variable is **log10(kcat)**, allowing the model to operate on the highly skewed distribution of experimentally measured catalytic rates.

---

## Project Objectives

The main objectives of this project are to:

* Develop a machine-learning model for predicting enzyme kcat.
* Represent substrates using interpretable molecular descriptors.
* Incorporate experimentally relevant conditions such as temperature and pH.
* Compare machine-learning models for regression.
* Identify important predictive features.
* Evaluate the contribution of protein, substrate, and experimental feature groups.
* Assess generalization using a **protein-disjoint evaluation strategy**.
* Investigate sequence overlap, substrate overlap, chemical similarity, and applicability domain.
* Characterize prediction errors across different kcat ranges.
* Establish a final locked model for reproducible prediction.

---

## Dataset

## Dataset

The project uses the **CatPred-DB** dataset, which contains processed datasets for enzyme kinetic parameter prediction. The dataset is publicly available through Zenodo.

### Dataset Access

The complete dataset can be accessed and downloaded directly from the official Zenodo record:

**[CatPred-DB Dataset — Zenodo](https://zenodo.org/records/14775076)**

**DOI:** `10.5281/zenodo.14775076`

The Zenodo record provides the `catpred-db.tar.gz` archive containing the processed CatPred-DB datasets.

> **Note:** The dataset is not included directly in this GitHub repository because of its large size. Users should download it from the official Zenodo repository before running the notebook.

The original CatPred Kcat data are divided into:

| Split      | Samples |
| ---------- | ------: |
| Training   |  18,751 |
| Validation |   2,084 |
| Test       |   2,316 |

The dataset contains enzyme sequences, UniProt identifiers, substrate/reaction information, experimentally measured kcat values, temperature, pH, EC information, taxonomy, sequence clusters, and related metadata.

The raw kcat values span several orders of magnitude, making a logarithmic transformation appropriate.

### Target

The prediction target is:

```text
log10(kcat)
```

rather than raw kcat.


### Target

The prediction target is:

```text
log10(kcat)
```

rather than raw kcat.

---

# Feature Engineering

## Protein Features

The initial feature set incorporates sequence-derived biochemical characteristics, including sequence length, amino-acid composition, and physicochemical composition-related features.

These features were used during the broader model-development and protein-disjoint analyses.

## Substrate Features

Substrate structures are represented using **RDKit** molecular descriptors calculated from SMILES representations.

The molecular descriptors include:

* Molecular weight
* LogP
* Topological polar surface area (TPSA)
* Hydrogen-bond donors (HBD)
* Hydrogen-bond acceptors (HBA)
* Rotatable bonds
* Ring count
* Heavy-atom count

Example:

```python
Descriptors.MolWt(mol)
Crippen.MolLogP(mol)
Descriptors.TPSA(mol)
Lipinski.NumHDonors(mol)
Lipinski.NumHAcceptors(mol)
Lipinski.NumRotatableBonds(mol)
```

## Experimental Features

The workflow also incorporates:

* Temperature
* pH

---

# Model Development Workflow

The project follows a staged modeling strategy:

```text
CatPred-DB
    │
    ▼
Data preprocessing and quality control
    │
    ▼
log10(kcat) target construction
    │
    ▼
Protein + substrate + experimental features
    │
    ▼
Initial 38-feature model
    │
    ├── Model comparison
    ├── Feature importance
    ├── Permutation importance
    ├── Feature-group ablation
    └── Residual/range analysis
    │
    ▼
Protein-disjoint evaluation
    │
    ├── Sequence-overlap analysis
    ├── Substrate-overlap analysis
    ├── Chemical similarity
    ├── Applicability domain
    └── Distance/error analysis
    │
    ▼
Feature reduction
    │
    ▼
Final 10-feature model
    │
    ▼
ExtraTreesRegressor
    │
    ▼
Held-out protein-disjoint test
```

---

# Machine-Learning Model

The principal final model is an **ExtraTreesRegressor**.

The final locked model uses:

```text
n_estimators      = 200
min_samples_leaf  = 4
min_samples_split = 2
max_features      = 1.0
max_depth         = None
random_state      = 42
```

The model is trained on the protein-disjoint training set.

---

# Final Feature Set

The final locked model uses 10 features:

| Feature           | Category     |
| ----------------- | ------------ |
| `mol_weight`      | Substrate    |
| `logp`            | Substrate    |
| `tpsa`            | Substrate    |
| `hbd`             | Substrate    |
| `hba`             | Substrate    |
| `rotatable_bonds` | Substrate    |
| `rings`           | Substrate    |
| `heavy_atoms`     | Substrate    |
| `temperature`     | Experimental |
| `ph`              | Experimental |

Thus, the final locked predictor uses **substrate molecular properties and experimental conditions** rather than protein sequence descriptors.

---

# Protein-Disjoint Evaluation

A major component of the project is evaluation under protein-disjoint conditions.

The protein-disjoint dataset contains:

| Split         | Samples |
| ------------- | ------: |
| Training      |  12,390 |
| Validation    |   2,376 |
| Held-out Test |   3,985 |

Preprocessing is performed without allowing information from validation or test sets to influence training preprocessing.

For example, missing-value imputation is fitted on the training data and subsequently applied to validation and test data.

---

# Final Held-Out Performance

The final locked 10-feature ExtraTrees model was evaluated on **3,985 protein-disjoint held-out samples**.

| Metric          |       Result |
| --------------- | -----------: |
| R²              |     0.064264 |
| RMSE            |     1.709281 |
| MAE             |     1.310406 |
| Pearson r       |     0.296529 |
| Pearson p-value | 1.06 × 10⁻⁸¹ |

These values correspond specifically to the final protein-disjoint held-out evaluation.

---

# Feature Importance

The final model's feature importance analysis identified the following relative importance values:

| Feature          | Importance |
| ---------------- | ---------: |
| pH               |   0.120034 |
| LogP             |   0.116178 |
| Temperature      |   0.114590 |
| Rotatable bonds  |   0.104843 |
| TPSA             |   0.101779 |
| HBD              |   0.095901 |
| HBA              |   0.090566 |
| Molecular weight |   0.089780 |
| Heavy atoms      |   0.083856 |
| Rings            |   0.082472 |

Feature importance is used as an interpretation tool rather than as evidence of causal relationships.

---

# Model Interpretation

Several complementary analyses are included in the workflow.

### Permutation Importance

Permutation importance is used to estimate how much model performance changes when individual features are randomly permuted.

### Feature-Group Ablation

Feature groups are removed or combined to investigate the contribution of:

* Protein/sequence information
* Substrate information
* Experimental conditions

### Residual Analysis

Prediction errors are examined to identify systematic deviations between observed and predicted kcat values.

### kcat-Range Analysis

Model behavior is evaluated across low-, medium-, and high-kcat regions.

The final model shows prediction-range compression:

* Low kcat values tend to be overpredicted.
* High kcat values tend to be underpredicted.
* Predictions are more concentrated around the middle of the target distribution.

---

# Applicability-Domain Analysis

The project evaluates whether predictions are being made for samples similar to those encountered during training.

The analysis includes:

* Sequence novelty
* Sequence-cluster overlap
* Substrate overlap
* Molecular similarity
* Nearest-training distances
* Error across applicability-domain groups

This provides additional information about when the model can be expected to generalize and where prediction uncertainty/error may increase.

---

# Reproducibility

The final model and associated artifacts are saved in:

```text
FINAL_LOCKED_MODEL/
```

Important outputs include:

```text
FINAL_LOCKED_10FEATURE_PROTEIN_DISJOINT_EXTRATREES.pkl
FINAL_LOCKED_10FEATURE_IMPUTER.pkl
FINAL_LOCKED_10FEATURE_NAMES.pkl
FINAL_LOCKED_10FEATURE_HELDOUT_TEST_RESULTS.csv
FINAL_LOCKED_10FEATURE_HELDOUT_TEST_PREDICTIONS.csv
FINAL_LOCKED_10FEATURE_MODEL_MANIFEST.txt
```

The final model can therefore be reused without retraining.

---

# Repository Structure

```text
Kcat/
│
├── Untitled2 (6).ipynb
│
├── Figures/
│   ├── model performance figures
│   ├── feature importance
│   ├── permutation importance
│   ├── residual analysis
│   └── applicability-domain analysis
│
└── README.md
```

---

# Software and Libraries

The workflow uses Python-based scientific and machine-learning tools, including:

* Python
* pandas
* NumPy
* scikit-learn
* SciPy
* RDKit
* matplotlib
* seaborn
* joblib/pickle

---

# Key Outputs

The project produces:

* Cleaned datasets
* Engineered molecular descriptors
* Protein-disjoint train/validation/test matrices
* Trained regression models
* Model-performance tables
* Feature-importance analyses
* Permutation-importance analyses
* Feature-group ablation results
* Residual analyses
* kcat-range analyses
* Sequence novelty analyses
* Chemical similarity analyses
* Applicability-domain analyses
* Held-out test predictions
* Final locked model artifacts

---

# Scientific Scope

This project is intended as a **data-driven prediction framework for enzyme catalytic activity**.

The final model should be interpreted as a predictive model of experimentally observed kcat values. Feature importance and association analyses do not establish mechanistic or causal relationships between individual molecular properties and enzyme catalytic efficiency.

---


