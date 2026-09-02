# CogniStream: Developer Flow-State & Cognitive Load Analytics

A data analytics project tracking how software developers spend their working hours — measuring deep, focused "flow" work versus time lost to interruptions like Slack messages, meetings, and alerts. Built to replace flawed productivity metrics (lines of code, tickets closed) with a real picture of developer focus and friction.

## Problem Statement

Engineering productivity is often measured using flawed metrics like "lines of code written" or "tickets closed." These measure output but ignore the friction of the development process, failing to identify what actually blocks deep, focused work.

## Use Case

An Engineering Manager reviews the CogniStream dashboard. Instead of seeing how many commits a team pushed, they see a "Context-Switching Tax" analysis — proving how much of a team's peak cognitive flow state is lost to poorly timed interruptions, so they can adjust notification policies and improve developer experience.

## Tools Used

- **Python** (pandas, numpy) — data cleaning and merging
- **Jupyter Notebook** — step-by-step cleaning and merging process
- **Power BI** — dashboard build, DAX measures, data modeling

## Project Structure

    CogniStream-Analytics/
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
    ├── notebooks/
    │   ├── Dev_Flow_Data_Cleaning.ipynb
    │   └── Dev_Flow_Data_Merging.ipynb
    └── dashboard/
        └── CogniStream_Dashboard.pbix

## The Dataset

A star-schema dataset made up of 2 fact tables and 4 dimension (lookup) tables, covering 150 developers across 5 teams over a 3-month period (Jan–Mar 2026).

| File | Rows | Description |
|---|---|---|
| `fact_developer_activity_log.csv` | 158,152 | One row per work session — activity type, timestamps, flow state, interruptions |
| `fact_flow_daily.csv` | 9,600 | One row per developer per day — daily summary of focus time and interruptions |
| `dim_developer.csv` | 150 | Developer info — name, team, seniority, IDE used, timezone |
| `dim_activity_type.csv` | 6 | Activity type lookup (coding, debugging, code review, meetings, etc.) |
| `dim_interruption.csv` | 8 | Interruption type lookup (Slack DM, PagerDuty alert, GitHub PR, etc.) |
| `dim_date.csv` | 90 | Calendar lookup — day of week, sprint name, weekend flag |

## What Was Done

### 1. Data Cleaning (`Dev_Flow_Data_Cleaning.ipynb`)
- Loaded all 6 raw files and checked shape and structure
- Checked for duplicate rows and missing values
- Converted timestamp/date columns to proper date types
- Validated logic — no negative durations, no sessions ending before they start
- Found and fixed a tricky issue: the word `"None"` in `dim_interruption.csv` was being read by pandas as a missing value instead of actual text
- Verified every ID in the activity log correctly matches its lookup table
- Saved all 6 cleaned files

### 2. Data Merging (`Dev_Flow_Data_Merging.ipynb`)
- Loaded the cleaned files
- Re-checked ID matches before joining
- Combined the activity log with all 4 lookup tables into `activity_log_merged.csv`
- Confirmed no missing values were introduced by the merge

### 3. Power BI Dashboard (3 pages)

**Page 1 — Executive Overview**
- 5 KPI cards: Total Developers, Active Developers, Total Interrupted Sessions, Avg Flow Efficiency, Avg Recovery Minutes
- Team slicer (tile panel)
- Donut chart: Cognitive Tax by Team
- Horizontal bar: Recovery time by interruption type
- Matrix: Team × Urgency Tier
- Column chart: Sessions by Activity Type

**Page 2 — Flow State & Interruption Analysis**
- Team and Sprint slicers
- Line chart: Flow efficiency trend by sprint
- Clustered column: Sessions by Activity Type, split by flow state
- Horizontal bar: Recovery latency by interruption category
- Table: Bottom 10 developers by flow efficiency

**Page 3 — Team & Individual Deep Dive**
- Developer slicer
- Matrix: Cognitive Tax by Team and Interruption Type (conditional formatting)
- Line chart: Selected developer's flow efficiency over time
- Card: Uninterrupted Flow Blocks (90+ min sessions, no interruption)
- Card: Context-Switching Tax % (per selected developer)
- Column chart: Flow Efficiency by Seniority Level

## Data Model (Power BI)

Relationships built in Model view:

| From | To | Join Column | Cardinality |
|---|---|---|---|
| `fact_flow_daily_cleaned` | `dim_developer_cleaned` | `developer_id` | Many-to-One |
| `fact_flow_daily_cleaned` | `dim_date_cleaned` | `date_key` | Many-to-One |
| `activity_log_merged` | `dim_developer_cleaned` | `developer_id` | Many-to-One |
| `activity_log_merged` | `dim_date_cleaned` | `date_key` | Many-to-One |
| `activity_log_merged` | `dim_interruption_cleaned` | `interruption_id` | Many-to-One |

The `Measures` table is a dummy organizational table with no relationships — every measure references its source table directly in DAX.

## Key DAX Measures

    Total Developers = DISTINCTCOUNT(dim_developer_cleaned[developer_id])

    Active Developers = DISTINCTCOUNT(fact_flow_daily_cleaned[developer_id])

    Total Sessions = COUNTROWS(activity_log_merged)

    Total Interrupted Sessions =
    CALCULATE(COUNTROWS(activity_log_merged), activity_log_merged[interruption_id] <> 0)

    Total Cognitive Tax Minutes = SUM(fact_flow_daily_cleaned[total_cognitive_tax_minutes])

    Context-Switching Tax % =
    DIVIDE(
        SUM(fact_flow_daily_cleaned[total_cognitive_tax_minutes]),
        SUM(fact_flow_daily_cleaned[total_tracked_minutes])
    )

    Avg Flow Efficiency = AVERAGE(fact_flow_daily_cleaned[flow_efficiency_pct])

    Avg Recovery Minutes =
    CALCULATE(
        AVERAGEX(
            activity_log_merged,
            RELATED(dim_interruption_cleaned[avg_recovery_latency_min])
        ),
        activity_log_merged[interruption_id] <> 0
    )

    Uninterrupted Flow Blocks =
    CALCULATE(
        COUNTROWS(activity_log_merged),
        activity_log_merged[is_deep_work] = TRUE,
        activity_log_merged[interruption_id] = 0,
        activity_log_merged[session_duration_minutes] >= 90
    )

## Key Findings

- Average flow efficiency across all developers is about **79%**
- About **43,000 of 158,152 sessions (~27%)** involved some form of interruption
- **Core Engine** and **Product Eng** carry the highest cognitive tax load among all teams
- Recovery time varies significantly by interruption source — Production Alerts and Direct Communication cost the most recovery time per interruption
- Developer names repeat across a shared name pool (e.g. multiple people named "Marcus Vance"), but each has a unique `developer_id`, so this is expected and not a data error
 
