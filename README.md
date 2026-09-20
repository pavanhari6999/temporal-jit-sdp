# Temporal Robustness and Lightweight Adaptation for Just-In-Time Software Defect Prediction

This repository contains the research artifacts associated with the manuscript:

**Temporal Robustness and Lightweight Adaptation for Just-In-Time Software Defect Prediction**

The study investigates temporal robustness, distribution shift, and lightweight within-project adaptation for Just-In-Time Software Defect Prediction (JIT-SDP) using a leakage-resistant chronological evaluation protocol.

Repository: https://github.com/pavanhari6999/temporal-jit-sdp

---

## Authors

**D. Pavan Hari Krishna**  
M.Tech, Department of Computer Science and Engineering  
KL University, Vaddeswaram, India

**Dr. M. V. D. Prasad**  
Associate Professor, Department of Electronics and Communication Engineering  
KL University, Vaddeswaram, India

---

## Research scope

The study addresses the following questions:

1. How robust is JIT-SDP performance when models are evaluated on chronologically later software changes?
2. Do feature distributions change between historical development data and future observations?
3. Does temporal performance vary across projects and future periods?
4. Do lightweight temporal adaptation strategies improve performance consistently?
5. How sensitive are the results to feature count, class-imbalance treatment, learner choice, and calibration-window size?

The contribution is intentionally **empirical rather than algorithmic**. The study does not claim a new machine-learning algorithm. Instead, it provides a controlled multi-project evaluation of temporal behavior and lightweight adaptation under a common chronological protocol.

---

## Projects and datasets

The experiments cover five JIT-SDP project streams:

- Brackets
- Apache Tomcat
- JGroups
- Spring-Integration
- Apache Camel

The original Brackets dataset contains 11,601 change records and 14 change-level metrics.

The additional project streams are processed JIT-SDP streams used for online/temporal evaluation. Their processing semantics are preserved in the experimental notebook and accompanying documentation.

### Important dataset note

Raw upstream datasets are not necessarily redistributed in this repository when their original licenses or source terms do not permit redistribution. The repository therefore distinguishes between:

- original/upstream datasets,
- processed experimental inputs,
- generated results, and
- executable analysis artifacts.

Users reproducing the experiments should obtain the corresponding upstream datasets from their original sources and follow their licensing conditions.

---

## Main methodology

The experimental protocol follows a future-oriented chronological design.

### 1. Chronological ordering

Records are ordered by their author-date timestamp before model development and evaluation.

Future observations are never moved into the historical training set.

### 2. Historical/future separation

For the original Brackets experiment, a strict chronological 70/30 split produces:

- Historical records: 8,120
- Future records: 3,481

The historical portion is further divided into chronological development and validation data for feature selection and hyperparameter selection.

### 3. Feature selection

Recursive Feature Elimination (RFE) is fitted only using historical development data.

For the controlled Brackets temporal representation, six features are retained:

```text
lt
la
rexp
exp
ld
nuc
```

Feature-selection results are not fitted using future-test labels.

### 4. Primary learner

Random Forest is used as the controlled primary learner for the temporal adaptation experiments.

The primary configuration uses:

- 100 trees
- maximum depth = 5
- minimum samples split = 2
- minimum samples leaf = 4
- max_features = sqrt
- class_weight = balanced
- random_state = 42

Hyperparameter selection uses a controlled chronological validation procedure.

### 5. Temporal evaluation

Future observations are divided into four chronological periods:

```text
T1
T2
T3
T4
```

The frozen model is evaluated without retraining across the future periods.

### 6. Adaptation strategies

The study evaluates:

- Frozen model
- Cumulative retraining
- Sliding window of 2,000 records
- Sliding window of 4,000 records
- Sliding window of 6,000 records
- Sliding window of 8,000 records
- Adaptive thresholding

Cumulative retraining means a new Random Forest is trained from scratch using all labeled observations available at the relevant temporal boundary. No warm-start or incremental parameter update is assumed.

### 7. Adaptive thresholding

The adaptive-threshold experiment uses a 4,000-record labeled window.

Within that window:

- 3,000 records are used for model training.
- 1,000 recent records are used for threshold calibration.
- The threshold is selected from 0.20 to 0.80 in increments of 0.05.
- Future-test labels are not used for threshold selection.

Calibration-window sensitivity is also evaluated.

---

## Main findings

The experiments show substantial project-level and temporal variation.

For the original Brackets future test set, the controlled frozen Random Forest obtains:

| Metric | Value |
|---|---:|
| Accuracy | 0.7236 |
| Precision | 0.4204 |
| Recall | 0.7417 |
| F1-score | 0.5366 |
| ROC-AUC | 0.7961 |
| PR-AUC | 0.5161 |
| MCC | 0.3900 |

