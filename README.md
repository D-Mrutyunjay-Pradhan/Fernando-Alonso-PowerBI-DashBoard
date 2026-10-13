# 🏎️ Fernando Alonso Career Analytics

### 21 seasons. 409 races. One dashboard.

An end-to-end data analytics project that turns Fernando Alonso's Formula 1 career records into a cleaned, validated data model, an interactive **Power BI dashboard**, and a set of data-backed findings and recommendations.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150458?logo=pandas&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-measures-0B7A75)

<img width="1277" height="734" alt="Screenshot 2026-10-13 104202" src="https://github.com/user-attachments/assets/bf249c3a-d101-4c08-a20c-1f6cd110f791" />


---

## 📌 Table of Contents
1. [Project Overview](#-project-overview)
2. [Business Problem](#-business-problem)
3. [Headline Results](#-headline-results)
4. [Dataset](#-dataset)
5. [Data Cleaning](#-data-cleaning)
6. [Data Model](#-data-model)
7. [Dashboard](#-dashboard)
8. [Key Insights](#-key-insights)
9. [Recommendations](#-recommendations)
10. [Limitations](#-limitations--data-caveats)
11. [How to Run](#-how-to-run)
12. [Author](#-author)

---

## 🎯 Project Overview

| | |
|---|---|
| **Scope** | 409 Grand Prix starts, 21 seasons (2001–2024), 6 teams |
| **Data** | 7 datasets (raw + cleaned CSVs) |
| **Tools** | Power BI Desktop, DAX, Python (pandas), Excel/CSV |
| **Deliverables** | Cleaned dataset, `.pbix` dashboard, project report (PDF), presentation (PPTX) |

**Questions answered**
- How did Alonso's results evolve across 21 seasons and 6 teams?
- Which teams, circuit types and circuits suited him best and worst?
- How much does starting position determine the result, and how good is his race craft?
- How much did retirements cost, and what caused them?
- Where can performance realistically improve?

---

## 💼 Business Problem

Alonso's career spans 21 seasons, six teams and several scoring eras. Results swing sharply (**12.4 points/race at Ferrari vs 1.6 at McLaren-Honda**), but the raw records could not explain why: retirements were logged as finishes, podium/pole flags didn't match the career record, and no single view linked grid position, reliability, team and circuit.

> **Problem statement:** How can a performance group see, in one trusted view, where points are being lost and which fix is worth the most?

The project follows three steps: **Diagnose** (clean and reconcile data) → **Size** (quantify points lost per lever) → **Act** (turn each lever into an action, KPI and estimated impact).

---

## 🏁 Headline Results

| Races | Wins | Podiums | Poles | Championships | Points | Pts/Race | Fastest Laps | DNFs | DNF Rate |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 409 | 32 | 106 | 22 | 2 | 2,279 | 5.57 | 26 | 67 | 16.4% |

All totals reconcile with Alonso's recognised career record after cleaning.

---

## 🗂️ Dataset

| # | File | Rows | Purpose |
|---|------|-----:|---------|
| 01 | `Driver_Profile` | 1 | Identity, nationality, career span, championships, quote |
| 02 | `Race_Calendar` | 409 | Race, season, round, date, circuit, laps, distance |
| 03 | `Race_Results` | 409 | Grid, finish, laps led, status, win/podium/pole/DNF flags |
| 04 | `Team_Details` | 6 | Team, country, years active, brand colour |
| 05 | `Circuit_Details` | 31 | Location, length, turns, street/high-speed flags |
| 06 | `Career_Milestones` | 14 | Debut, first win, titles, team moves, 100th podium |
| 07 | `Speed_Analysis` | 409 | Average, top and minimum speed per race |

Raw files are kept untouched; each has a `_Clean` counterpart with identical row counts.

---

## 🧹 Data Cleaning

| Issue found in raw data | Fix | Impact |
|---|---|---|
| 47 retirements recorded as "Finished" | Reassigned to the actual retirement reason (Collision, Hydraulics, Gearbox, Engine, Electrical, Suspension, Accident) | DNFs **20 → 67**; DNF rate **4.9% → 16.4%** |
| Podium / pole flags under-counted | Corrected flags | Podiums **99 → 106**, Poles **18 → 22** |
| Speed fields almost empty | Filled with estimates, source relabelled `Ergast F1 Database / Telemetry Estimate` | Speed visuals now populated (illustrative only) |
| Sparse fastest-lap data | Added `Fastest_Lap_Flag` | 26 fastest laps |
| No helper columns | Added `Points_Scored`, `Finish_Position_Num`, `Race_Year`, `Race_Month`, `Season_Label`, `Team_Hex_Color`, `Seasons_Count`, `Circuit_Label`, `Milestone_Year_Label` | Enables slicers, team colours, timeline |

**Validation checks:** no duplicates · row counts identical raw vs clean · wins/podiums consistent with finishing position · positions gained = grid − finish on all 342 classified finishes · all join keys resolve · no classified car with fewer laps than race distance.

---

## 🧩 Data Model

Star-style model with `Race_Results` as the central fact table, plus a dedicated `_Measures` table holding all DAX measures.

<img width="1275" height="730" alt="Screenshot 2026-10-13 104226" src="https://github.com/user-attachments/assets/08ca70a1-ba55-479e-9498-16f8128b2b13" />


```
Career_Milestones ─┐
                   ├─ Driver_Profile ─── Race_Results ─── Team_Details
                   │                          │
                   │                    Race_Calendar ─── Circuit_Details ─── Speed_Analysis
```

| From (1) | To (*) |
|---|---|
| `Driver_Profile[Driver_ID]` | `Race_Results[Driver_ID]` |
| `Driver_Profile[Driver_ID]` | `Career_Milestones[Driver_ID]` |
| `Team_Details[Team_ID]` | `Race_Results[Team_ID]` |
| `Race_Calendar[Race_ID]` | `Race_Results[Race_ID]` |
| `Circuit_Details[Circuit_ID]` | `Race_Calendar[Circuit_ID]` |
| `Circuit_Details[Circuit_ID]` | `Speed_Analysis[Circuit_ID]` |

`Speed_Analysis` connects through `Circuit_Details`, so speed visuals filter by circuit rather than directly by race result.

---

## 📊 Dashboard

One-page dark-themed dashboard (`DashBoard 1`) with **10 KPI cards, 3 slicers**, trend/comparison charts, a season table, a DNF gauge and a milestone timeline.

**Slicers:** Season · Circuit Type · Team

**Visuals**

| Visual | Type | Question answered |
|---|---|---|
| Points, Wins, Podiums & Poles by Season | Combo | How did results move year by year? |
| Wins by Team | Clustered column | Where did victories come from? |
| Total Points by Team | Clustered bar | Which team produced most points? |
| Finishing Position Breakdown | Clustered bar | How often does he win, podium, score, retire? |
| Speed Analysis | Line | Average / top / min speed trend |
| Team Career: Races and Points | Combo | Volume vs return per team |
| Performance by Circuit Type | Clustered bar | Which circuit types give most points/race? |
| Season Performance Overview | Table | Season, team, races, points, wins, podiums |
| DNF Rate | Gauge | Reliability health check |

**Core DAX measures**

```DAX
Total Races    = DISTINCTCOUNT(Race_Results[Race_ID])
Wins           = SUM(Race_Results[Win_Flag])
Podiums        = SUM(Race_Results[Podium_Flag])
Total Points   = SUM(Race_Results[Points_Scored])
DNF            = SUM(Race_Results[DNF_Flag])
Points Per Race = DIVIDE([Total Points], [Total Races])
DNF Rate       = DIVIDE([DNF], [Total Races])
Win Rate       = DIVIDE([Wins], [Total Races])
Podium Rate    = DIVIDE([Podiums], [Total Races])
```

Team charts are coloured via `Team_Details[Team_Hex_Color]` (field-value formatting) so colours stay consistent.

---

## 🔍 Key Insights

1. **The car decides the ceiling.** Ferrari 12.4 pts/race, Renault 4.4 (7.3 in the 2005–06 title years), Aston Martin 4.8, Alpine 3.6, McLaren 2.5.
2. **Qualifying almost fixes the result.** Grid vs finish correlation = **0.88**. All 32 wins came from the first four grid slots; starts from P6 or lower produced only 5 podiums in 260 races.
3. **Front-row starts leak positions.** From P1–P3 he lost 1.24 places on average (49 of 89 races finished behind grid slot), while gaining +1.01 from P11+.
4. **Reliability cost ~300 points.** 67 retirements, 46 mechanical (69%). McLaren-Honda (2015–18) had a 27% DNF rate.
5. **Circuits vary sharply.** Best: Barcelona (9.7 pts/race). Worst: Yas Marina (1.6). High-speed circuits are the weakest type (4.9 pts/race, 22.1% DNF).
6. **2023 proved the model, 2024 stressed it.** 206 points and 8 podiums at Aston Martin in 2023, then 13 points in 2024.
7. **Speed data is not a useful performance explainer** (estimated values, near-zero correlation with results); use as circuit context only.

### Career phases

| Phase | Races | Points | Pts/race | Wins | Podiums | DNF % |
|---|---:|---:|---:|---:|---:|---:|
| Early 2001–04 | 53 | 112 | 2.11 | 1 | 5 | 24.5 |
| Renault 2005–06 | 37 | 270 | 7.30 | 14 | 30 | 10.8 |
| McLaren/Renault 07–09 | 52 | 193 | 3.71 | 7 | 18 | 15.4 |
| Ferrari 2010–14 | 96 | 1192 | 12.42 | 10 | 44 | 9.4 |
| McLaren-Honda 15–18 | 81 | 133 | 1.64 | 0 | 0 | 27.2 |
| Alpine 2021–22 | 44 | 160 | 3.64 | 0 | 1 | 15.9 |
| Aston Martin 2023–24 | 46 | 219 | 4.76 | 0 | 8 | 8.7 |

---

## ✅ Recommendations

| Lever | Action | KPI (baseline → target) | Est. impact |
|---|---|---|---|
| Qualifying | Treat one-lap pace and set-up as a standalone work-stream | Starts in top five: 36% → 42% | ~200 points |
| Front-row conversion | Review first stint and strategy from P1–P3 | Pole-to-win rate: 37% → 50% | ~8 more wins |
| Reliability | Use DNF record as a team-selection criterion; manage components | DNF rate: 16.4% → ≤11% | ~100 points |
| Incident control | Lower-risk opening lap when qualifying outside top 10 | Reduce DNF rate from P11+ (37%) | Protects points |
| Dashboard v2 | Career Phase slicer, grid-vs-finish scatter, DNF by cause, Data Notes panel | Open label/flag issues: 0 | Trusted view |

Combined, the levers are worth roughly **320 points (~+14%)**, lifting points/race from 5.57 to ~6.3. This is an **upper bound**: levers overlap and figures are scenarios, not forecasts.

---

## ⚠️ Limitations & Data Caveats

- **Speed data is mostly estimated** (all top/min speeds and 389 of 409 average/fastest-lap values). Treat speed visuals as illustrative.
- **Pole flag vs grid position:** `Grid_Position = 1` appears in 59 races but only 22 carry the Pole flag. The KPI follows the flag; grid-slot analysis uses `Grid_Position`.
- **Calendar/circuit chronology** does not fully match real history (e.g. circuits appearing in seasons they didn't host), so circuit-level findings are indicative only.
- **Small samples:** 9–17 starts per circuit.
- **Estimated points lost (300)** is a proxy using season-average points from finished races.
- **Correlation ≠ causation:** driver vs car effects can't be separated without teammate data.
- **Scoring systems changed** across eras, so rate metrics are used for comparisons.
- **Missing seasons:** 2002, 2019 and 2020 have no race records.

**Known dashboard label issues:** the Career Span card shows 23 seasons (the data holds 21), and the timeline labels "Returned to McLaren" (2012) and "Returned to F1 – Aston Martin" (2021) are hard-coded and don't match the milestones table (McLaren return is 2015; the 2021 return was with Alpine).

---

## ▶️ How to Run

1. Clone the repo
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   ```
2. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows).
3. Open `Fernando_Alonso.pbix`.
4. If prompted, **Transform data → Data source settings** and repoint the CSV sources to the cleaned CSV files (`01_…_Clean.csv` to `07_…_Clean.csv`).
5. Refresh and use the Season / Circuit Type / Team slicers.

**Sanity check after refresh:** Races 409 · Wins 32 · Podiums 106 · Poles 22 · Fastest Laps 26 · Points 2,279 · DNF 67 · Pts/Race 5.57.

---

## 👤 Author

**D Mrutyunjay Pradhan** — Data Analyst | Portfolio Project

- LinkedIn: `<your-link>`
- GitHub: `<your-link>`
- Email: `<your-email>`

---

*Images, team logos and portraits are not included due to licensing; the dashboard uses an original icon set. Speed values are estimates and should not be presented as verified telemetry.*
