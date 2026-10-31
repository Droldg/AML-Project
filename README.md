# Applied Machine Learning Project – Context

## 1. Project Idea

The project focuses on **migration patterns among young adults across Danish municipalities**, specifically people aged **20–29 years**.

The overall idea is to investigate whether historical migration patterns combined with demographic, housing, income, and employment data can be used to predict how young-adult migration will develop in Danish municipalities.

The project should have a clear and specific **use case**, rather than simply predicting migration for the sake of prediction.

---

## 2. Business Use Case

The primary stakeholder is currently considered to be **Danish municipalities**.

Municipalities need to understand how their ability to attract and retain young adults may develop over time. Changes in young-adult migration can affect future demand for housing, local services, infrastructure, and broader municipal planning.

The ML model is therefore intended as an **early-warning or screening tool**.

It should help municipalities identify whether the development in young-adult migration is likely to improve or deteriorate, so that municipalities can identify areas that may require further investigation or policy attention.

The model is **not intended to explain causality or automatically recommend policy decisions**.

### Current Business Case

> Municipalities need to understand how their ability to attract and retain young adults may develop over time. Changes in young-adult migration can affect future demand for housing, local services, infrastructure, and broader municipal planning.
>
> The project aims to predict whether the net migration rate among 20–29-year-olds in Danish municipalities is likely to increase or decrease in the following year. By combining historical migration data with demographic, housing, income, and employment characteristics, the project will assess whether machine learning can improve predictions compared with a simple historical baseline.
>
> The model could be used as an early-warning or screening tool, helping municipalities identify areas where the development in young-adult migration may require further investigation or policy attention.

---

## 3. Current ML Question

The current preferred formulation is:

> **Can historical migration, demographic, housing, income, and employment characteristics predict whether the net migration rate among 20–29-year-olds in a Danish municipality will increase or decrease in the following year?**

This would make the main task a **binary classification problem**:

\[
Target =
\begin{cases}
1 & \text{Net migration rate increases next year} \\
0 & \text{Net migration rate decreases next year}
\end{cases}
\]

The change can be calculated as:

\[
Change_{t+1}
=
NetMigrationRate_{t+1}
-
NetMigrationRate_t
\]

### Alternative still possible

An alternative is to treat the project as a **regression problem** and predict:

\[
NetMigrationRate_{t+1}
\]

directly.

The classification approach currently appears to fit the business use case particularly well because municipalities receive a simple indication of whether the development is expected to move in a positive or negative direction.

This decision should be finalized before the modeling stage.

---

## 4. Unit of Analysis

The intended observation level is:

\[
Municipality \times Year
\]

Example:

| Municipality | Year | Migration | Demographics | Housing | Income | Employment | Target |
|---|---:|---:|---:|---:|---:|---:|---|
| Copenhagen | 2020 | ... | ... | ... | ... | ... | Increase |
| Roskilde | 2020 | ... | ... | ... | ... | ... | Decrease |

The intention is therefore **not** to predict the behavior of individual people.

---

## 5. Target Population

The project currently focuses on:

> **Young adults aged 20–29**

This group was selected because migration among young adults is particularly relevant for settlement patterns, education, employment, housing demand, and municipal development.

---

# 6. Planned Data Sources

The current plan is to combine approximately **five datasets from Statistics Denmark / StatBank**.

## FLY66 – Internal migration between municipalities

**Purpose:** Core migration dataset.

Provides:

- in-migration municipality
- out-migration municipality
- age
- sex
- year
- number of people moving

We focus on ages **20–29**.

From FLY66 we intend to calculate:

- `inflow_20_29`
- `outflow_20_29`
- `net_migration_20_29`

where:

\[
NetMigration = Inflow - Outflow
\]

---

## BEFOLK3 – Population

**Purpose:** Population and demographic characteristics.

Potential features:

- population aged 20–29
- total population
- share of population aged 20–29
- population growth

It is also needed to create a comparable **net migration rate**:

\[
NetMigrationRate =
\frac{NetMigration_{20-29}}
{Population_{20-29}}
\times 1000
\]

Using a rate is preferable to raw migration counts because municipalities differ greatly in population size.

---

## BOL101 – Housing

**Purpose:** Describe the housing structure of each municipality.

Potential features:

- rental housing share
- owner-occupied housing share
- apartment share
- housing stock

The objective is to test whether housing characteristics contain useful predictive information about future young-adult migration.

---

## INDKP101 – Income

**Purpose:** Describe economic characteristics.

Potential feature:

- average disposable income

Only a small number of relevant income variables should be selected rather than including every available income measure.

---

## RAS200 – Employment

**Purpose:** Describe local labour-market conditions.

Potential feature:

- employment rate

This provides another socioeconomic dimension that may improve predictions.

---

# 7. Expected Final Dataset

The five sources will ultimately be merged into **one ML dataset**.

Conceptually:

```text
municipality
year
inflow_20_29
outflow_20_29
net_migration_20_29
net_migration_rate
young_population_share
rental_share
income
employment_rate
target_next_year
```

Each row represents one municipality in one year.

---

# 8. Time Period

A likely period is approximately:

> **2010–2025**

because this provides reasonable overlap between the selected Statistics Denmark datasets.

The intended predictive structure is:

\[
X_t \rightarrow Y_{t+1}
\]

Example:

```text
Data from 2020 → predict migration development in 2021
Data from 2021 → predict migration development in 2022
...
```

This is important to avoid **data leakage**.

---

# 9. Geographic Scope

The original idea is to use **Danish municipalities**.

A possible alternative discussed is to focus only on municipalities located geographically on **Zealand (Sjælland)**.

If this is done, it should be justified through the business case rather than simply to reduce the dataset.

For example:

> A municipality-focused or regional planning use case concerning young-adult settlement patterns on Zealand.