Bootstrap confidence intervals are reported in the manuscript and supporting result files.

Across the four additional projects, frozen future-test F1 varies considerably:

| Project | Frozen F1 |
|---|---:|
| Tomcat | 0.4565 |
| JGroups | 0.3145 |
| Spring-Integration | 0.6741 |
| Camel | 0.2975 |

The temporal experiments show that performance can change substantially between T1 and T4 and that the pattern is project-dependent.

The multi-project adaptation analysis covers 16 project-period blocks. The Friedman test detects differences among temporal strategies:

```text
Friedman statistic = 31.4018
p < 0.0001
Kendall's W = 0.3271
```

However, the results do **not** establish one universally superior adaptation strategy.

In particular, the 2,000-record sliding window differs significantly from the frozen strategy in the tested direction of lower F1 after multiplicity-aware comparison.

The repository therefore does not claim universal adaptation superiority.

---

## Distribution-shift analysis

Distribution shift is examined in detail for Brackets using:

- Population Stability Index (PSI)
- Two-sample Kolmogorov-Smirnov testing
- Benjamini-Hochberg correction

The largest observed PSI values among the six selected features include:

| Feature | PSI |
|---|---:|
| exp | 0.2750 |
| rexp | 0.2056 |
| lt | 0.1902 |
| nuc | 0.1560 |
| la | 0.0153 |
| ld | 0.0092 |

The PSI = 0.20 line shown in the manuscript figure is used only as a **reference level for interpretation**. It is not treated as a universal statistical decision threshold.

Distributional change is not interpreted as definitive proof of concept drift.

---

## Sensitivity analyses

The repository includes sensitivity analyses covering:

### RFE feature-count sensitivity

Feature counts from 2 through 14 are evaluated on the chronological Brackets validation set.

The results demonstrate that performance is not uniquely determined by one feature count. The six-feature configuration is retained as the controlled representation used for the main temporal analysis.

### Class imbalance

The primary class-weighted Random Forest is compared with:

- SMOTE
- ADASYN

These experiments are performed without using future-test labels for resampling decisions.

### XGBoost temporal comparison

XGBoost is evaluated as a temporal comparator on Brackets using the same selected feature representation.

### Adaptation-window sensitivity

Sliding windows of:

- 2,000
- 4,000
- 6,000
- 8,000

records are compared.

### Threshold-calibration sensitivity

Calibration windows of:

- 500
- 1,000
- 2,000

records are evaluated.

### Runtime

Runtime per temporal update is reported for the evaluated adaptation strategies. Runtime measurements are hardware- and environment-dependent and are therefore reported as experimental reference values rather than universal performance guarantees.

---

## Baseline positioning

The manuscript discusses several published JIT-SDP baselines and related approaches, including:

- BORB
- ODaSC
- PBSA
- DeepJIT
- CC2Vec
- JITLine

The repository explicitly distinguishes between:

1. experiments executed under the present protocol,
2. controlled baseline-style comparisons, and
3. literature-based positioning.

Published scores from other studies are not presented as if they were reproduced results.

The executed online-baseline-style comparator is also reported separately in the manuscript.

---

## Reproducibility safeguards

The following safeguards are central to the experiments:

1. Chronological ordering is preserved before model development and testing.
2. Future-test labels are not used for model fitting.
3. Future-test labels are not used for adaptive-threshold calibration.
4. Feature selection is fitted only on historical development data.
5. Hyperparameter selection uses chronological validation.
6. Bootstrap confidence intervals use future-test predictions.
7. Adaptation experiments are separated from the frozen baseline.
8. Distribution-shift analysis is separated from predictive-performance evaluation.
9. Literature baselines are clearly distinguished from executed experiments.
10. Random seed 42 is used for the primary Random Forest experiments.

---

## Verified software environment

The principal verified experimental environment includes:

```text
Python 3.13.5
NumPy 2.3.5
pandas 2.2.3
scikit-learn 1.8.0
XGBoost 3.1.3
SciPy 1.17.0
Matplotlib 3.10.8
```

Primary Random Forest experiments use:

```text
random_state = 42
```

Bootstrap confidence intervals use:

```text
2,000 resamples
```

---

## Repository structure

The intended repository organization is:

