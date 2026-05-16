# Mind vs Clock — Analysis Pipeline
## AUA DS 299 Capstone | Anzhelika Simonyan | Supervisor: Arman Asryan

---

## Setup Instructions (PyCharm)

### Step 1 — Install Python
Make sure you have Python 3.10 or higher installed.
Download from: https://www.python.org/downloads/

### Step 2 — Open project in PyCharm
1. Open PyCharm
2. File → Open → select this folder (mindvsclock_analysis)

### Step 3 — Create a virtual environment
In PyCharm terminal (bottom of screen):
```
python -m venv venv
```
Then activate it:
- Windows:  `venv\Scripts\activate`
- Mac/Linux: `source venv/bin/activate`

### Step 4 — Install all required libraries
```
pip install -r requirements.txt
```

### Step 5 — Add your data file
Place your Excel file in this folder:
```
Mind_vs_Clock_Full_Dataset.xlsx
```

---

## Run Order

Run the scripts in this exact order:

```
python step1_cleaning.py
python step2_features.py
python step3_hmm.py
python step4_regression.py    ← coming next
```

Each script reads from the output/ folder and writes to the output/ folder.
The output/ folder is created automatically on first run.

---

## Output Files

| File | Created by | Description |
|---|---|---|
| output/clean_participants.csv | step1 | Verified participant data |
| output/clean_trials.csv | step1 | Main trial rows only, cleaned |
| output/exclusion_log.csv | step1 | Who was excluded and why |
| output/featured_trials.csv | step2 | Trials with 5 behavioral features |
| output/hmm_results.csv | step3 | Per-participant HMM results |
| output/hmm_states.csv | step3 | Per-trial state labels |
| output/hmm_visualization.png | step3 | 10-panel results figure |
| output/regression_results.csv | step4 | Prediction model results |
| output/regression_visualization.png | step4 | Regression figures |

---

## What Each Script Does

**step1_cleaning.py**
Removes practice trials, flags impossibly fast responses (<200ms),
excludes participants with >20% timeout rate, incomplete sessions,
or zero choice variation.

**step2_features.py**
Computes 5 behavioral features per trial:
- rt_norm: normalized response time (z-score within participant)
- rt_trend: rolling 5-trial slope (is the person speeding up?)
- choice_entropy: rolling 10-trial choice consistency
- perf_delta_norm: rolling performance vs personal baseline
- timeout_flag: binary cognitive overload indicator

**step3_hmm.py**
Fits a 2-state Gaussian Hidden Markov Model per participant.
Labels states as Analytical (System 2) or Intuitive (System 1).
Detects strategy shift points — the exact trial where state changes.
Produces a 10-panel visualization.

**step4_regression.py** (coming next)
Mixed-effects logistic regression to predict shift probability
from time pressure, complexity, trial number, and demographics.

---

## Troubleshooting

**"ModuleNotFoundError: No module named hmmlearn"**
→ Run: pip install hmmlearn

**"FileNotFoundError: output/clean_trials.csv"**
→ You need to run step1_cleaning.py first

**"Model is not converging" warnings in step3**
→ This is normal and expected with short sequences (40 trials).
  The model still produces valid results. Ignore these warnings.

**Visualization not showing (only saving to file)**
→ This is correct — the scripts save PNG files to the output/ folder.
  Open output/hmm_visualization.png to view the results.

---

## Libraries Used

| Library | Purpose |
|---|---|
| pandas | Data manipulation |
| numpy | Numerical computing |
| scipy | Statistics (linear regression for RT slope) |
| hmmlearn | Hidden Markov Model |
| matplotlib | Visualization |
| seaborn | Heatmap visualization |
| statsmodels | Mixed-effects regression (step 4) |
| scikit-learn | Data preprocessing |
| openpyxl | Reading Excel files |
