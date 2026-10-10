CareFlow dataset bundle — synthetic project data
=================================================

Files:
1. careflow_event_log.csv
   10485 event rows across 1500 patient cases.
   Primary columns: Case_ID, Activity_Name, Timestamp, Event_Order,
   Department, Triage_Priority, Shift, Doctor_ID.
   Use this file for PM4Py process discovery and transition-time analysis.

2. careflow_case_details.csv
   1500 patient-case records with synthetic demographic and operational attributes.
   Join to event log using Case_ID. Use for Power BI slicers and case-level analysis.

3. careflow_reference_pathways.csv
   Reference activity sequences for three illustrative pathways.
   This is a project assumption for practice, not an official clinical protocol.

Important:
- All records are fully synthetic and contain no real patient information.
- Events and delays are simulated for learning, not clinically validated.
- Event timestamps represent activity event times; the gap between consecutive
  event timestamps can be used as an approximate transition/wait interval.
- Do not treat simulated patterns as evidence about real hospitals.

Suggested workflow:
Python/Pandas: validate, clean, derive transition intervals, analyze repeated activities.
PM4Py: discover process models from careflow_event_log.csv.
MySQL: load event, case, and reference pathway tables.
Power BI: visualize patient journey duration, transition waits, loops, and variations.

Example PM4Py setup:
    import pandas as pd
    import pm4py
    df = pd.read_csv("careflow_event_log.csv")
    df["Timestamp"] = pd.to_datetime(df["Timestamp"])
    log = pm4py.format_dataframe(
        df, case_id="Case_ID", activity_key="Activity_Name",
        timestamp_key="Timestamp"
    )
    tree = pm4py.discover_process_tree_inductive(log)
    pm4py.view_process_tree(tree)
