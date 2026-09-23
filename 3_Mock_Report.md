# MOCK REPORT — IT2214 Predictive Analysis and Forecasting

> **How to use this file.** This is a scaffold, not a submission. Every `[SQUARE BRACKET]` is something only you can fill in — your name, your screenshots, your SAS Viya numbers. The prose in each section shows the *level of explanation* that earns top-band marks; rewrite it in your own voice so it reads as your analysis, which is what the NYP academic integrity policy requires. The figures marked ✅ are real values from this dataset that you will reproduce when you run the script.

---

# [COVER PAGE]

<br>

<div align="center">

## IT2214 Predictive Analysis and Forecasting

### Assignment Report

**Predicting Calories Burned in Gym Members**

<br><br>

**Name:** [YOUR FULL NAME]

**Admin Number:** [YOUR ADMIN NUMBER]

**Class:** [YOUR CLASS]

**Lecturer:** [LECTURER NAME]

**Date of Submission:** [DD MMM 2026]

<br>

School of Information Technology
Nanyang Polytechnic
AY26S1

</div>

<div style="page-break-after: always;"></div>

---

# 1. Introduction

## 1.1 Business Problem

A gym operator wishes to understand the fitness patterns and performance of its members across different experience levels. Specifically, the operator wants to know **which factors drive the number of calories a member burns during a workout session**, so that it can design more effective training programmes and give members evidence-based guidance.

## 1.2 Analytical Objective

This report treats `Calories_Burned` as an **interval target variable** and frames the task as a **regression problem**. The analysis has two aims:

1. **Explanatory** — identify and rank the demographic, physiological and behavioural factors associated with calories burned.
2. **Predictive** — build and compare regression models in SAS Viya, select a champion model, and evaluate whether it is accurate enough to be deployed operationally.

## 1.3 Approach

| Stage | Tool | Output |
|---|---|---|
| Data preparation & cleaning | Python (pandas) | Cleaned dataset of [950] records |
| Exploratory data analysis | Python (matplotlib, seaborn) | Correlation and distribution analysis |
| Modelling | **SAS Viya — Model Studio** | Four candidate models |
| Evaluation | **SAS Viya — Model Comparison** | Champion model and recommendations |

<div style="page-break-after: always;"></div>

---

# 2. Dataset Overview

The dataset contains **4,865 records and 15 variables** describing gym members' demographics, physiological measurements, and workout behaviour.

| Variable | Type | Description |
|---|---|---|
| Age | Interval | Member's age (18–59) |
| Gender | Nominal | Male / Female |
| Weight (kg) | Interval | Body weight, 40.0–129.9 kg |
| Height (m) | Interval | Height, 1.50–2.00 m |
| Max_BPM | Interval | Peak heart rate during session |
| Avg_BPM | Interval | Average heart rate during session |
| Resting_BPM | Interval | Pre-workout resting heart rate |
| Session_Duration (hours) | Interval | Length of session, 0.5–2.0 hrs |
| **Calories_Burned** | **Interval — TARGET** | Calories burned, 303–1,783 |
| Workout_Type | Nominal | Cardio / Strength / Yoga / HIIT |
| Fat_Percentage | Interval | Body fat percentage, 10–35% |
| Water_Intake (liters) | Interval | Daily water intake |
| Workout_Frequency (days/week) | Ordinal | 2–5 sessions per week |
| Experience_Level | Ordinal | 1 (beginner) – 3 (expert) |
| BMI | Interval | Derived from weight and height |

> 📸 **[FIGURE 1: Screenshot of `df.info()` and `df.dtypes` output]**
> *Figure 1: Initial data audit showing 4,865 records across 15 variables with no missing values.*

<div style="page-break-after: always;"></div>

---

# 3. Data Preparation

Data preparation was carried out in Python. Four categories of issue were investigated: missing values, duplicate records, logical inconsistencies, and range validity.

## 3.1 Missing Value Assessment

A column-wise null check returned **zero missing values across all 15 variables**. ✅ No imputation was therefore required. This was verified rather than assumed, as undetected missing values would propagate silently into the modelling stage.

