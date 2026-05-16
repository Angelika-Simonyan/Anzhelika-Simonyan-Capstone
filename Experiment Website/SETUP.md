# Mind vs Clock — Google Sheets Setup

This guide explains how to connect the `index.html` experiment to a Google Sheets backend so that every participant's data is saved automatically when they finish.

The total setup takes about 10 minutes. You only need to do this once.

---

## Overview of the Pipeline

```
  Participant finishes experiment
              │
              ▼
   index.html sends POST request
              │
              ▼
   Google Apps Script (doPost function)
              │
              ▼
   Two tabs appended in your Google Sheet:
     • Participants — one row per session
     • Trials       — one row per trial
```

---

## STEP 1 — Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet.
2. Name it **Mind vs Clock Data**.
3. Rename the first tab to exactly `Participants` (case-sensitive).
4. Create a second tab named exactly `Trials` (case-sensitive).
5. Paste the following column headers into **row 1** of each tab.

### Participants tab — row 1 headers

```
participantId | nickname | age | gender | occupation | field | education | stress | sleep | gaming | handedness | totalScore | completedAt | trialCount
```

### Trials tab — row 1 headers

```
participantId | nickname | age | gender | occupation | field | education | stress | sleep | gaming | handedness | trialNumber | block | timePressure | complexity | timeLimit | trialType | choice | responseTimeMs | timedOut | pointsEarned | totalScoreAfter | timestamp
```

Tip: paste each row of headers into cell A1, then drag the divider between columns to widen them.

---

## STEP 2 — Create the Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**.
2. Delete any existing code in the editor.
3. Paste the entire code block from the **APPS SCRIPT CODE** section at the bottom of this file.
4. Click the floppy-disk **Save** icon.
5. Name the project **MindVsClock**.

---

## STEP 3 — Deploy as a Web App

1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Fill in the deployment settings:
   - **Description:** `Mind vs Clock API`
   - **Execute as:** `Me (your-email@aua.am)`
   - **Who has access:** `Anyone`
     ⚠️ This must be **Anyone**, not "Anyone with the link" or "Anyone at AUA". If access is restricted, participants outside your organization will fail silently and no data will be saved.
4. Click **Deploy**.
5. Authorize the script when prompted. Google will show a warning that the script is unverified — this is normal for personal scripts. Click **Advanced → Go to MindVsClock (unsafe) → Allow**.
6. Copy the **Web app URL** that appears. It looks like:
   ```
   https://script.google.com/macros/s/XXXXXXXXXXXX/exec
   ```

---

## STEP 4 — Connect the URL to the Website

1. Open `index.html` in any text editor (Notepad, VS Code, Sublime, etc.).
2. Search for this line (around line 825):
   ```javascript
   const SHEETS_ENDPOINT = 'YOUR_APPS_SCRIPT_ENDPOINT_HERE';
   ```
   In the current version of the file this should already contain the active URL. Replace it only if you have re-deployed the Apps Script.
3. Save the file.

---

## STEP 5 — Verify End-to-End

1. Open `index.html` in your browser.
2. Complete one full run as a test participant — use the nickname **TestUser**.
3. Open your Google Sheet:
   - **Participants** tab should contain one new row with `TestUser`.
   - **Trials** tab should contain ~40 rows (one per main trial) for that participant.
4. If data appears: setup is complete.

If no data appears after a test run, see the Troubleshooting section below.

---

## Troubleshooting

**Problem: No data appears in the Sheet after completing the experiment.**

Check, in this order:

1. Open the Apps Script editor → click **Executions** in the left sidebar. If you see no recent executions, the website never reached the script. Either the endpoint URL in `index.html` is wrong, or the deployment has expired.
2. Open Deploy → **Manage deployments** and verify **Who has access** is set to **Anyone**. AUA Google accounts often default to organization-only access.
3. If you edited the Apps Script code, you must create a **new deployment** for the changes to take effect. Editing alone does nothing.

**Problem: The page shows "Script function not found: doGet".**

You opened the Apps Script URL directly in a browser. This is expected and not an error. The endpoint only responds to POST requests sent by the experiment website. Ignore this message.

**Problem: Some columns are empty in the Sheet.**

