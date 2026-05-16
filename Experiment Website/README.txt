================================================================
Mind vs Clock — Experiment Website
================================================================

AUA DS 299 Capstone Project | 2026
Student:    Anzhelika Simonyan
Supervisor: Arman Asryan, PhD
Institution: American University of Armenia, College of Science
            and Engineering

----------------------------------------------------------------
WHAT THIS FOLDER CONTAINS
----------------------------------------------------------------

index.html
    The complete single-file web experiment. Opens in any modern
    browser (Chrome, Firefox, Safari, Edge). No installation,
    no server, no dependencies — everything is self-contained
    in this one file (HTML + CSS + JavaScript).

SETUP.md
    Step-by-step instructions for connecting the website to a
    Google Sheets backend via Google Apps Script. Read this if
    you want to re-deploy the experiment to collect new data.

README.txt
    This file.

----------------------------------------------------------------
HOW TO VIEW THE EXPERIMENT
----------------------------------------------------------------

OPTION 1 — Open the live deployment (recommended)
    The experiment was deployed during data collection at:
    https://anzhelika-simonyan.github.io/mindvsclock
    (Update this URL if it has changed.)

OPTION 2 — Open the local file
    Double-click index.html. It opens in your default browser
    and runs the full experiment locally. Data submission to
    Google Sheets will only work if the SHEETS_ENDPOINT constant
    in the script is set to an active Apps Script deployment.

----------------------------------------------------------------
EXPERIMENT STRUCTURE
----------------------------------------------------------------

Languages:          English / Armenian (toggle in top-right)
Practice trials:    10 (5 low-pressure + 5 high-pressure)
Main trials:        40 (4 blocks × 10 trials)
Estimated duration: 6-8 minutes

Experimental design (2×2 within-subjects):
    Block 1: low time pressure (12 s)  ×  low complexity
    Block 2: low time pressure (12 s)  ×  high complexity
    Block 3: high time pressure (7 s)  ×  low complexity
    Block 4: high time pressure (7 s)  ×  high complexity

Seven trial types:
    1. Classic Stroop (color-word conflict)
    2. Reverse Stroop
    3. Numerical Stroop (size vs value)
    4. Flanker interference task
    5. Number-word conflict
    6. Cognitive Reflection Test (bat-and-ball items)
    7. Risk gambles (prospect-theory framing)

Per-trial data logged: participant demographics, trial number,
block, time pressure, complexity, time limit, trial type,
choice, response time (ms), timeout flag, points earned,
running total score, timestamp.

----------------------------------------------------------------
DATA STORAGE
----------------------------------------------------------------

When a participant completes the experiment, the website sends
their data via HTTP POST to a Google Apps Script web app, which
appends rows to a Google Sheet with two tabs:

    Participants  — one row per participant (summary)
    Trials        — one row per trial

See SETUP.md for the complete data schema and re-deployment
instructions.

----------------------------------------------------------------
CITATION
----------------------------------------------------------------

If referencing this experiment, please cite the accompanying
IEEE paper:

    Simonyan, A. (2026). Detecting and Predicting Behavioral
    Strategy Shifts in Human Decision-Making Under Changing
    Conditions. American University of Armenia, DS 299
    Capstone Project.