Technically, Statistics Denmark's API allows us to specify exactly which municipality codes should be retrieved.

No final decision has been made yet between:

- all Danish municipalities, or
- municipalities on Zealand.

Using all municipalities provides more observations for machine learning.

---

# 10. Data Collection Progress

## Manual FLY66 test

A manual CSV was downloaded for:

- destination: Copenhagen
- year: 2025
- age: 20–29
- sex: men and women
- origins: all municipalities

The CSV was successfully loaded and cleaned in Python.

The total inflow of 20–29-year-olds to Copenhagen in 2025 was:

\[
24,858
\]

---

## StatBank API test

A separate notebook, `api_test.ipynb`, was created.

The StatBank API was successfully used to:

1. retrieve FLY66 metadata,
2. identify variable codes,
3. retrieve the same Copenhagen 2025 observations automatically.

Important codes identified include:

```text
TILKOMMUNE = in-migration municipality
FRAKOMMUNE = out-migration municipality
ALDER = age
KØN = sex
Tid = year
```

For the test:

```text
Copenhagen = 101
Age = 20–29
Men = M
Women = K
Year = 2025
```

The API returned:

\[
24,858
\]

which exactly matched the manually downloaded CSV.

Therefore:

\[
Manual\ CSV = API\ result
\]

The API approach has been validated and can now be scaled to more municipalities and years.

---

# 11. Planned ML Approach

The initial plan is to compare several models rather than immediately selecting one complex algorithm.

Possible models include:

- Baseline model
- Logistic Regression
- Decision Tree
- Random Forest
- potentially KNN

If the project instead uses regression:

- historical baseline
- Linear Regression
- Decision Tree Regressor
- Random Forest Regressor

A key part of the project is not simply whether an ML model performs well, but:

> **Does machine learning provide better predictions than a simple historical forecast?**

---

# 12. Baseline

A simple baseline should be included.

For example, the baseline might assume that next year's development follows the recent historical pattern.

The ML models should then be evaluated against this baseline.

This makes it possible to answer:

> Does adding demographic, housing and socioeconomic information actually improve prediction?

---

# 13. Possible Feature-Set Comparison

A useful analytical structure could be to gradually add information.

### Model A
Historical migration only.

### Model B
Historical migration + demographics.

### Model C
Historical migration + demographics + housing.

### Model D
Historical migration + demographics + housing + income + employment.

This would allow us to investigate:

> **Does additional contextual information improve predictive performance beyond historical migration alone?**

---

# 14. Model Evaluation

The evaluation metrics depend on the final target.

### If classification is used

Potential metrics:

- Accuracy
- Precision
- Recall
- F1-score
- Confusion Matrix
- ROC-AUC

### If regression is used

Potential metrics:

- MAE
- RMSE
- potentially \(R^2\)

Evaluation should use a **temporal train/test structure** rather than relying only on a random split.

Earlier years should generally be used to predict later years.

---

# 15. Model Inspection

After creating the predictive model, the project should also investigate which variables contribute to predictions.

Examples:

- previous migration
- rental housing share
- employment rate
- income
- young population share

This can be investigated through model inspection and feature importance methods.

However:

> **Predictive importance does not imply causality.**

If rental housing is an important predictor, we may conclude:

> Rental housing share is associated with and useful for predicting migration development.

We cannot automatically conclude:

> Increasing rental housing causes young adults to move to the municipality.

This distinction will be important when interpreting the model.

---

# 16. Prediction vs. “Why”

The project can answer questions such as:

> Which municipalities are predicted to experience increasing or decreasing young-adult net migration?

and:

> Which variables are most useful for making those predictions?

The project cannot directly answer:

> Why did young people move to this municipality?

Answering causal “why” questions would require a different research design.

The predictive model can instead identify patterns that may warrant further investigation.

---

# 17. Ethics and Fairness

Because the model operates at municipality level rather than making decisions about individuals, many individual-level fairness concerns are less direct.

However, fairness can still be relevant.

For example, the model could perform:

- very accurately for large urban municipalities,
- but poorly for small or rural municipalities.

This could create a **quality-of-service disparity** if municipalities use the model for decision support.

Later error analysis could therefore compare model performance across different types or sizes of municipalities.

---

# 18. Current Overall Direction

The current recommended project structure is:

\[
Historical\ data_t
\]

↓

\[
Migration + Demographics + Housing + Income + Employment
\]

↓

\[
Machine\ Learning
\]

↓

\[
Prediction\ of\ development_{t+1}
\]

↓

\[
Increase / Decrease\ in\ young-adult\ net\ migration
\]

↓

\[
Municipal\ early-warning / screening\ tool
\]

The objective is therefore not simply to create the most accurate model possible.

The objective is to determine whether machine learning can provide **useful and actionable predictive information for municipalities** about future young-adult settlement patterns.

---

# 19. Immediate Next Steps

1. Finalize whether the target should be **classification (increase/decrease)** or **regression (next year's migration rate)**.
2. Decide whether the geographic scope should be **all Danish municipalities or Zealand only**.
3. Scale the validated FLY66 API extraction to the selected municipalities and years.
4. Retrieve BEFOLK3 and calculate the net migration rate.
5. Add BOL101, INDKP101 and RAS200.
6. Merge the datasets into one municipality-year panel.
7. Perform EDA and inspect missing values.
8. Create the next-year target without data leakage.
9. Establish a simple baseline.
10. Train and compare ML models.
11. Perform model inspection and error analysis.
12. Interpret results in relation to the municipal use case.

---

## Working Principle

Throughout the project, the main question should remain:

> **How could a municipality actually use this prediction?**

If a feature, model, dataset, or analysis does not contribute meaningfully to answering that question, it should probably not be included.
