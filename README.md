<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=220&section=header&text=CogniStream%20Squad&fontSize=42&fontColor=ffffff&fontAlignY=38&desc=Engineering%20Productivity%20Analytics%20Team&descSize=16&descAlignY=62" width="100%"/>
</div>

<h3 align="center">
  ⚡ <em>The Brilliant Minds Powering CogniStream</em>
</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Team-Core%20Contributors-success?style=for-the-badge&logo=git&logoColor=white"/>
  <img src="https://img.shields.io/badge/Project-CogniStream-blue?style=for-the-badge&logo=powerbi&logoColor=white"/>
</p>

---

<div align="center">

  ### 👥 Team Members

  | Name | Role / Focus |
  | :---: | :---: |
  | **Mameeth C** | 👑 Project Lead & Data Architecture |
  | **Aparna C** | 💻 ETL Pipeline & SQL Cleansing |
  | **Malavika Nair** | 📊 DAX Analytics & Metric Modeling |
  | **Lucky Aswal** | 🎨 Dashboard UX/UI & BI Lead |

</div>

  <p>⭐ Built with precision by Team CogniStream ⭐</p>
</div>

<h3 align="center">
  🚀 <em>Phase 1 Status Report: Foundation, SQL Staging & Star Schema Scaffolding</em>
</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Week-1%20Complete-success?style=for-the-badge&logo=git&logoColor=white"/>
  <img src="https://img.shields.io/badge/Status-Green%20(On%20Schedule)-blue?style=for-the-badge&logo=statuspage&logoColor=white"/>
  <img src="https://img.shields.io/badge/Records%20Staged-168k%2B%20Rows-orange?style=for-the-badge&logo=databricks&logoColor=white"/>
</p>

---

## 📅  # Developer Flow & Interruption Analytics

A data cleaning project analyzing how software developers spend their working hours — tracking deep focus ("flow") time versus time lost to interruptions like Slack messages, meetings, and alerts.

## Project Overview

This project uses IoT-style activity tracking data from 150 developers across 5 teams (Infra & CI/CD, Frontend Platform, Core Engine, Product Eng, Data Platform), logged over a 3-month period (Jan–Mar 2026). Every work session is recorded — what the developer was doing, whether they were interrupted, and how long it took them to mentally recover afterward.

The goal so far has been to take the raw source files and turn them into clean, reliable, ready-to-analyze data.

## Tools Used

- **Python** (pandas, numpy) — data cleaning and merging
- **Jupyter Notebook** — step-by-step cleaning process

## Project Structure

    Developer-Flow-Analytics/
    ├── README.md
    ├── data/
    │   ├── raw/
    │   │   ├── fact_developer_activity_log.csv
    │   │   ├── fact_flow_daily.csv
    │   │   ├── dim_developer.csv
    │   │   ├── dim_activity_type.csv
    │   │   ├── dim_interruption.csv
    │   │   └── dim_date.csv
    │   └── cleaned/
    │       ├── fact_developer_activity_log_cleaned.csv
    │       ├── fact_flow_daily_cleaned.csv
    │       ├── dim_developer_cleaned.csv
    │       ├── dim_activity_type_cleaned.csv
    │       ├── dim_interruption_cleaned.csv
    │       ├── dim_date_cleaned.csv
    │       └── activity_log_merged.csv
    └── notebooks/
        ├── Dev_Flow_Data_Cleaning.ipynb
        └── Dev_Flow_Data_Merging.ipynb

## The Dataset

This is a star-schema dataset made up of 2 fact tables and 4 dimension (lookup) tables:

| File | Rows | Description |
|---|---|---|
| `fact_developer_activity_log.csv` | 158,152 | The most detailed table — one row per work session (coding, meetings, debugging, etc.) |
| `fact_flow_daily.csv` | 9,600 | One row per developer per day — a daily summary of focus time and interruptions |
| `dim_developer.csv` | 150 | Developer info — name, team, seniority, IDE used, timezone |
| `dim_activity_type.csv` | 6 | What each activity type means (e.g. coding, code review, meetings) |
| `dim_interruption.csv` | 8 | What each interruption type means (e.g. Slack DM, PagerDuty alert) |
| `dim_date.csv` | 90 | Calendar lookup — day of week, sprint name, weekend flag |

## What Was Done So Far

### 1. Data Cleaning (`Dev_Flow_Data_Cleaning.ipynb`)
- Loaded all 6 raw files and checked their shape and structure
- Checked every file for duplicate rows and missing values
- Converted timestamp and date columns into proper date/time types
- Checked that the numbers made sense — no negative durations, no sessions ending before they start, no impossible values
- Found and fixed one tricky issue: the word `"None"` in `dim_interruption.csv` (meaning "no interruption") was being read by pandas as a missing value instead of actual text
- Checked that every ID in the main activity log (developer, activity, interruption, date) correctly matches a row in its lookup table
- Saved all 6 cleaned files

### 2. Data Merging (`Dev_Flow_Data_Merging.ipynb`)
- Loaded the cleaned files
- Re-checked that all IDs still matched correctly before joining
- Combined the main activity log with all 4 lookup tables into a single merged file (`activity_log_merged.csv`), so all the useful info (developer name, team, activity type, interruption type, date/sprint info) is available in one place
- Confirmed no missing values were introduced by the merge

## Key Findings from Cleaning

- The raw data was already in good shape — no duplicate rows and no missing values in the true source data
- Average flow efficiency across all developers and days is about 79%
- Developer names repeat across a shared name pool (e.g. multiple people named "Marcus Vance"), but each has a unique `developer_id`, so this is expected and not a data error

## Next Steps

- Explore the merged data further (which teams/interruption types affect flow the most, trends by sprint, etc.)
- Build a Power BI dashboard on top of the merged dataset
- Write up business insights and recommendations, similar to the AtmoSync project
