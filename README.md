# Breast Cancer Subtype Classification

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-F2762E?logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-1.7%2B-red.svg)](https://xgboost.readthedocs.io/)

A bioinformatics and machine learning project analyzing high-dimensional RNA-seq transcriptomic data from **The Cancer Genome Atlas (TCGA-BRCA)** cohort (~1,215 samples, 20,530 genes). This project integrates unsupervised clustering, non-parametric/parametric statistical testing, and supervised classification models to uncover molecular subtype heterogeneity and identify driver biomarkers for breast cancer classification.

---

## 📌 Project Overview

Breast cancer is a heterogeneous disease classified into five main molecular subtypes using the **PAM50** gene signature: **Luminal A (LumA)**, **Luminal B (LumB)**, **HER2-enriched (Her2)**, **Basal-like (Basal)**, and **Normal-like (Normal)**. 

### Key Objectives
* Identify distinct patient clusters in high-dimensional expression space and evaluate alignment with PAM50 subtypes.
* Conduct non-parametric & ANOVA statistical hypothesis testing to identify subtype-specific differential gene expression.
* Train, compare, and hyperparameter-tune machine learning models to accurately classify PAM50 subtypes.
* Extract top feature importances to rank candidate biological biomarkers.

---

## 🛠️ Pipeline & Methodology

```text
TCGA-BRCA Data (RNA-seq + Clinical) 
       │
       ├──► 1. Data Cleaning & Aggregation (N = 615 clean samples)
       ├──► 2. Exploratory Data Analysis & Normality Assessment
       ├──► 3. Dimensionality Reduction (PCA: 40 PCs = 62.35% variance)
       ├──► 4. Unsupervised Clustering (K-Means k=4 vs. Hierarchical vs. DBSCAN)
       ├──► 5. Feature Selection (ANOVA F-Test / SelectKBest: Top 100 Genes)
       ├──► 6. Supervised ML Benchmark & GridSearchCV Tuning (XGBoost)
       └──► 7. Biomarker Extraction & Biological Validation
```
### 1. Data Preprocessing & Cleaning
* **Cohort Size**: Reduced from ~1,215 raw samples to **615 clean patient records** after removing missing values and matching patient IDs across expression, clinical, and survival files.
* **Duplicate Handling Strategy**: Feature-specific rules (numeric $\rightarrow$ mean, categorical $\rightarrow$ mode, tumor stage $\rightarrow$ maximum severity).
* **Scope Refinement**: Restricted feature selection and classification exclusively to **RNA-seq gene expression values** and **PAM50 target labels** to prevent clinical metric bias and overfitting.

### 2. Dimensionality Reduction & Unsupervised Clustering
* **PCA**: Retained **40 Principal Components**, capturing **62.35% cumulative variance**.
* **Optimal $k$ Selection**: Determined via the **Elbow Method** (inertia curve) and **Silhouette Scores** ($\approx 0.15 - 0.25$).
* **Clustering Benchmark (Adjusted Rand Index - ARI)**:
  * **K-Means ($k=4$)**: **ARI = 0.378** *(Best global structure; Basal clearly separated, LumA/LumB overlap)*
  * **Hierarchical Clustering**: **ARI = 0.271**
  * **DBSCAN**: **ARI = 0.000** *(Failed due to high dimensionality and lack of clear density bounds)*

### 3. Hypothesis Testing & Differential Expression
* Shapiro-Wilk testing rejected expression normality across genes ($p < 0.05$, e.g., CPB1: $p = 7.34 \times 10^{-15}$), justifying non-parametric Kruskal-Wallis testing during EDA, while a parametric ANOVA F-test was utilized on continuous expression profiles (gene_exp) to rank top features for ML modeling.
* **ANOVA Subtype Differential**: Evaluated $H_0$ (identical gene expression across subtypes) vs. $H_1$ (significant subtype differences).
* **Results**: Strongly rejected $H_0$ ($p \ll 0.05$). Top differential genes post-FDR correction:
  * **FOXA1**: $p = 3.21 \times 10^{-189}$
  * **ESR1**: $p = 5.26 \times 10^{-187}$
  * **MLPH**: $p = 7.98 \times 10^{-175}$
  * **GATA3**: $p = 1.94 \times 10^{-141}$

---

## 📊 Machine Learning & Results

Using `SelectKBest` with ANOVA F-test (`f_classif`), the top **100 most discriminative genes** were standardized using `StandardScaler` and evaluated on an 80/20 stratified train-test split ($N_{train} = 492$, $N_{test} = 123$).

### Model Comparison Benchmark

| Model | Accuracy | Macro Avg F1 | Weighted Avg F1 | Key Performance Notes |
| :--- | :---: | :---: | :---: | :--- |
| **Untuned XGBoost** | 86.18% | 0.67 | 0.85 | Default baseline parameters |
| **Logistic Regression** | 87.80% | 0.70 | 0.87 | Excellent for LumA and Basal; lower LumB recall |
| **Random Forest** | 87.80% | 0.68 | 0.86 | High LumA recall (0.98); LumB/Her2 confusion |
| **Support Vector Machine (SVM)**| 87.80% | 0.70 | 0.87 | Strong high-dimensional RBF kernel performance |
| **LightGBM** | 87.80% | 0.69 | 0.87 | Perfect Basal recall (1.00); balanced precision |
| **Tuned XGBoost (GridSearchCV)** | **89.43%** | **0.68** | **0.88** | **Best overall model (110/123 correct)** |

### Hyperparameter Tuning (GridSearchCV)
Grid search over 5-fold cross-validation optimized XGBoost:
```python
param_grid = {'learning_rate': 0.1, 'max_depth': 7, 'n_estimators': 200}
```
**Performance Boost:** Accuracy improved from 86.18% $\rightarrow$ 89.43% ($\approx 89.5\%$), achieving the highest overall test performance and 86.2% recall on LumB.
### 🧬 Biological Biomarker InsightsTop gene importances extracted from the tuned gradient boosting model strongly confirmed known breast cancer biology:
**Top Biomarkers Ranking**:
1. MLPH     (Importance: 0.3083)  --> Melanophilin; strong Luminal subtype lineage marker
2. GATA3    (Importance: 0.0412)  --> Luminal cell differentiation master regulator
3. CENPA    (Importance: 0.0336)  --> Centromere protein A; proliferation marker
4. MICALL1  (Importance: 0.0319)  --> Endosomal trafficking regulator
5. ESR1     (Importance: 0.0316)  --> Estrogen Receptor 1; hormone receptor status
6. FOXA1    (Importance: 0.0273)  --> Pioneer factor for estrogen receptor activity
7. FOXC1    (Importance: 0.0209)  --> Key transcriptional driver of Basal-like subtype

## 📊 Comprehensive Model Evaluation & Multi-Metric Comparison

Evaluating models purely on raw **Accuracy** is misleading in cancer transcriptomics due to PAM50 class imbalance (e.g., LumA represents ~49% of the dataset, whereas Normal-like represents <2.5%).. To ensure diagnostic reliability, models were benchmarked across **Precision** (preventing false alarms), **Recall/Sensitivity** (preventing missed aggressive cancers), **Macro F1** (unweighted mean across classes), and **Weighted F1** (sample-size adjusted).

### Multi-Metric Performance Matrix ($N_{test} = 123$)

| Model | Accuracy | Weighted F1 | Macro F1 | Basal Recall ($n=21$) | Her2 Recall ($n=10$) | LumA Recall ($n=60$) | LumB Recall ($n=29$) | Normal Recall ($n=3$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Untuned XGBoost** | 86.18% | 0.85 | 0.67 | 95.2% | 60.0% | 93.3% | 82.8% | 0.0% |
| **Random Forest** | 87.80% | 0.86 | 0.68 | 95.2% | 70.0% | **98.3%** | 75.9% | 0.0% |
| **Logistic Regression** | 87.80% | 0.87 | **0.70** | 95.2% | 70.0% | 91.7% | 89.7% | 0.0% |
| **LightGBM** | 87.80% | 0.87 | 0.69 | **100.0%** | 70.0% | 95.0% | 79.3% | 0.0% |
| **Support Vector Machine (RBF)** | 87.80% | 0.87 | **0.70** | 95.2% | **80.0%** | 93.3% | 82.8% | 0.0% |
| **Tuned XGBoost (GridSearchCV)** | **89.43%** | **0.88** | 0.68 | 95.2% | 60.0% | **98.3%** | **86.2%** | 0.0% |

---

### 🔍 Model-Specific Trade-Offs & Diagnostic Insights

#### 1. Tuned XGBoost (`GridSearchCV`) — Best Overall Classifier
* **Performance**: Achieved **89.43% Accuracy (110/123 correct)** and the highest **Weighted F1-score (0.884)**.
* **Subtype Strengths**: Significantly resolved LumA vs. LumB cross-misclassification, boosting **LumB Recall to 86.2% (25/29)** with 86.2% Precision. Maintained near-perfect **LumA Recall (98.3%, 59/60)** and high **Basal Sensitivity (95.2%, 20/21)**.
* **Clinical Value**: Most balanced model for hormone-receptor-positive breast cancers (LumA/LumB), optimizing treatment selection..

#### 2. Support Vector Machine (SVM - RBF Kernel) — Best for HER2-Enriched Detection
* **Performance**: **87.80% Accuracy**, highest **Macro F1 (0.70)**.
* **Subtype Strengths**: Delivered the highest **HER2 Recall (80.0%, 8/10)** among all models with **88.9% Precision**.
* **Clinical Value**: Essential for identifying candidates for HER2-targeted biological therapies (e.g., Trastuzumab) while minimizing false-positive treatment toxicity..

#### 3. LightGBM — Perfect Sensitivity on Aggressive Basal Subtypes
* **Performance**: **87.80% Accuracy**, **0.87 Weighted F1**.
* **Subtype Strengths**: Achieved **100.0% Basal Recall (21/21 correct)** with **95.5% Precision**.
* **Clinical Value**: Ideal clinical screening tool for aggressive Triple-Negative/Basal tumors, ensuring zero false-negative diagnoses for high-risk patients..

#### 4. Logistic Regression — Strong Linear Baseline
* **Performance**: **87.80% Accuracy**, **0.70 Macro F1**.
* **Subtype Strengths**: Produced high **LumB Recall (89.7%, 26/29)**. However, lower **LumB Precision (76.5%)** indicates linear boundary overlap with LumA profiles.

#### 5. Random Forest — High LumA Precision
* **Performance**: **87.80% Accuracy**, **0.86 Weighted F1**.
* **Subtype Strengths**: Exceptional **LumA Sensitivity (98.3%)**, but suffered lower **LumB Recall (75.9%)** due to decision tree axis-aligned split limits on overlapping hormone profiles.

#### 6. Untuned XGBoost — Default Bottleneck
* **Performance**: Lowest baseline score at **86.18% Accuracy** and **0.85 Weighted F1**.
* **Subtype Weakness**: Lowest **HER2 Recall (60.0%)** and **Basal Precision (83.3%)** prior to hyperparameter tuning, proving that gradient boosting requires parameter optimization (`max_depth=7`, `learning_rate=0.1`) on high-dimensional omics data.

---

### 🧬 Diagnostic Takeaways & Class Imbalance Analysis

1. **Class Imbalance Boundary (Normal Subtype)**: All models yielded **0.0% Recall/Precision on the Normal-like subtype**. This failure was driven by extreme class imbalance ($n=3$ test samples, $2.4\%$ of cohort). Macro F1 ($\sim 0.68 - 0.70$) penalizes models for minority class failures, whereas Weighted F1 ($0.88$) reflects population-level diagnostic accuracy.
2. **Biological Overlap (LumA vs. LumB)**: Cross-misclassification between Luminal A and Luminal B occurs naturally because both subtypes express shared estrogen pathway master regulators (*ESR1*, *GATA3*, *FOXA1*). Hyperparameter tuning on gradient boosted trees (XGBoost) successfully separated these subtle transcriptional differences.

