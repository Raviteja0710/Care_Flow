# CareFlow Pandas Work — Team Handover Guide

> Purpose: This document explains what each input/output file contains, what the Pandas notebook does, which KPIs are calculated, and how teammates should use the outputs.
>
> **Data warning:** The supplied CareFlow data is synthetic. Timestamp fields contain times only, not full calendar dates. Journey durations and transition gaps derived from reconstructed timestamps are provisional until validated. Do not present these as real hospital performance.

## 1. Project and ownership

**Project:** CareFlow — Clinical Pathway Process Mining  
**Pandas responsibility:** Dataset inspection, data-quality checks, repeated-activity analysis, journey-duration and transition-gap calculations, and preparing tables for teammates.  
**Other project tools:** MySQL for structured storage, PM4Py for process mining, Power BI for dashboard visuals.

## 2. Folder structure

The current VS Code workspace contains the source Excel files and `cleaning.ipynb`. The notebook writes analysis outputs to the `output of pandas` folder.

```text
Care Flow/
├── python scripts/
│   ├── cleaning.ipynb
│   ├── care_flow_event_log.xlsx
│   ├── care_flow_case_details.xlsx
│   ├── careflow_reference_pathway.xlsx
│   └── output of pandas/
│       ├── careflow_cleaned_events.xlsx
│       ├── careflow_journey_summary.xlsx
│       ├── careflow_repeated_activities.xlsx
│       ├── careflow_transition_analysis.xlsx
│       └── careflow_patient_summary.xlsx
└── README_Pandas_CareFlow.md
```

Some outputs are created only after their corresponding export cells have been run. Check the folder before assuming every file exists.

## 3. Input files — what each file means

### `care_flow_event_log.xlsx`
**Grain:** One row per recorded activity/event.  
**Known size:** 10,485 rows and 8 columns.

| Column | Meaning |
|---|---|
| `Case_ID` | Identifier for a patient journey; links event rows to case details. |
| `Activity_Name` | Activity recorded, such as Registration, Triage, Doctor Consultation, Treatment, Discharge, Blood Test, X-Ray, or CT Scan. |
| `Timestamp` | Time of the event in `HH:MM:SS` form; source has no calendar date. |
| `Event_Order` | Recorded sequence of events within a case. |
| `Department` | Department label associated with the event. |
| `Triage_Priority` | Priority category. |
| `Shift` | Shift label. |
| `Doctor_ID` | Synthetic doctor identifier. |

**Use it for:** activity sequence analysis, event counts, repeated activity detection, and transition-gap calculations.

### `care_flow_case_details.xlsx`
**Grain:** One row per patient case.  
**Known size:** 1,500 cases and 9 columns.

| Column | Meaning |
|---|---|
| `Case_ID` | Unique case key used to join tables. |
| `Age` | Age value in the synthetic data. |
| `Sex` | Recorded category. |
| `Triage_Priority` | Case-level priority. |
| `Department` | Case-level department. |
| `Shift` | Case-level shift. |
| `Assigned_Doctor_ID` | Assigned synthetic doctor ID. |
| `Arrival_Timestamp` | Arrival time only, without calendar date. |
| `Discharge_Status` | Recorded synthetic case outcome. |

**Use it for:** grouping case journeys by age, priority, department, shift, doctor assignment, and discharge status.

### `careflow_reference_pathway.xlsx`
**Grain:** One row per step in an illustrative reference pathway.  
**Use it for:** comparing observed activity sequences with example templates. These templates are illustrative, not clinically validated medical guidelines.

## 4. Notebook: `cleaning.ipynb`

The notebook is the main Pandas workflow. Run its cells in order from top to bottom after restarting the kernel to make sure all variables are recreated.

| Notebook task | Main Pandas methods | Purpose |
|---|---|---|
| Load source data | `pd.read_excel()` | Load Excel sheets into DataFrames. |
| Inspect structure | `.head()`, `.shape`, `.columns`, `.info()` | Understand rows, columns and data types. |
| Check data quality | `.isnull().sum()`, `.duplicated().sum()` | Identify missing values and exact duplicate rows. |
| Count categories | `.nunique()`, `.value_counts()` | Count cases and activity frequencies. |
| Count events per case/activity | `.groupby().size()` | Detect activity repetitions within cases. |
| Build journey summary | `.groupby().agg()` | Calculate first and last reconstructed event times and duration. |
| Build transition table | `.groupby().shift()` and time subtraction | Calculate gaps between consecutive events. |
| Merge case information | `.merge()` | Combine case-level details with calculated journey/repeat metrics. |
| Export results | `.to_excel()` | Save analysis tables for teammates. |

## 5. Initial checks already observed

The event-log inspection returned:
- **10,485** event rows and **8** columns.
- **1,500** unique `Case_ID` values.
- **8** distinct activity names.
- **0** missing values reported in event-log columns.
- **0** exact duplicate rows.
- `Event_Order` was increasing for all 1,500 cases.

Observed event counts (these are event rows, not unique patients):
- Doctor Consultation: 2,816
- Triage: 1,788
- Registration: 1,500
- Treatment: 1,500
- Discharge: 1,500
- Blood Test: 659
- X-Ray: 657
- CT Scan: 65

