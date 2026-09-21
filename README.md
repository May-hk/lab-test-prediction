# Pre-operative Lab Values & Patient Characteristics: Data Processing and Exploratory Analysis

School project – data cleaning, exploratory data analysis (EDA), and baseline modelling on surgical patient data.

## 1. Project Overview

In this project we look at how **basic patient information** (age, sex, height, weight, BMI, etc.) relates to **pre-operative blood test results**, mainly **hemoglobin (Hb)** and **hematocrit (Hct)**.

We combine two datasets, a clinical dataset (patient/surgery information) and a lab dataset (blood test results over time), clean them, explore them with plots and correlation measures, and finish with a simple baseline model to check whether clinical variables can predict Hct.

Everything is done in one notebook: `data_processing.ipynb`.

## 2. Data

| File | Description |
|------|-------------|
| `datasets/clinical_data.csv` | One row per surgical case. Contains `caseid`, `age`, `sex`, `height`, `weight`, `bmi`, `asa`, `emop`, `department`, `ane_type`, etc. |
| `datasets/lab_data.csv` | Many rows per case. Long format with `caseid`, `dt` (time of the measurement relative to surgery), `name` (test name), `result` (value). |

Both tables are linked by the key **`caseid`**.

> The data files are not included in this repository. Place them in a `datasets/` folder next to the notebook before running.

## 3. What We Did (Step by Step)

### Step 1 – Select the pre-operative lab values
Lab tests are recorded many times per patient. To match clinical practice, we only want the **most recent measurement before surgery**:

- Keep rows where `dt < 0` (before surgery) and `dt >= -43200` (not too far in the past).
- For each `caseid` + test `name`, sort by time and keep the **last** value.
- Checked the `dt` distribution per test to make sure the window looked sensible.

### Step 2 – Reshape (pivot) the lab table
The lab data was in long format (one row per test). We pivoted it into a **wide table**: one row per `caseid` and one column per lab test. All values were converted to numeric (non-numeric entries became `NaN`).

### Step 3 – Handle missing data
Many lab tests are only ordered for some patients. We computed the missing rate of each lab column and **dropped every column with 75% or more missing values**, keeping only tests that are available for enough patients.

### Step 4 – Explore the lab values
- **Box plots** for every remaining lab test to spot outliers and check spread.
- **Histograms** with skewness values to see which distributions are skewed.

### Step 5 – Explore and clean the clinical values
- Histograms with skewness for `age`, `height`, `weight`, `bmi`.
- Cleaning: ages recorded as `">89"` (a privacy-style cap) were replaced with `90` and the column converted to numeric.
- Bar charts for the categorical variables: `sex`, `asa`, `emop`, `department`, `ane_type`.

### Step 6 – Merge clinical and lab data
Joined both tables on `caseid` (inner join), so only patients that have both clinical info and lab results are kept.

### Step 7 – Relationships between clinical variables and Hb / Hct

**Numerical clinical variables** (age, height, weight, BMI):
- **Spearman correlation** with Hb and Hct, shown as a heatmap. Spearman was chosen because it does not assume a linear relationship or normal data.
- Correlations were also computed **separately for each sex**, since Hb/Hct levels typically differ between men and women.
- Scatter plots of height / weight / BMI against Hb and against Hct.
- **Combined-feature plots:** height, weight and BMI are strongly related to each other, so we standardised them (z-score) and used **PCA** to compress pairs (and all three) into a single combined axis (PC1), then plotted that against Hb and Hct.

**Categorical clinical variables** (sex, ASA, emergency operation, department, anaesthesia type):
- Measured with the **correlation ratio (eta, η)**, which ranges from 0 (no association) to 1 (perfect association) and works between a categorical and a continuous variable. Shown as a heatmap.

### Step 8 – Feature engineering
- `sex_bin`: sex encoded as 0/1 (1 = male).
- `age_x_sex`: an **interaction term** (age × sex), because the effect of age on Hct may differ between men and women.

### Step 9 – Baseline models (predicting Hct)
- Kept only rows without missing values in the selected features.
- **Linear regression (OLS)** with 5-fold cross-validation, scored with R².
  - Full model: age, height, weight, BMI, sex, age × sex
  - Single-feature models: age only, height only, weight only, BMI only
  - Full model **without** the interaction term, to see whether the interaction helps
- **Dummy baseline** (always predicts the mean). Its R² should be about 0, which gives us a reference point: any real model must beat this.

## 4. Tools and Libraries

- Python 3
- `pandas`, `numpy` – data handling
- `matplotlib`, `seaborn` – visualisation
- `scikit-learn` – `StandardScaler`, `PCA`, `LinearRegression`, `DummyRegressor`, `cross_val_score`

Install with:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
```

## 5. How to Run

1. Put `clinical_data.csv` and `lab_data.csv` in a folder called `datasets/`.
2. Open the notebook:
   ```bash
   jupyter notebook data_processing.ipynb
   ```
3. Run all cells from top to bottom (the order matters, since later cells reuse `lab_data` and `merged`).

## 6. Project Structure

```
.
├── data_processing.ipynb   # full analysis
├── datasets/
│   ├── clinical_data.csv
|   |── clinical_parameters.csv
|   ├── lab_parameters.csv
|   ├── preop_lab_values.csv
│   └── lab_data.csv
|   
└── README.md
```

## 7. Key Takeaways

<!-- Fill these in with your own results from the notebook outputs -->
- Which lab tests were kept after the 75% missing-data filter: _..._
- Strongest correlations between clinical variables and Hb / Hct: _..._
- Which categorical variables showed the highest eta: _..._
- Mean cross-validated R² of the full model vs. single-feature models vs. dummy baseline: _..._
- Did the age × sex interaction improve the model? _..._

## 8. Limitations and Possible Next Steps

- The analysis is **exploratory**. Correlation does not imply causation.
- Rows with missing values were dropped rather than imputed, which may bias results.
- The baseline is a simple linear model; nonlinear models (e.g. random forest, gradient boosting) or regularised regression could be tried.
- Only Hb and Hct were analysed in depth with basline model.
- Outliers were inspected visually but not removed as there are too many.
