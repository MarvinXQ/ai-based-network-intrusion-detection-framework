# Memory-Optimized Enhanced AI‑NIDS with SOTA Comparisons & Deep Error Analysis

A publication‑ready, memory‑efficient framework for Network Intrusion Detection Systems (NIDS) that combines a **two‑step hybrid classifier** with **state‑of‑the‑art baselines** and **comprehensive error analysis**. Designed to run in resource‑constrained environments (e.g., Kaggle kernels) while maintaining high accuracy across multiple benchmark datasets.

---

## Overview

This repository provides an end‑to‑end pipeline for:

1. **Data ingestion & memory optimisation** – loading large CSV/Parquet files, downcasting dtypes, and releasing unused objects.
2. **Leakage‑proof preprocessing** – detection and removal of information‑leaking features *without* losing critical categorical information.
3. **Advanced imbalance handling** – configurable oversampling (SMOTE, BorderlineSMOTE, ADASYN, SMOTE‑ENN, SMOTE‑Tomek) applied **only where beneficial**.
4. **Two‑step classification framework**  
   - **Phase 1** – binary filter (normal vs. attack)  
   - **Phase 2** – one‑vs‑rest specialised Random Forest classifiers for each attack class
5. **SOTA baseline comparison** – XGBoost, Naïve Bayes, DNN, LSTM, CNN‑LSTM evaluated on the same train/test split.
6. **Deep error analysis** – per‑class error rates, confusion patterns, and the most frequent misclassifications.
7. **Reproducibility** – fixed random seeds, stratified splits, and a strict separation of train/test data.

All components are integrated into a single `EnhancedNIDSPipeline` class that can be executed for multiple datasets in a loop.

---

## Key Features (v2.2)

- **Safe categorical encoding**  
  Columns with ≤50 unique values are one‑hot encoded (drop‑first); high‑cardinality columns are frequency‑encoded. This preserves discriminative information while preventing memory blow‑up.

- **Guaranteed normal‑class alignment**  
  The binary filter **always** treats the normal/benign class as class `0`, regardless of the original label ordering. This fixes a subtle bug that previously degraded performance on multi‑class datasets like UNSW‑NB15.

- **Dataset‑specific preprocessing hooks**  
  UNSW‑NB15, NSL‑KDD, and CIC‑ToN‑IoT receive tailored cleaning steps to handle their peculiarities (e.g., protecting the `attack_cat` column, preserving protocol/service/state strings).

- **Memory‑aware resampling**  
  For very large datasets, SMOTE is applied to a random subset to avoid memory exhaustion, while the remaining data is concatenated without duplication.

- **All fixes from previous versions**  
  - `optimize_dataframe_dtypes()` now handles duplicate column names.  
  - `AdvancedImbalanceHandler.fit_resample()` correctly uses `.iloc` for large data.  
  - UNSW‑NB15 selects the original 49‑column file with the `attack_cat` label column.

---

## Architecture

---

## Datasets

The pipeline is configured for five popular NIDS benchmarks:

| Dataset          | Expected Path (Kaggle)                                       | Typical Rows | Key Characteristics            |
|------------------|--------------------------------------------------------------|--------------|--------------------------------|
| NSL‑KDD          | `/kaggle/input/datasets/karthikragavenderb/nsl-kdd-bert`     | 185k         | Classical benchmark, 41 features |
| UNSW‑NB15        | `/kaggle/input/datasets/likkisamarthreddy/unsw-nb15`         | 82k          | Modern attacks, 45 columns      |
| CIC‑IDS2017      | `/kaggle/input/datasets/bertvankeulen/cicids-2017`           | up to 300k   | Extreme imbalance, 78+ features |
| CSE‑CICIDS2018   | `/kaggle/input/datasets/shrey213/cicids2018`                  | up to 300k   | Large‑scale Parquet file        |
| CIC‑ToN‑IoT      | `/kaggle/input/datasets/wahidulislambayazid/cic-ton-iot`     | up to 300k   | IoT‑specific traffic patterns   |

The exact file is automatically selected based on file size and column structure. You can modify the `DATASETS` dictionary to point to your own paths.

---

## Installation

```bash
pip install pandas numpy scikit-learn imbalanced-learn xgboost tensorflow shap psutil tqdm
