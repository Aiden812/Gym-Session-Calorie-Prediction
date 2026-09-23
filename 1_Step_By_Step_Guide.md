# IT2214 Assignment — Step-by-Step Guide

**Deadline:** 16 Aug 2026, 2359 hrs · **Today:** 2 Aug 2026 · **You have 14 days.**

---

## What you are being marked on

| Rubric row | Weight | Full marks means | Where it comes from |
|---|---|---|---|
| Data Prep | 5% | "Comprehensive data preparation" | Section 3 of report + screenshots |
| Modelling | 10% | "Comprehensive test design and considerations" | SAS Viya only — 4 models + partition |
| Evaluation | 10% | "Comprehensive evaluation and recommendation" | Model comparison + business recommendations |
| Report | 5% | "Well-organized with ideas clearly conveyed" | Structure, captions, flow |
| Presentation | 10% | "Well-organized with ideas clearly conveyed" | Live SAS Viya demo, face on camera |
| AI4I certificate | 5% | Submitted | Free marks — do it first |

**The single biggest differentiator in this dataset:** it contains two deliberate data-quality traps. Finding and fixing both is what separates "simple data preparation" (3 marks) from "comprehensive" (5 marks) — and it gives you the story to lead your report and video with.

---

# DAY 1 (today) — Lock in the free 5%

### Step 1.1 — Register for the AI4I course
1. Go to **learn.aisingapore.org** and search for **"Literacy in AI"** (AI4I programme).
2. Register using the **exact name on your NYP records** — the rubric requires the certificate to clearly show your registered name.
3. Complete the modules (self-paced, roughly 2–4 hours).
4. Download the certificate as **PDF**. Save it as `AI4I_Certificate_<YourName>.pdf`.

✅ **Done = 5% secured.** Do not leave this to the last week.

### Step 1.2 — Confirm your SAS Viya access
1. Log in to the school's SAS Viya environment.
2. Open **Model Studio** and confirm you can create a new project.
3. If anything fails, email your lecturer **today**. Access problems in week 3 will cost you the assignment.

---

# DAYS 2–3 — Data Preparation (5%)

Use Python in Jupyter. I've written the complete script for you: **`2_data_prep_script.py`** (attached). Run it one cell at a time so each output is a clean screenshot.

### Step 2.1 — Audit the raw data
Run Cell 1. You will see:
- **4,865 rows, 15 columns**
- **0 missing values** — say this explicitly in your report, don't just skip it
- **3,892 exact duplicate rows** ← trap #1

### Step 2.2 — Fix trap #1: duplicate records
Run Cell 3.

| | Rows |
|---|---|
| Raw dataset | 4,865 |
| Duplicates removed | −3,892 |
| Unique records | **973** |

**Screenshot this.** Then write 2–3 sentences in your report explaining *why* it matters:
> Duplicate records cause identical observations to appear in both the training and validation partitions. The model can then "memorise" rather than generalise, producing artificially low validation error and an over-optimistic view of model performance.

That "why" sentence is worth more marks than the code itself.

### Step 2.3 — Fix trap #2: impossible heart rates
Run Cell 4. **23 records have Avg_BPM greater than Max_BPM** — physically impossible, since the average heart rate during a session cannot exceed the maximum recorded in that same session.

You have three defensible options. Pick one and **justify it in writing**:

| Option | When to argue for it |
|---|---|
| **Drop the 23 rows** (recommended) | Only 2.4% of data; the true values are unknowable |
| Swap the two values | If you argue it's a data-entry column swap |
| Impute Avg_BPM from the group median | If you want to preserve every record |

The script drops them → **950 clean records remain.**

### Step 2.4 — Run the validity checks that pass
Run Cell 5. These all come back clean, but **showing that you checked is itself evidence of thoroughness**:
- Recalculated BMI matches the BMI column (0 mismatches)
- No records where Resting_BPM > Avg_BPM
- All ranges plausible: Age 18–59, Height 1.50–2.00 m, Resting_BPM 50–74
- Gender has exactly 2 valid values; Workout_Type has exactly 4