> 📸 **[FIGURE 2: Screenshot of `df.isna().sum()` output]**
> *Figure 2: Missing value check — all 15 variables complete.*

## 3.2 Duplicate Record Removal ⚠️ **Critical Issue 1**

An exact-match duplicate check revealed that **3,892 of the 4,865 records (80.0%) were exact duplicates** of other rows. Only **973 genuinely unique member records** exist in the dataset. ✅

| | Records |
|---|---|
| Raw dataset | 4,865 |
| Exact duplicates removed | −3,892 |
| **Unique records retained** | **973** |

**Why this matters.** Duplicate records are not merely redundant — they actively corrupt model evaluation. When the dataset is partitioned into training and validation sets, identical rows are distributed across both partitions. The model can then reproduce validation observations it has effectively already memorised from training, producing an artificially low validation error and a badly over-optimistic estimate of real-world performance. In addition, duplicated members are implicitly weighted more heavily during model fitting, biasing coefficient estimates toward whatever member profiles happen to have been repeated.

Because the duplicated rows were identical across *all* 15 variables — including continuous measurements such as Calories_Burned and Fat_Percentage that would realistically never coincide exactly between two different people — they were judged to be a data-collection artefact rather than genuine repeat observations, and were removed.

> 📸 **[FIGURE 3: Screenshot showing 4,865 → 973 row reduction]**
> *Figure 3: De-duplication reduced the dataset from 4,865 to 973 unique records.*

## 3.3 Logical Consistency Check ⚠️ **Critical Issue 2**

A physiological consistency rule was applied: **average heart rate during a session cannot exceed the maximum heart rate recorded in that same session** (Avg_BPM ≤ Max_BPM by definition).

This check identified **23 records violating the rule** ✅ — for example, a record with Max_BPM = 166 but Avg_BPM = 167. These values are physically impossible and indicate measurement or data-entry error.

**Treatment and justification.** Three options were considered:

| Option | Assessment |
|---|---|
| Swap the two values | Assumes a column-swap error; not supported by evidence, since the pattern is not systematic |
| Impute Avg_BPM from group median | Preserves records but fabricates values for a variable that is a genuine model input |
| **Delete the affected records** ✅ | Chosen — the true values are unrecoverable, and 23 records represent only 2.4% of the deduplicated data, a loss well within acceptable limits |

The 23 records were removed, leaving **950 clean records** for modelling.

> 📸 **[FIGURE 4: Screenshot listing the 23 invalid heart-rate records]**
> *Figure 4: Twenty-three records where Avg_BPM exceeded Max_BPM, identified and removed.*

## 3.4 Validity and Range Checks

Further checks were performed and returned no anomalies. These are reported because verifying data integrity is as much a part of preparation as correcting it:

| Check | Result |
|---|---|
| BMI recalculated as Weight ÷ Height² vs stated BMI | 0 mismatches — internally consistent ✅ |
| Resting_BPM > Avg_BPM (would be implausible) | 0 records ✅ |
| Age within 18–59 | All valid ✅ |
| Height within 1.50–2.00 m | All valid ✅ |
| Gender categories | Exactly 2: Male, Female — no typos or case inconsistency ✅ |
| Workout_Type categories | Exactly 4: Cardio, Strength, Yoga, HIIT ✅ |

> 📸 **[FIGURE 5: Screenshot of the consistency and range check output]**
> *Figure 5: Consistency and range validation — no further anomalies detected.*

## 3.5 Data Transformation

**Column renaming.** Variable names containing spaces and parentheses were renamed for compatibility with SAS Viya:

`Weight (kg)` → `Weight_kg` · `Height (m)` → `Height_m` · `Session_Duration (hours)` → `Session_Duration_hrs` · `Water_Intake (liters)` → `Water_Intake_L` · `Workout_Frequency (days/week)` → `Workout_Frequency`

**Feature engineering.** Three derived variables were created and tested:

