# Plant Data Lab — Build Plan

A browser app for middle school groups (3–5 students) to plan an environmental
investigation, collect Arduino sensor data, clean and summarize it, test
predictions with a garden model, and develop solutions. Every action is
recorded for research.

Design canvas (screens): https://claude.ai/artifact/UrJAiYBchdX5d2QWn3aYoW

---

## 1. Decisions made so far

| Topic | Decision |
|---|---|
| Who logs in | Groups log in from **Lesson 7**. |
| Lessons 2–3 | **Plant Profiler** and the **Lesson 2 dashboard** are public pages (no login). |
| Structure | One tab per lesson activity; tab names match the Group Data Journal (01 Start Here, 02 Data Log, 03 Clean, 04 Summarize, Assess & Revise). |
| Pacing | The teacher opens each lesson's tab for all groups. Earlier tabs stay open; later tabs stay locked. |
| Shared work | All members of a group see and edit the same workspace. Every save keeps earlier versions. |
| Sensors | Each group has one Arduino UNO R4 WiFi and chooses 2 variables. |
| Field collection | The Arduino runs the group's plan on its own outside (buttons, LED matrix, SD card, RTC). No internet or Chromebook is needed outdoors. Data is imported over USB-C back in class. |
| Plant database | The teacher uploads an Excel file in the same layout as `Database_1_Final.xlsx`. The app shows the changes and publishes a new version. |
| AI helper | Behavior per lesson is a teacher setting (instructions, what it can see), versioned. Not hard-coded. |
| Units | Temperature in °F (database and Arduino agree). |

## 2. Screens (see canvas)

| Tab | Lesson | Purpose |
|---|---|---|
| Plant Profiler (public) | 2, 3, 7, 10 | Ideal and survivable ranges per plant and variable. |
| Lesson 2 dashboard (public) | 2 | Change one variable, see effects on others (needs the interaction database). |
| 01 Start Here | 7 | Members + roles, pathway (1 plant × 2 locations or 2 plants × 1 location), plants, locations (approved list + group-added), decision, 2 variables with the 3 checks, research question frame, boundaries. |
| Plan | 7 | Collection plan builder: variables, interval, duration, IF/THEN rules → live pseudocode → sent to the Arduino. Reliability rules, reasoning, AI planning check. |
| 02 Data Log | 8–9 | Day × location round blocks. Import session files from the Arduino. Context / problem / solution notes. Quality check, journal, role sign-off. Backup dataset unlockable by teacher. |
| 03 Clean | 10 | One variable at a time. Auto flag (FAIL, outside plausible range, spike), suggested action, group decision + reason. Raw values never edited. |
| 04 Summarize | 10–11 | Below / in / above ideal per location, % in ideal, preliminary decision with trade-off. |
| Visualize | 12 | Graph-ready table, graph builder with required features, interpretation table, AI feedback. *(not drawn yet)* |
| Assess & Revise | 13 | Causal / correlational / comparative predictions tested with the garden model. Assess the model against real data. Revise prediction with reason. |
| Solutions | 14 | Prediction–solution connections, action plan. *(not drawn yet)* |
| Communicate | 15 | Audience, format, planning template, draft. *(not drawn yet)* |
| Teacher dashboard | all | Open lessons, group status, alerts, research export. |
| Teacher settings | all | Plant database, variable interactions, approved locations, cleaning rules, AI helper per lesson, lesson text. |

## 3. Garden model (Lesson 13)

- **Fit per variable:** trapezoid from the Plant Profiler — 0 at or beyond the
  absolute min/max, rising linearly to 1 at the ideal min, 1 across the ideal
  range, falling to 0 at the absolute max.
- **Overall growth fit:** the minimum across variables (law of the minimum).
- **Interactions:** rules from the second (interaction) database, e.g. higher
  air temperature lowers soil moisture. Placeholder rules until it is final.
- **Scenarios:** start from the group's measured data; change variables
  (e.g. +5 °F); choose days ahead and number of runs.
- **Runs:** each run adds random day-to-day variation; results show the mean
  and the range across runs.
- **Assessing the model:** compare model output with measured days; list what
  the model leaves out.
- Same database as the Lesson 2 dashboard, different interface and depth.