Verify that the row 1 headers in your Sheet match the column names exactly as listed in Step 1, including case and spelling. The Apps Script appends data positionally, so a missing or misspelled header will cause downstream misalignment when you analyze the data.

---

## Data Schema Reference

Each trial row contains the following fields, written by `logTrial()` in `index.html`:

| Field             | Type    | Description                                              |
|-------------------|---------|----------------------------------------------------------|
| participantId     | string  | Auto-generated unique ID per session                     |
| nickname          | string  | Participant-chosen display name                          |
| age               | integer | Self-reported age                                        |
| gender            | string  | Self-reported gender                                     |
| occupation        | string  | e.g., student, employed, unemployed, retired             |
| field             | string  | Field of study or work                                   |
| education         | string  | Highest level of education completed                     |
| stress            | string  | Self-rated stress level (low/medium/high)                |
| sleep             | string  | Self-rated sleep quality                                 |
| gaming            | string  | Gaming frequency (never/occasionally/regularly/daily)    |
| handedness        | string  | Left / right / ambidextrous                              |
| trialNumber       | integer | Global trial index across the session (1-based)          |
| block             | integer | 0 = practice; 1–4 = main blocks                          |
| timePressure      | string  | `low` (12 s) or `high` (7 s)                             |
| complexity        | string  | `low` or `high`                                          |
| timeLimit         | integer | Time window in seconds (12 or 7)                         |
| trialType         | string  | gamble, stroop, reverseStroop, numStroop, flanker, numWord, crt |
| choice            | string  | The option the participant selected                      |
| responseTimeMs    | integer | Reaction time in milliseconds                            |
| timedOut          | boolean | True if no response within the time window               |
| pointsEarned      | integer | Points awarded for this trial                            |
| totalScoreAfter   | integer | Running session score after this trial                   |
| timestamp         | ISO-8601| Server-side completion time of this trial                |

---

## APPS SCRIPT CODE — paste this into the editor

```javascript
// ─────────────────────────────────────────────────────────────
//  Mind vs Clock — Google Apps Script backend
//  Receives POST requests from index.html and writes rows
//  into the Participants and Trials tabs.
// ─────────────────────────────────────────────────────────────

const PARTICIPANT_SHEET = 'Participants';
const TRIALS_SHEET      = 'Trials';

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss   = SpreadsheetApp.getActiveSpreadsheet();

    // ── Save participant summary ──────────────────────────────
    const pSheet = ss.getSheetByName(PARTICIPANT_SHEET);
    pSheet.appendRow([
      data.participantId,
      data.nickname,
      data.age,
      data.gender,
      data.occupation,
      data.field,
      data.education,
      data.stress,
      data.sleep,
      data.gaming,
      data.handedness,
      data.totalScore,
      data.completedAt,
      data.trials ? data.trials.length : 0
    ]);

    // ── Save each trial as its own row ────────────────────────
    const tSheet = ss.getSheetByName(TRIALS_SHEET);
    if (data.trials && data.trials.length > 0) {
      const rows = data.trials.map(t => [
        t.participantId,
        t.nickname,
        t.age,
        t.gender,
        t.occupation,
        t.field,
        t.education,
        t.stress,
        t.sleep,
        t.gaming,
        t.handedness,
        t.trialNumber,
        t.block,
        t.timePressure,
        t.complexity,
        t.timeLimit,
        t.trialType,
        t.choice,
        t.responseTimeMs,
        t.timedOut,
        t.pointsEarned,
        t.totalScoreAfter,
        t.timestamp
      ]);
      tSheet.getRange(tSheet.getLastRow() + 1, 1, rows.length, rows[0].length)
            .setValues(rows);
    }

    return ContentService
      .createTextOutput(JSON.stringify({status: 'ok', saved: data.participantId}))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({status: 'error', message: err.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: run this once from the Apps Script editor to verify
// that the sheet tabs exist and are named correctly.
function testSetup() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const p  = ss.getSheetByName(PARTICIPANT_SHEET);
  const t  = ss.getSheetByName(TRIALS_SHEET);
  Logger.log('Participants tab: ' + (p ? 'found' : 'MISSING'));
  Logger.log('Trials tab: '       + (t ? 'found' : 'MISSING'));
}
```