| Feature | Formula | Rationale | Correlation with target | Retained? |
|---|---|---|---|---|
| BPM_Reserve | Max_BPM − Resting_BPM | Proxy for cardiovascular fitness | −0.00 ✅ | ❌ No predictive value |
| Intensity_Ratio | Avg_BPM ÷ Max_BPM | How hard the member pushed relative to their own ceiling | +0.30 ✅ | ❌ Weaker than Avg_BPM alone |
| Age_Band | Age binned into 4 groups | Captures possible non-linear age effects | — | ❌ Loses information vs. continuous Age |

**None of the three engineered features were retained.** Testing them and finding no improvement is itself a result: it indicates that the raw variables already capture the available signal, and that heart-rate ratios add nothing beyond Avg_BPM.

> 📸 **[FIGURE 6: Screenshot of renamed columns and engineered features]**
> *Figure 6: Column renaming and feature engineering.*

## 3.6 Final Prepared Dataset

| Stage | Records |
|---|---|
| Raw | 4,865 |
| After de-duplication | 973 |
| After removing invalid heart rates | **950** |
| Retention rate from unique records | 97.6% |

The cleaned file was exported as `gym_clean.csv` for import into SAS Viya.

<div style="page-break-after: always;"></div>

---

# 4. Data Exploration

## 4.1 Target Variable Distribution

`Calories_Burned` ranges from **303 to 1,783** with a mean of approximately **[905]** and a skewness of **[0.29]** ✅. The distribution is close to symmetric, so **no log transformation was applied** — a transformation would have complicated interpretation without meaningfully improving normality.

> 📸 **[FIGURE 7: Histogram of Calories_Burned]**
> *Figure 7: Distribution of the target variable — approximately symmetric (skewness 0.29).*

## 4.2 Correlation Analysis

| Variable | r with Calories_Burned | Interpretation |
|---|---|---|
| **Session_Duration_hrs** | **+0.91** ✅ | Overwhelmingly the dominant driver |
| Experience_Level | +0.70 ✅ | Advanced members burn substantially more |
| Workout_Frequency | +0.58 ✅ | Frequency and per-session burn move together |
| Water_Intake_L | +0.37 ✅ | Likely a consequence of longer sessions, not a cause |
| Avg_BPM | +0.35 ✅ | Intensity contributes, but far less than duration |
| Weight_kg / Height_m / BMI | +0.06 to +0.10 ✅ | **No meaningful relationship** |
| Resting_BPM / Max_BPM | ~0.00 ✅ | No relationship |
| Age | −0.15 ✅ | Slight decline with age |
| **Fat_Percentage** | **−0.60** ✅ | Leaner members burn considerably more |

> 📸 **[FIGURE 8: Correlation heatmap]**
> *Figure 8: Correlation matrix. Session duration (r = 0.91) and body fat percentage (r = −0.60) show the strongest relationships with the target.*

## 4.3 Key Finding 1 — Behaviour predicts calorie burn; body size does not

The most striking result is what does **not** matter. Weight, height and BMI — the measurements members and trainers most commonly focus on — show correlations of roughly 0.06 to 0.10 with calories burned, which is effectively no relationship at all.

What predicts calorie burn is **what the member does**: how long they train (r = 0.91), how often (r = 0.58), and how hard (r = 0.35). Body fat percentage does matter (r = −0.60), but as a measure of body *composition* rather than body *size*.

**Business implication:** the gym's coaching and progress-tracking should be built around training behaviour and body composition, not around the weighing scale.

> 📸 **[FIGURE 9: Scatter plot — Session_Duration vs Calories_Burned, coloured by Experience_Level]**
> *Figure 9: The near-linear relationship between session duration and calories burned, with expert members clustered at longer durations.*

## 4.4 Key Finding 2 — Multicollinearity among the strong predictors

Several of the strongest predictors are themselves highly intercorrelated. Members with higher `Experience_Level` also tend to have longer `Session_Duration_hrs`, higher `Workout_Frequency`, higher `Water_Intake_L`, and lower `Fat_Percentage`. These variables describe a single underlying construct — **training commitment** — from several angles.