## 4. Arduino firmware v2 (changes to the existing logger)

Keep: sensor reading code, NPK Modbus reads and retries, PM2.5 warm-up,
calibration constants, one CSV file per session, FAIL rows.

Add:
1. **Plan over USB serial** from the app (sensors, interval, duration, rules,
   location list). Stored in flash so it survives power-off.
2. **Location and Round columns** in each CSV row. Buttons choose the location;
   the LED matrix shows its number.
3. **Auto-stop** after the planned duration.
4. **File transfer over USB**: list session files and send one to the app.
5. **Set RTC from the Chromebook clock** (replaces the two-upload RTC workflow).
6. **Error symbols on the LED matrix** for missing SD card or RTC instead of
   stopping silently (`while(1)`).
7. Rule actions (e.g. show "HOT" when a threshold is crossed).

The app talks to the Arduino with the browser's Web Serial API (Chrome on
Chromebooks supports it over USB-C).

## 5. Research record

Every event is stored with group, member, lesson, tab, timestamp, and the
plant-database and AI-instruction versions in use:

- field edits (with previous value), plan versions and generated pseudocode
- raw imported files (unchanged) and their assignment to day/location
- context / problem / solution notes
- cleaning decisions with reasons
- summaries and preliminary decisions
- model runs (inputs and outputs) and prediction versions with reasons
- AI helper conversations
- role sign-offs and journal entries

Export: CSV and JSON per class, per group, or per event type.

## 6. Technical approach

- **Web app:** Next.js (React), deployed on Vercel.
- **Database, auth, file storage:** Supabase (Postgres). Group workspaces are
  isolated with row-level security.
- **Live shared editing:** Supabase Realtime.
- **Arduino:** Web Serial API in the browser.
- **AI helper:** server-side route calling the chosen model API with the
  lesson's instructions and permitted group context. API keys stay on the
  server.
- **Teacher-editable content:** plant database, interaction rules, locations,
  cleaning rules, lesson text, AI instructions — all stored as versioned data,
  not code.

## 7. Build phases

1. **Foundation:** login, class/groups, teacher lesson control, shared saving
   with version history, research event log, export. Plant Profiler with
   database upload.
2. **Lesson 7:** 01 Start Here, Plan builder with pseudocode. Firmware v2 and
   "Send plan to Arduino".
3. **Lessons 8–9:** Data Log with Arduino import (and CSV file upload as a
   fallback), notes, quality check, journal, backup datasets.
4. **Lessons 10–11:** Clean and Summarize.
5. **Lesson 12:** Visualize.
6. **Lesson 13:** Garden model and Assess & Revise.
7. **Lessons 14–15:** Solutions and Communicate.
8. **AI helper:** panel available from phase 2 with placeholder instructions;
   final behaviors added as they are decided.
9. **Pilot:** classroom dry run with real kits before the spring implementation.

## 8. Open decisions

1. **Login method:** Google sign-in (if the district uses Google Workspace)
   or class code + email + group name.
2. **Soil moisture units:** sensor reports 0–100 % between dry and saturated
   calibration points; database uses volumetric water content (≈9–21 %), and
   Hibiscus uses % of field capacity. Choose: recalibrate, convert ranges, or
   teach as a boundary.
3. **Turbidity scale:** sensor reports classroom-relative NTU* (0–1000);
   database ideal is 0–5 NTU.
4. **NPK:** probe readings vs lab-extraction ranges (Olsen, Mehlich).
5. **Database fixes:** rank columns were converted to dates by Excel.
6. **Sensor limit:** enforce 2 variables (lessons) or allow up to 3 (firmware).
7. **Lesson 3 drafts:** record them (no-login form) or only the Lesson 7 versions.
8. **Interaction database** for the dashboard and model (in progress).
9. **AI agent:** where it runs and the per-lesson behaviors (from Lesson 4).
10. **Lessons 4 and 11** details.
11. Approvals: IRB, district data-privacy review, AI use with minors.

## 9. Files in this repository

- `docs/BUILD_PLAN.md` — this plan
- `data/plant-database-v1.json` — ranges from `Database_1_Final.xlsx` (draft),
  used to seed the Plant Profiler and the model