```text
temporal-jit-sdp/
│
├── README.md
├── CITATION.cff
├── LICENSE
│
├── code/
│   └── Adaptive_JIT_SDP_Research_Stage8C_BORB_ready.ipynb
│
├── data/
│   └── README.md
│
├── results/
│   ├── stage_results/
│   └── supplementary_results/
│
├── figures/
│   ├── Figure_1_Temporal_F1_All_Projects.png
│   ├── Figure_2_Frozen_vs_Adaptive.png
│   ├── Figure_3_Frozen_F1_Project_Comparison.png
│   ├── Figure_4_Adaptation_Strategy_Comparison_UPDATED.png
│   ├── Figure_5_Defect_Rate_Temporal_Change.png
│   ├── Figure_6_Brackets_PSI_Shift.png
│   ├── Figure_7_RFE_Feature_Count_Sensitivity_UPDATED.png
│   └── Figure_9_RF_vs_XGBoost_Temporal.png
│
├── supplementary_figures/
│   ├── Figure_8_Class_Imbalance_Sensitivity.png
│   └── Figure_10_Brackets_Class_Distribution.png
│
├── docs/
│   ├── REPRODUCIBILITY.md
│   ├── MANUSCRIPT_STATUS_V5.md
│   └── RESPONSE_TO_REVIEWERS_V4.md
│
└── manuscript/
    ├── JIT_SDP_Major_Revision_Draft_v5.pdf
    └── JIT_SDP_Major_Revision_Draft_v5.tex
```

The exact files present in a particular repository release may differ slightly depending on whether upstream datasets and manuscript artifacts are redistributed.

---

## Reproducing the experiments

The principal research notebook is:

```text
code/Adaptive_JIT_SDP_Research_Stage8C_BORB_ready.ipynb
```

The notebook contains the experimental workflow for:

- dataset auditing,
- preprocessing,
- chronological splitting,
- RFE,
- Random Forest evaluation,
- temporal segmentation,
- adaptation strategies,
- adaptive thresholding,
- XGBoost comparison,
- class-imbalance sensitivity,
- distribution-shift analysis,
- bootstrap confidence intervals,
- statistical analysis,
- figure generation,
- result consolidation.

### Recommended execution order

1. Obtain the upstream datasets.
2. Place the datasets in the paths expected by the notebook.
3. Run the dataset audit and schema checks.
4. Run preprocessing.
5. Run the historical/future chronological split.
6. Run feature selection and model selection.
7. Run the frozen future evaluation.
8. Run the four-period temporal evaluation.
9. Run adaptation experiments.
10. Run sensitivity analyses.
11. Run statistical analysis.
12. Generate figures.
13. Compare generated outputs with the stored verified results.

Do not bypass the chronological split or use future-test labels for model selection.

---

## Brackets input verification

The verified Brackets input file used in the experiments has the following SHA-256 checksum:

```text
72d3d69c337a7dd7693363e4c368c459878ef989be6de3a0e37b8aa1081e81fc
```

This checksum can be used to verify that the input file corresponds to the dataset used for the reported Brackets experiments.

---

## Data availability

The Brackets dataset is part of publicly available JIT-SDP data used in prior research.

The additional project streams are processed JIT-SDP data used for online/temporal JIT-SDP research.

Because upstream datasets may have their own licensing and redistribution conditions, users should obtain restricted or externally hosted datasets from their original sources and comply with the applicable terms.

The repository focuses on preserving:

- preprocessing decisions,
- experimental code,
- evaluation procedures,
- generated results,
- figures,
- environment information, and
- manuscript artifacts.

---

## Manuscript

The revised manuscript associated with this repository is:

**Temporal Robustness and Lightweight Adaptation for Just-In-Time Software Defect Prediction**

The repository may contain the manuscript PDF and LaTeX source when redistribution is appropriate.

---

## Citation

Please cite the associated manuscript when using the experimental protocol, code, figures, or results.

A DOI should only be claimed after the repository has been archived through an archival service such as Zenodo.

**No DOI is claimed by this repository until an archival DOI has actually been issued.**

---

## Research integrity statement

This repository is intended to support transparent and reproducible empirical software-engineering research.

Reported experimental values correspond to the executed experimental results used in manuscript preparation. Literature results are identified as literature results and are not presented as newly reproduced measurements.

The study's conclusions are deliberately limited to the evaluated projects, datasets, learners, temporal protocol, and adaptation strategies.

---

## Contact

**D. Pavan Hari Krishna**  
M.Tech, Department of Computer Science and Engineering  
KL University, Vaddeswaram, India

For artifact questions, please use the GitHub Issues section of this repository.

---

## License

The code, datasets, figures, and manuscript artifacts may have different licensing conditions depending on their original source.

Before redistributing any upstream dataset, verify its original license and attribution requirements.

For newly authored code in this repository, use the repository license specified by the authors.