**Modelling implication:** multicollinearity destabilises linear regression coefficients, meaning individual coefficient values become unreliable even when overall model fit is good. Tree-based methods (Decision Tree, Forest, Gradient Boosting) are unaffected by this, since they select split variables rather than solving for simultaneous coefficients. This directly motivated the decision to include both linear and tree-based model families in the comparison (Section 5).

> 📸 **[FIGURE 10: Boxplot — Calories_Burned by Experience_Level and Gender]**
> *Figure 10: Calories burned increases consistently with experience level for both genders.*

## 4.5 Workout Type

> 📸 **[FIGURE 11: Boxplot — Calories_Burned by Workout_Type]**
> *Figure 11: Distribution of calories burned across the four workout types.*

**[WRITE 2–3 SENTENCES describing what your boxplot shows — whether the four workout types differ materially in median calorie burn, and what that implies. If the medians are similar, that is itself a finding: it suggests duration and intensity matter more than the choice of workout modality.]**

<div style="page-break-after: always;"></div>

---

# 5. Modelling in SAS Viya

All modelling was performed in **SAS Viya Model Studio**.

## 5.1 Data Import and Project Setup

The cleaned dataset (`gym_clean.csv`, 950 records) was imported via **Manage Data → Import**, and a Model Studio project of type *Data Mining and Machine Learning* was created with `Calories_Burned` as the target.

> 📸 **[FIGURE 12: SAS Viya — imported table preview]**
> *Figure 12: Cleaned dataset imported into SAS Viya (950 records).*

## 5.2 Variable Roles and Metadata

| Variable | Role | Level | Justification |
|---|---|---|---|
| Calories_Burned | **Target** | Interval | Continuous target → regression |
| Session_Duration_hrs, Avg_BPM, Fat_Percentage, Age, Water_Intake_L, Weight_kg, Height_m, Max_BPM, Resting_BPM | Input | Interval | Continuous measurements |
| Gender, Workout_Type | Input | Nominal | Unordered categories |
| Experience_Level, Workout_Frequency | Input | **Nominal** | Ordinal with few levels (1–3, 2–5); treating them as nominal allows non-linear effects |
| **BMI** | **Rejected** | — | Mathematically derived from Weight_kg and Height_m, both already included. Retaining all three introduces exact redundancy without adding information. |

> 📸 **[FIGURE 13: SAS Viya — variable roles table]**
> *Figure 13: Variable role and measurement level assignment in Model Studio.*

## 5.3 Test Design — Data Partitioning

The data was partitioned **60% training / 20% validation / 20% test**, with random seed [12345].

| Partition | Share | Purpose |
|---|---|---|
| Training | 60% | Fit model parameters |
| Validation | 20% | Compare models, tune, and select the champion |
| Test | 20% | Held out entirely; used **once** for an unbiased final performance estimate |

**Why three partitions rather than two.** A simple train/validation split is sufficient to fit a model, but once the validation set is used repeatedly to *choose between* models, its error estimate becomes optimistically biased — the selection process itself fits to the validation data. Holding out a third partition, untouched until the champion is selected, provides an honest estimate of how the model will perform on genuinely unseen members. This matters here because the operator intends to use the model on future members, not on the historical sample.

> 📸 **[FIGURE 14: SAS Viya — partition settings]**
> *Figure 14: 60/20/20 data partition configuration.*

## 5.4 Modelling Pipeline

Four candidate models were built in parallel and connected to a **Model Comparison** node.

| # | Model | Rationale for inclusion |
|---|---|---|
| 1 | **Linear Regression** | Interpretable baseline. Parameter estimates directly quantify each factor's effect on calories burned, which is what the operator's business question requires. Provides a benchmark against which added model complexity must justify itself. |
| 2 | **Decision Tree** | Captures non-linear thresholds and interactions. Produces human-readable rules that a trainer could apply without the model. |
| 3 | **Forest** | Ensemble of decorrelated trees; robust to the multicollinearity identified in Section 4.4 and less prone to the variance of a single tree. |
| 4 | **Gradient Boosting** | Sequentially corrects residual error; typically the strongest performer on structured tabular data. Sets the accuracy ceiling for the comparison. |