### Step 2.5 — Rename columns for SAS Viya
Run Cell 6. Column names with spaces and parentheses cause friction in Viya:

`Weight (kg)` → `Weight_kg` · `Session_Duration (hours)` → `Session_Duration_hrs` · `Water_Intake (liters)` → `Water_Intake_L` · `Workout_Frequency (days/week)` → `Workout_Frequency`

### Step 2.6 — Engineer 3 new features (optional, scores well)
Run Cell 7:
- `BPM_Reserve` = Max_BPM − Resting_BPM (cardiovascular fitness proxy)
- `Intensity_Ratio` = Avg_BPM / Max_BPM (how hard they pushed)
- `Age_Band` = 18-29 / 30-39 / 40-49 / 50-59

⚠️ Be honest about the result: `BPM_Reserve` correlates ~0.00 with the target and `Intensity_Ratio` only 0.30. **Report that they added little value.** Saying "I engineered these features, tested them, and they did not improve the model" demonstrates better analytical judgement than pretending they worked.

### Step 2.7 — Export
Run Cell 11 → produces `gym_clean.csv` with **950 rows**. This is your SAS Viya input.

---

# DAY 4 — Data Exploration

Run Cells 8–10. These are your report's Section 4.

### Step 3.1 — Correlation with Calories_Burned

| Variable | r | Read as |
|---|---|---|
| **Session_Duration_hrs** | **+0.91** | Overwhelmingly the dominant driver |
| Experience_Level | +0.70 | Experts burn more |
| Workout_Frequency | +0.58 | More sessions/week → more per session |
| Water_Intake_L | +0.37 | Proxy for session length, not a cause |
| Avg_BPM | +0.35 | Intensity matters, but less than duration |
| **Fat_Percentage** | **−0.60** | Leaner members burn more |
| Age | −0.15 | Mild decline with age |
| Weight_kg / Height_m / BMI | ~0.06–0.10 | **Essentially no relationship** |
| Max_BPM / Resting_BPM | ~0.00 | No relationship |

### Step 3.2 — The two insights to build your whole report around
1. **Body size does not predict calorie burn — behaviour does.** BMI, weight and height are all near-zero. What matters is how long and how hard the member trains. This is counter-intuitive and makes an excellent talking point.
2. **The predictors are heavily intercorrelated.** Experienced members train longer, more often, drink more water, and carry less body fat. Flag this multicollinearity and note that it inflates linear-regression coefficient instability, while tree-based models handle it without difficulty. **Mentioning this puts you in the top band.**

### Step 3.3 — Caption every chart
Under every single figure, write one or two sentences of interpretation. A chart with no commentary reads as "simple" analysis. A chart with a stated finding reads as "comprehensive."

---

# DAYS 5–8 — Modelling in SAS Viya (10%)

⚠️ **Every modelling and evaluation screenshot must come from SAS Viya.** Python outputs are only acceptable for the data prep and exploration sections.

### Step 4.1 — Import the data
1. SAS Viya home → **Manage Data**
2. **Import** → Local File → upload `gym_clean.csv`
3. Click **Import Item**, then confirm the table loaded (950 rows)
4. 📸 Screenshot the imported table preview

### Step 4.2 — Create the project
1. Open **Build Models** (Model Studio)
2. **New Project** → Name: `IT2214_Calories_Prediction`
3. Type: **Data Mining and Machine Learning**
4. Data source: your imported `GYM_CLEAN` table
5. Click **Save**

### Step 4.3 — Set variable roles (the Data tab)
| Variable | Role | Level |
|---|---|---|
| Calories_Burned | **Target** | Interval |
| Session_Duration_hrs, Avg_BPM, Fat_Percentage, Age, Water_Intake_L, Weight_kg, Height_m, Max_BPM, Resting_BPM | Input | Interval |
| Gender, Workout_Type | Input | Nominal |
| Experience_Level, Workout_Frequency | Input | **Nominal** (they are ordinal, 1–3 and 2–5) |
| BMI | **Rejected** | — |