An initial analysis run reported 1,158 cases with repeated activities (77.20%). Re-run the notebook and validate the logic before treating this as a final result.

## 6. Output files — what each file contains and which KPI uses it

All outputs are intended to be saved under `output of pandas/`.

### `careflow_cleaned_events.xlsx`
**Contains:** Event rows plus derived columns created during the notebook, such as event time, arrival time, reconstructed timestamp, previous activity/time, and transition gap. Exact columns depend on which cells were run before export.

**Use for:**
- Total Patients — distinct count of `Case_ID`.
- Total Events — row count.
- Activity frequency — counts of `Activity_Name`.
- Repeat Activity Rate — derived by grouping `Case_ID` and `Activity_Name`.
- Transition gap analysis — uses consecutive timestamps within each `Case_ID`.

### `careflow_journey_summary.xlsx`
**Contains:** One row per `Case_ID`, with `Start`, `End`, and `Duration_Min` (duration in minutes), based on the reconstructed timestamp field.

**Use for:**
- Average Journey Duration — mean of `Duration_Min`.
- Median Journey Duration — median of `Duration_Min`.
- Journey duration comparisons after joining to case details.

**Caution:** These are provisional until the time-only source data and reconstruction assumptions are validated.

### `careflow_repeated_activities.xlsx`
**Contains:** Case/activity combinations where the same `Activity_Name` occurs more than once in the same `Case_ID`, with a `Count` column.

**Use for:**
- Repeated activity investigation.
- Repeat Activity Rate: number of distinct cases with at least one repeated activity divided by total distinct cases, multiplied by 100.
- Identifying which activity types repeat most frequently.

**Definition used:** A case counts as a repeat case if at least one activity type has `Count > 1`. This is not the same as counting every repeated event row.

### `careflow_transition_analysis.xlsx`
**Contains:** One row per observed `Previous_Activity` → `Activity_Name` transition, with:
- `count`: number of observed transitions.
- `mean`: average timestamp gap in minutes.
- `median`: median timestamp gap in minutes.

**Use for:**
- Comparing gaps between activity transitions.
- Finding transitions with relatively large recorded time gaps.

**Caution:** A gap between recorded events is a timestamp-gap proxy; it is not automatically verified queue waiting time.

### `careflow_patient_summary.xlsx`
**Contains:** Intended patient-level combination of case details, journey duration, and repeat activity fields such as `Repeated_Activity_Types` and `Total_Repeated_Events`.

**Use for:**
- Comparing journey duration and repeated activities by triage priority, department, shift, age group, or discharge status.
- Supplying case-level attributes for downstream analysis.

This file is created only if the merge and export cells run successfully. Confirm it has one row per `Case_ID`.

## 7. KPI definitions

| KPI | Definition | Main data/output |
|---|---|---|
| Total Patients | Number of distinct cases: `events["Case_ID"].nunique()` | Event log / cleaned events |
| Average Journey Duration | Mean of each case's last reconstructed timestamp minus first reconstructed timestamp, in minutes | Journey summary |
| Median Journey Duration | Median of per-case journey duration, in minutes | Journey summary |
| Average Transition Gap | Mean gap between consecutive event timestamps, in minutes | Cleaned events / transition analysis |
| Repeat Activity Rate | `distinct cases with any repeated activity / total distinct cases × 100` | Repeated activities + event log |
| Pathway Deviation Rate | Percentage of cases whose observed activity sequence differs from a chosen reference template, using a documented matching rule | Event log + reference pathway file; **not yet implemented in the described Pandas code** |

Do not report Pathway Deviation Rate as calculated until the comparison rule and code are implemented.

## 8. Timestamp limitation — read before using duration KPIs

The source `Timestamp` and `Arrival_Timestamp` contain only clock times. They do not include event dates. Direct conversion to datetime can assign a default date such as 1900-01-01, which is not a real event date.

The notebook has explored building a relative timeline from arrival time, `Event_Order`, and midnight rollovers. This produces an estimate under assumptions; it does not recover verified original calendar timestamps. Validate the reconstruction, especially if activities may be out of chronological order or if a case spans more than one midnight. Until validated, mark journey duration and transition-gap results as **provisional**.

## 9. Handover to teammates

- **MySQL teammate:** Use validated event/case tables and keep `Case_ID` consistent as the join key.
- **PM4Py teammate:** Use the event log with case identifier `Case_ID`, activity column `Activity_Name`, and a validated chronological timestamp. Confirm event ordering and timestamp suitability before discovery.
- **Power BI teammate:** Use the patient summary and transition/journey tables only after definitions are agreed. Keep KPI names and units consistent; do not label timestamp gaps as confirmed queue waiting times.
- **All teammates:** The dataset is synthetic. Describe results as a demonstration of the analysis workflow, not actual hospital findings.

## 10. Before final submission

- [ ] Restart kernel and run all notebook cells in order.
- [ ] Confirm all expected Excel outputs exist.
- [ ] Check output row counts and nulls after every merge.
- [ ] Verify `careflow_patient_summary.xlsx` contains one row per case.
- [ ] Validate the timestamp reconstruction before publishing duration KPIs.
- [ ] Document the exact rule used for pathway deviations before calculating that KPI.
