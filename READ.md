# EGFR Binding Affinity ($pKi$) Prediction Pipeline
**Team ID:** MM2621  
**Submission File:** `MM2621_MM25ML01.csv`

---

## 1. Executive Summary
This repository contains the machine learning framework developed by Team **MM2621** to predict compound binding affinity ($pKi$) for the EGFR kinase target. Addressing high dimensionality ($1,033$ molecular descriptors across $276$ small-molecule training samples), our solution combines rigorous feature screening with a **6-Model High-Diversity Weighted Ensemble** (`VotingRegressor`). 

The pipeline achieves strong cross-validation stability, reaching an Out-of-Fold **Pearson $R = 0.8149$** and **$R^2 = 0.6388$**.

---

## 2. Pipeline Workflow & Methodology
A. Preprocessing & Feature Screening
1. **Variance Thresholding:** Filtered uninformative and near-constant features (`threshold=0.01`), reducing descriptor space from $1,033$ to $559$.
2. **Median Imputation:** Filled missing descriptor values using `SimpleImputer(strategy='median')` to preserve distribution medians without leaking target information.
3. **Feature Screening:** Applied `SelectFromModel` with a `LGBMRegressor` estimator to isolate the top **69 highly predictive non-redundant molecular features** (fingerprint bits, aromatic ring density, hydrogen-bond donors/acceptors).

### B. Champion Ensemble Architecture
To minimize model variance and prevent overfitting on small sample sizes ($N = 276$), we engineered a balanced ensemble (`VotingRegressor`) combining gradient boosters ($80\%$ total weight) and bagging trees ($20\%$ total weight):

| Model | Hyperparameter Highlights | Weight |
| :--- | :--- | :--- |
| **XGBoost** (`XGBRegressor`) | `max_depth=3`, `learning_rate=0.018`, `reg_alpha=1.2`, `reg_lambda=1.5` | $0.25$ |
| **LightGBM** (`LGBMRegressor`) | `max_depth=3`, `num_leaves=7`, `learning_rate=0.02`, `reg_alpha=1.0` | $0.25$ |
| **CatBoost** (`CatBoostRegressor`) | `depth=3`, `learning_rate=0.02`, `l2_leaf_reg=5.0` | $0.20$ |
| **HistGradientBoosting** | `max_depth=3`, `learning_rate=0.02`, `l2_regularization=1.5` | $0.10$ |
| **Random Forest** | `max_depth=5`, `max_features=0.20`, `n_estimators=350` | $0.10$ |
| **Extra Trees** | `max_depth=5`, `max_features=0.20`, `n_estimators=350` | $0.10$ |

---

## 3. Validation Strategy & Results

All models were evaluated using 5-Fold Cross-Validation (`KFold(n_splits=5, shuffle=True, random_state=42)`).

### Iterative Progression Table

| Model Iteration | Mean MAE | Mean RMSE | Mean $R^2$ Score | Mean Pearson $R$ |
| :--- | :--- | :--- | :--- | :--- |
| **Baseline XGBoost** | 0.7999 | 0.9932 | 0.5857 | 0.7822 |
| **3-Model Blend** | 0.7950 | 0.9733 | 0.6034 | 0.7930 |
| **4-Model Optimized Blend** | 0.7889 | 0.9647 | 0.6106 | 0.7977 |
| **Screened 5-Model Ensemble** | 0.7762 | 0.9481 | 0.6238 | 0.8066 |
| **Screened 6-Model Ensemble (Champion)** | **0.7665** | **0.9297** | **0.6388** | **0.8149** |

---

## 4. Key Takeaways & Interpretability

* **Feature Noise Reduction:** Screening descriptors down from $1,033$ to $69$ provided the largest jump in $R^2$ (+0.038) and Pearson $R$ (+0.014) by eliminating redundant structural descriptors.
* **Model Diversity:** Combining tree-bagging algorithms (Random Forest & Extra Trees) with regularized shallow gradient boosters (depth = 3) prevented over-predictions on high-$pKi$ compounds.

---

## 5. Submission Compliance
* **File Name:** `MM2621_MM25ML01.csv`
* **Rows:** 71 test compounds (+ 1 header row)
* **Required Columns:** `BindingDB MonomerID`, `Predicted pKi`
* **Null Check:** 0 missing values
Once pasted, click the green Commit changes... button at the top right.

