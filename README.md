# Phage Lifestyle Prediction (Temperate vs Lytic) — Semi-Supervised ML

This project predicts **phage lifestyle** (temperate vs non-temperate/lytic) from engineered proteomic features using a **semi-supervised pipeline**:

1) trained on the small labeled set  
2) expanded labels with **Label Spreading** (high-confidence pseudo-labels)  
3) retrained supervised models on the expanded training set  
4) evaluated once on a held-out ground-truth test set

---

## Project Overview

### Goal
Classify phages as:
- **yes** = temperate  
- **no** = non-temperate 

### What’s in the data (high level)
- `phage_id`: unique identifier per phage
- `temperate`: ground truth label when available (`yes`/`no`), otherwise missing
- ~thousands of engineered features (binary and numeric) derived from annotations (e.g., functional labels / domains / counts)

In our run:
- **337 labeled** samples
- **6819 unlabeled** samples
- **1569 total features** before feature selection

---

## Method (What we did)

### 1) Preprocessing
- **Data cleaning** (type coercion, handling missing values)
- **Feature aggregation** (multiple feature sources combined into a single matrix keyed by `phage_id`)
- **Normalization / scaling** (StandardScaler where needed for PCA/SVM/LabelSpreading)
- **Zero-variance filtering** (remove features with no information)
- 
### 2) Train/Validation/Test split (ground truth only)
We split **only the labeled (337)** samples using **stratification** to preserve the yes/no ratio:
- 70% train
- 15% validation
- 15% test

The **test set is never used** for choosing thresholds, features, or hyperparameters.

### 3) Feature selection
We used a **correlation-based filter** (computed on train), and chose the threshold using validation performance.
This produced a reduced feature set (`best_feats`) used consistently downstream.

### 4) Semi-supervised Label Spreading (pseudo-labeling)
We fit `LabelSpreading(kernel="knn")` on:
- labeled training subset **+** unlabeled pool  
then kept pseudo-labels above a confidence threshold (`thr`, e.g. 0.8).

Example outcome from our run:
- Validation BAC (LabelSpreading): **~0.889**
- Unlabeled total: **6819**
- Pseudo-labeled kept (thr=0.8): **5113**
  - predicted **no**: 3274
  - predicted **yes**: 1839

### 5) Supervised models
We trained and tuned (train + CV, checked on validation), then retrained on (train+val) and tested once on test:
- Decision Tree (DT)
- Random Forest (RF)
- SVM (SVC)
- XGBoost (XGBClassifier)

Metrics reported consistently:
- Best CV Balanced Accuracy (BAC)
- Validation BAC (using best estimator from the search)
- Test BAC (final report metric)
- Confusion matrix + classification report

---

## Repository Contents

- `src.ipynb` — full Colab-style notebook containing:
  - preprocessing
  - feature selection
  - label spreading + confidence thresholding
  - supervised training + evaluation
  - UMAP visualizations (before/after pseudo-labeling)

- `README.md` — you’re reading it 🙂
- labels.csv

> Note: data files are not committed to the repo. See “Data” below.

---

## Data

Source: Open-source dataset used in the course project from Yukgehnaish et al. : https://sid.erda.dk/cgi-sid/ls.py?share_id=bW2RmLjL6A

Expected CSVs:
- `Phage_lifestyle_info.csv`
- `Final_phage_lifestyle_df.csv`
- `Final_training_df_concat.csv`
- `Final_validation_df_concat.csv`

A manually curated labels.csv file was created by cross-referencing RefSeq accessions from the Phage_lifestyle_info.csv with the GenBank accessions in Final_training_df_concat.csv

Data link on google drive:
https://drive.google.com/drive/folders/1tynProOOrxh0KxqiFJ7vIuTzqgIKSKLJ?usp=sharing

## How to Run

### Google Colab
1. Upload `src.ipynb` to Colab
2. Mount Google Drive (if using Drive)
3. Set your working directory / data paths
4. Run cells top-to-bottom
