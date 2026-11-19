# MJ-Insight-Data-Engineering-Assessment

Case 1 — Incremental Load at Terabyte Scale


Problem and scope
The production scenario describes a SQL Server transaction table with 6 TB (~9 billion rows) and ~25 million new or changed rows daily. Records can change for up to 90 days, some are hard deleted, and the application's ModifiedDate is neither indexed nor reliable.

This repository demonstrates the ingestion and recovery logic using 10 million synthetic records in Databricks Delta tables. It does not connect to a live SQL Server, implement SQL Server CDC capture, or prove performance at 6 TB.
Repository contents
- Case1__Incremental_Load.ipynb — complete synthetic demonstration: source generation, chunked full load, CDC events, MERGE, failure/restart test, reconciliation.

Architecture
Synthetic source_transactions Delta table (10,000,000 rows)
    | ID-range chunks (1,000,000 keys each)
    v
Delta target_transactions  <--- replay-safe MERGE --- synthetic_cdc_events
    |                                    | 100,000 updates, 10,000 deletes
    |                                    | plus 1,000 duplicate update events
    v                                    v
full_load_manifest                cdc_batch_manifest
    |
    v
Compare target with independently built source_current_state

Prerequisites
- Azure Databricks workspace with a Python notebook and Delta Lake support; Serverless can be used.
- Permission to create a schema and managed Delta tables in a Unity Catalog catalog.

Synthetic data and expected results
Item	Expected value
Initial source transactions	10,000,000
Update events	100,000
Hard-delete events	10,000
Extra deliberately duplicated update events	1,000
Total raw CDC event rows	111,000
Distinct changed transaction IDs	110,000
Updated rows whose modified_date is unchanged	10,000
Full-load chunks at 1 million rows each	10
Final target rows	9,990,000
Reconciliation missing rows	0
Reconciliation extra rows	0


The source schema is transaction_id, customer_id, transaction_date, amount, status, and modified_date. CDC adds operation, change_sequence, and change_timestamp.
