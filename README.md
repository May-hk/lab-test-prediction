# Predicting Pre-operative Haematocrit from Clinical Features

ITCS444 Data Mining, Project 2. Mahidol University, Faculty of ICT.

## Overview

Haematocrit (Hct) is the percentage of blood volume made up of red blood cells. A low pre-operative Hct can indicate anaemia and a higher risk of transfusion. This project tests whether a patient's pre-operative Hct can be predicted from four simple measurements: **BMI, height, weight and sex**.

- **Data:** surgical cases from the open VitalDB dataset (clinical information + laboratory results).
- **Target:** the latest Hct result recorded **within 30 days before surgery**.
- **Patients:** 4,138 after cleaning.
- **Split:** 80% training (3,310) / 20% test (828), `random_state = 42`, the same split for all models.
- **Tuning:** `GridSearchCV` with shuffled 5-fold cross-validation on the training set only. The test set is used once, for the final evaluation.

## Repository structure

```
.
├── datasets/
│   ├── clinical_data.csv          # one row per surgical case (age, sex, height, weight, bmi, ...)
│   ├── clinical_parameters.csv    # description of each clinical column
│   ├── lab_data.csv               # lab results in long format (caseid, dt, name, result)
│   └── lab_parameters.csv         # description of each lab test
├── starting_point.ipynb           # Project 1 starting notebook
├── data_processing.ipynb          # Project 1: cleaning, EDA and baseline model
├── polynomial_regression.ipynb    # Project 2: polynomial ridge regression
├── decision_tree.ipynb            # Project 2: decision tree regression
├── kernel_ridge.ipynb             # Project 2: kernel ridge regression (RBF)
└── README.md
```

## Data preparation (shared by all model notebooks)

1. Keep lab results recorded before surgery and within 30 days (`-2592000 <= dt < 0`).
2. For each case and test, keep the most recent result, then pivot to one row per case.
3. Drop lab columns with 75% or more missing values.
4. Remove impossible values (Hct outside 10–65%, height/weight/BMI ≤ 0) and convert age `">89"` to 90.
5. Merge clinical and lab data on `caseid`, encode sex as `sex_bin` (1 = male), and drop rows missing Hct or any input.

## Results (held-out test set, 828 patients)

| Model | Author | Best settings | Test R² | Test RMSE |
|---|---|---|---|---|
| Mean predictor (baseline) | – | – | 0.000 | 5.33 |
| Linear regression (baseline) | – | – | 0.133 | 4.96 |
| Polynomial ridge regression | P | degree = 2, alpha = 100 | 0.148 | 4.92 |
| Decision tree regression | May | max_depth = 4, min_samples_leaf = 100 | 0.130 | 4.97 |
| Kernel ridge regression (RBF) | Petch | alpha = 1, gamma = 0.0316 | 0.152 | 4.91 |

**Conclusion:** all three models beat the mean baseline but explain only about 13–15% of the variation in Hct, with a typical error of about 5 points. Body size and sex alone are not enough to replace a pre-operative blood test.

## How to run

1. Install Python 3 and the required packages:
   ```bash
   pip install pandas numpy scipy matplotlib seaborn scikit-learn jupyter
   ```
2. Clone the repository. The data files are already included in `datasets/`.
3. Start Jupyter from the repository folder and open one of the model notebooks:
   ```bash
   jupyter notebook
   ```
4. Run all cells from top to bottom (Kernel → Restart & Run All). Each model notebook repeats the data preparation, so it runs on its own.

## Team

| Member | Part |
|---|---|
| P | Polynomial regression |
| May | Decision tree |
| Petch | Kernel ridge regression |