**Why reject BMI:** it is mathematically derived from Weight and Height, both already in the model. Including all three is redundant. State this reasoning in your report — it's exactly the kind of "consideration" the rubric asks for.

📸 Screenshot the variable roles table.

### Step 4.4 — Set the partition (this is where marks live)
1. Project **Settings** → **Partition Data**
2. Set **Training 60% / Validation 20% / Test 20%**
3. Note the **random seed** (default 12345) so your work is reproducible

> A 70/30 split is acceptable, but a three-way split scores better because it lets you hold out the test set entirely and touch it only once, for the final champion comparison. Explain that logic in the report.

📸 Screenshot the partition settings.

### Step 4.5 — Build the pipeline
On the **Pipelines** tab, right-click nodes to add these in parallel off the Data node:

| # | Node | Why you included it (write this in the report) |
|---|---|---|
| 1 | **Linear Regression** | Interpretable baseline. Coefficients directly quantify each factor's effect — this is what the gym operator actually wants to know. |
| 2 | **Decision Tree** | Captures non-linear thresholds; produces rules a trainer could read. |
| 3 | **Forest** | Ensemble; robust to the multicollinearity found in exploration. |
| 4 | **Gradient Boosting** | Usually the strongest performer; sets the accuracy benchmark. |

Then add a **Model Comparison** node and connect all four into it.

Optional extras that impress: a **Variable Selection** node ahead of the regression, and a **Neural Network** as a fifth model.

📸 Screenshot the full pipeline diagram.

### Step 4.6 — Run and capture
1. Click **Run Pipeline** (wait for all nodes to complete)
2. For **each** model, right-click → **Results** and screenshot:
   - Fit statistics (train vs validation ASE/RMSE)
   - Variable importance plot
   - For Linear Regression: the coefficient/parameter estimates table
   - For Decision Tree: the tree diagram

---

# DAY 9 — Evaluation (10%)

### Step 5.1 — Compare the models
Open the **Model Comparison** node results. Build this table in your report:

| Model | Train ASE | Valid ASE | Valid RMSE | R² | Champion? |
|---|---|---|---|---|---|
| Linear Regression | | | | | |
| Decision Tree | | | | | |
| Forest | | | | | |
| Gradient Boosting | | | | | |

*(Fill in from your own run — numbers vary with the random seed.)*

### Step 5.2 — Say three things about the results
1. **Which model won, and why** — not just "lowest error." Mention the accuracy/interpretability trade-off. If Gradient Boosting wins by a small margin over Linear Regression, argue whether the gain is worth losing interpretability.
2. **Overfitting** — compare each model's training error to its validation error. A large gap (typically the Decision Tree or Forest) indicates overfitting. Point directly at it.
3. **Variable importance** — expect `Session_Duration_hrs` far ahead of everything else, then `Avg_BPM`, `Fat_Percentage`, `Age`, `Gender`. Confirm this matches your correlation analysis. Consistency between exploration and modelling is a sign of a sound analysis — say so.

### Step 5.3 — Write recommendations for the gym operator (not for your lecturer)
Every recommendation must be traceable to a finding. Use this pattern:

> **Finding →** Session duration correlates 0.91 with calories burned and is the top-ranked variable in the champion model.
> **Recommendation →** Restructure class scheduling toward 75–90 minute sessions and introduce a "long session" challenge, rather than promoting more frequent short visits.
> **Expected impact →** Extending an average session from 1.25 to 1.75 hours is associated with roughly a 40% increase in calories burned per session.

