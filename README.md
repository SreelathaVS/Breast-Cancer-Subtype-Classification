# Comprehensive Analysis of TCGA-BRCA Gene Expression Data for Breast Cancer Subtype Insights

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