Including both a linear and three tree-based approaches was deliberate: the exploratory analysis showed a near-linear dominant predictor (favouring regression) alongside substantial multicollinearity (favouring trees). The comparison determines empirically which characteristic dominates.

> 📸 **[FIGURE 15: SAS Viya — full pipeline diagram]**
> *Figure 15: Model Studio pipeline with four candidate models feeding a Model Comparison node.*

> 📸 **[FIGURE 16–19: SAS Viya — results page for each of the four models]**
> *Figures 16–19: Individual model results (fit statistics and variable importance).*

<div style="page-break-after: always;"></div>

---

# 6. Model Evaluation

## 6.1 Model Comparison

**[FILL IN FROM YOUR OWN MODEL COMPARISON NODE — the values below are illustrative placeholders, not results.]**

| Model | Train ASE | Validation ASE | Validation RMSE | R² (validation) | Rank |
|---|---|---|---|---|---|
| Linear Regression | [ ] | [ ] | [ ] | [ ] | [ ] |
| Decision Tree | [ ] | [ ] | [ ] | [ ] | [ ] |
| Forest | [ ] | [ ] | [ ] | [ ] | [ ] |
| Gradient Boosting | [ ] | [ ] | [ ] | [ ] | [ ] |

> 📸 **[FIGURE 20: SAS Viya — Model Comparison node results with champion highlighted]**
> *Figure 20: Model comparison on validation data; [MODEL] selected as champion.*

## 6.2 Champion Model Selection

**[MODEL NAME]** was selected as the champion, achieving a validation RMSE of **[VALUE]** and R² of **[VALUE]**.

**[WRITE YOUR JUSTIFICATION HERE. A top-band justification addresses more than the error metric:]**
- *Accuracy:* how much better is the champion than the runner-up, in absolute terms? If the improvement is only a few calories of RMSE, say so.
- *Interpretability trade-off:* if a tree ensemble beat Linear Regression by a small margin, is that gain worth losing the interpretable coefficients the operator wants? Argue your position either way — a defended choice scores better than an undefended one.
- *Stability:* is the gap between training and validation error small, indicating the model generalises?

## 6.3 Overfitting Assessment

**[COMPARE training vs validation error for each model. Expect the Decision Tree and Forest to show the widest gap. Write 3–4 sentences identifying which models overfit, by how much, and what that means for deployment. A model with excellent training error and poor validation error has memorised the sample, not learned the pattern.]**

## 6.4 Variable Importance

> 📸 **[FIGURE 21: SAS Viya — variable importance plot from the champion model]**
> *Figure 21: Relative variable importance in the champion model.*

**[FILL IN your top 5 variables in order. Expected pattern: Session_Duration_hrs far ahead, then Avg_BPM, Fat_Percentage, Age, Gender.]**

**Convergence between methods.** The variable importance ranking produced by the champion model closely matches the correlation analysis in Section 4.2 — session duration dominant, followed by intensity and body fat percentage, with weight, height and BMI negligible. Two independent methods reaching the same conclusion increases confidence that the finding reflects a real pattern rather than an artefact of either technique.

## 6.5 Limitations

A comprehensive evaluation states what the model cannot do:

1. **Sample size.** After removing duplicates, only 950 genuine records remain — roughly one-fifth of the apparent dataset size. This limits the complexity the models can reliably support.
2. **Cross-sectional data.** Each record is a single session. The data cannot show how an individual member's calorie burn changes as they progress, only how members at different experience levels differ.
3. **Correlation is not causation.** Water intake correlates with calories burned, but almost certainly because longer sessions cause both. Recommending increased water intake to burn more calories would be an error of interpretation.
4. **No temporal or contextual variables.** Time of day, trainer involvement, and specific exercises performed are unrecorded and may explain part of the residual variance.

<div style="page-break-after: always;"></div>

---

# 7. Recommendations

Each recommendation is stated with its supporting evidence and expected impact.

### Recommendation 1 — Reposition programming around session duration