Write **five** recommendations in that format. Suggested themes:
1. Session duration as the primary lever
2. Heart-rate-zone / HIIT coaching to raise intensity (Avg_BPM)
3. A structured beginner → expert progression pathway (Experience_Level, Workout_Frequency)
4. Track body composition, not weight or BMI, since weight/BMI showed no relationship to calorie burn
5. Deploy the model in the member app to set per-session calorie targets and flag under-performing sessions for trainer follow-up

---

# DAYS 10–12 — Write the report (5%)

Use the attached **`3_Mock_Report.md`** as your scaffold. Max 10 pages.

**Page budget:**
| Section | Pages |
|---|---|
| Cover page (name + admin number) | 0.5 |
| 1. Introduction & objective | 0.5 |
| 2. Dataset overview | 0.5 |
| 3. Data preparation | 2.0 |
| 4. Data exploration | 2.0 |
| 5. Modelling in SAS Viya | 2.0 |
| 6. Model evaluation | 1.5 |
| 7. Recommendations & conclusion | 1.0 |

**Report checklist:**
- [ ] Every screenshot is cropped and legible at 100% zoom
- [ ] Every figure has a caption: "Figure 3: De-duplication reduced the dataset from 4,865 to 973 records."
- [ ] Every figure is referenced in the body text ("As shown in Figure 3, ...")
- [ ] Modelling and evaluation screenshots are **all from SAS Viya**
- [ ] Name and admin number on the cover page
- [ ] Exported to **PDF**, ≤ 10 pages

---

# DAYS 13–14 — Record the video (10%)

**Rules:** max 10 minutes · face clearly visible · **live SAS Viya demo, no report screenshots**.

### Timing plan
| Time | Content | On screen |
|---|---|---|
| 0:00–0:45 | Introduce yourself, state the business problem | Your face / title slide |
| 0:45–2:30 | Data prep — lead with the duplicates and the 23 impossible heart rates | Jupyter notebook |
| 2:30–4:00 | Key exploration findings (0.91 duration, −0.60 fat %, BMI irrelevant) | Your charts |
| 4:00–7:00 | **Live SAS Viya walkthrough** — open Model Studio, show variable roles, partition, pipeline, run results | SAS Viya |
| 7:00–8:30 | Model comparison, champion, variable importance | SAS Viya |
| 8:30–9:45 | Your five recommendations | Slides |
| 9:45–10:00 | Conclusion | Your face |

### Recording tips
1. **Write a script first** and rehearse once with a timer. Most people overrun.
2. Record with **OBS Studio**, **MS Teams** (share screen + camera), or PowerPoint's Record Slideshow. Any of these gives you a picture-in-picture webcam.
3. **Open your SAS Viya project before recording** so it loads instantly — dead air while a pipeline runs wastes your 10 minutes.
4. Do a 30-second test recording first to check that your **audio is audible** and your **face is visible**. Bad audio kills otherwise good presentations.
5. Export as **MP4**.

---

# DAY 14 — Submit

Upload **three separate files** to Brightspace. **Do not zip them.**

- [ ] `IT2214_Report_<AdminNo>.pdf` — max 10 pages
- [ ] `IT2214_Presentation_<AdminNo>.mp4` — max 10 minutes
- [ ] `AI4I_Certificate_<YourName>.pdf`

⚠️ **Late penalty:** ≤5 working days late = capped at 50% of base marks. >5 days = zero. Submit at least a day early — Brightspace upload failures at 2358 are not an accepted excuse.

---

## The 60-second summary

1. **Today:** AI4I certificate + confirm SAS Viya login.
2. **This week:** Run the Python script, find the 3,892 duplicates and 23 impossible heart rates, export `gym_clean.csv`.
3. **Next week:** Four models in one Model Studio pipeline with a 60/20/20 partition and a Model Comparison node. Screenshot everything.
4. **Final days:** Report to PDF, record the video with a live Viya demo, submit three separate files.
5. **Your headline story throughout:** *"Calories burned is driven by behaviour — how long and how hard members train — not by body size. Session duration alone explains 83% of the variance."*