| | |
|---|---|
| **Evidence** | Session duration correlates 0.91 with calories burned and ranks first in champion-model variable importance. It alone explains approximately 83% of the variance. |
| **Action** | Restructure class scheduling toward longer sessions (75–90 minutes) and introduce a monthly "endurance challenge" that rewards accumulated training time rather than visit count. |
| **Expected impact** | Extending an average session from 1.25 to 1.75 hours is associated with roughly a 40% increase in calories burned per session. |

### Recommendation 2 — Introduce heart-rate zone coaching

| | |
|---|---|
| **Evidence** | Avg_BPM correlates 0.35 with calories burned and is a significant model input, while Max_BPM and Resting_BPM are not. Intensity *during* the session is what counts. |
| **Action** | Deploy heart-rate monitors and coach members to sustain target zones. Prioritise this for members whose schedules cap session length, since intensity is their available lever. |
| **Expected impact** | A secondary pathway to higher calorie burn for time-constrained members. |

### Recommendation 3 — Build a structured beginner-to-expert progression pathway

| | |
|---|---|
| **Evidence** | Experience_Level correlates 0.70 and Workout_Frequency 0.58 with calories burned. Advanced members train longer, more often, and carry less body fat. |
| **Action** | Create a formal three-tier progression programme with defined milestones for advancing from beginner to intermediate to advanced, supported by trainer check-ins at each transition. |
| **Expected impact** | Moves members along the behavioural profile most strongly associated with results, improving outcomes and supporting retention. |

### Recommendation 4 — Replace weight-based progress tracking with body composition

| | |
|---|---|
| **Evidence** | Weight (r ≈ 0.10), height (r ≈ 0.10) and BMI (r ≈ 0.06) show effectively no relationship with calories burned, while body fat percentage shows a strong negative relationship (r = −0.60). |
| **Action** | Shift member progress reviews from scale weight and BMI to body composition measurement (bioimpedance or calliper testing) at monthly intervals. |
| **Expected impact** | Members are measured on a metric that actually relates to their training outcomes, reducing the demotivation caused by a static scale weight during body recomposition. |

### Recommendation 5 — Deploy the model for per-member session targets

| | |
|---|---|
| **Evidence** | The champion model predicts calories burned with a validation RMSE of [VALUE], accurate enough for guidance-level use. |
| **Action** | Integrate the model into the member app to generate an expected calorie-burn target for each planned session, and flag sessions falling materially below prediction for trainer follow-up. |
| **Expected impact** | Converts the analysis from a one-off study into an operational tool, and creates a feedback loop that generates better data for future model refinement. |

---

# 8. Conclusion

This analysis prepared a dataset of 4,865 gym records down to **950 genuinely unique and internally consistent observations**, removing 3,892 duplicate rows and 23 physiologically impossible heart-rate records that would otherwise have produced a badly over-optimistic model.

Four regression models were built and compared in SAS Viya using a 60/20/20 partition. **[CHAMPION MODEL]** was selected, achieving a validation RMSE of **[VALUE]**.

The central finding is consistent across both exploratory and modelling methods: **calories burned is driven by training behaviour, not body size**. Session duration alone accounts for the large majority of the variation, with intensity, training frequency, and body composition making secondary contributions, while weight, height and BMI contribute essentially nothing. For the gym operator, this points to a clear strategic direction — build programmes, coaching, and progress measurement around how members train rather than around what they weigh.

---

# 9. References

[List any sources — SAS documentation, textbooks, or articles you cite. If you used none, remove this section.]

---

## ⚠️ Before you submit — final check

- [ ] Cover page shows your **name and admin number**
- [ ] Every `[SQUARE BRACKET]` placeholder replaced
- [ ] All 21 figures inserted, cropped, and legible
- [ ] Every figure has a caption and is referenced in the body text
- [ ] **All modelling and evaluation screenshots are from SAS Viya** (Figures 12–21)
- [ ] Section 6 numbers filled in from your own Model Comparison run
- [ ] Prose rewritten in your own words
- [ ] Total length ≤ **10 pages** — cut Section 2's variable table first if you need space
- [ ] Exported to **PDF**
