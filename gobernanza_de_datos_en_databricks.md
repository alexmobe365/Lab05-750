# Lab: Govern Data with Unity Catalog

In this exercise, you apply governance controls to the **AutoSphere AG** data platform, including tagging data assets, managing retention policies, tracking lineage, and querying audit logs.

---

## Exercise 2: Tag Data Assets for Governance

AutoSphere's compliance team requires that all tables and columns containing personally identifiable information (PII) are consistently labelled. Tags enable automated discovery and will later drive access policies.

In this exercise, you apply table-level and column-level tags to classify the `customer_registrations` table.

### Task 2.1 — Add table-level tags

Tag the `customer_registrations` table to classify it by domain and sensitivity:
- `domain = customer`
- `data_classification = confidential`

🤖 **Genie Code tip:** Ask: *"How do I add key-value tags to a Unity Catalog table in Databricks SQL?"*  
*Hint:* Use `ALTER TABLE ... SET TAGS ('key' = 'value', 'key2' = 'value2')`.

![Set Table Tags](1_6.png)

### Task 2.2 — Add column-level PII tags

Tag the PII columns in `customer_registrations` to identify them as sensitive:
- Column `email` → `pii = email`
- Column `driver_license_no` → `pii = driver_license`

🤖 **Genie Code tip:** Ask: *"How do I add a tag to a specific column in a Unity Catalog table using SQL in Databricks?"*  
*Hint:* Use `ALTER TABLE ... ALTER COLUMN <col> SET TAGS (...)`.

![Set Column PII Tags](2_6.png)

### Task 2.3 — Verify the tags

Use `system.information_schema.table_tags` to confirm the table-level tags have been applied. Then describe the columns to confirm the column tags are present. Use `system.information_schema.column_tags` for this.

🤖 **Genie Code tip:** Ask: *"How can I verify tags on a Unity Catalog table and its columns in Databricks SQL?"*

![Verify Column Tags Query](3_6.png)

---

## Exercise 3: Configure Data Retention

AutoSphere's vehicle telemetry table is written to continuously. To comply with GDPR data minimisation principles and control storage costs, the platform must:
- Retain only 14 days of table history.
- Permanently remove old data files using `VACUUM`.
- Enable predictive optimization so that future maintenance runs automatically.

In this exercise, you configure retention settings, simulate record deletion, and run a `VACUUM` operation.

### Task 3.1 — Configure retention properties

Set the following Delta Lake table properties on `vehicle_telemetry`:
- `delta.logRetentionDuration = interval 14 days`
- `delta.deletedFileRetentionDuration = interval 14 days`

🤖 **Genie Code tip:** Ask: *"How do I set Delta Lake retention duration properties on a Unity Catalog table in Databricks?"*  
*Hint:* Use `ALTER TABLE ... SET TBLPROPERTIES (...)`.

![Set Table Retention Properties](4_5.png)

### Task 3.2 — Simulate a GDPR deletion request

A customer has submitted a right-to-erasure request. Delete all telemetry records for vehicle `VH-003` from the `vehicle_telemetry` table.

After deletion, verify the records are gone by querying the table.

🤖 **Genie Code tip:** Ask: *"How do I delete specific rows from a Delta Lake table in Databricks SQL?"*

### Task 3.3 — Run VACUUM to purge deleted data files

The deleted rows still exist in underlying Parquet files until `VACUUM` is run. Execute `VACUUM` on `vehicle_telemetry` to permanently remove files that are no longer referenced by any current or recent table version.

Use the default retention of 168 hours (7 days). This is safe, works on Serverless compute, and is appropriate for a lab environment.

🤖 **Genie Code tip:** Ask: *"How do I run VACUUM on a Delta Lake table in Databricks, and what does the RETAIN option do?"*

> **Note:** On Serverless compute, overriding the minimum retention duration is not supported. Always specify a `RETAIN` value of at least 168 hours, or omit `RETAIN` entirely to use the default.

![Execute VACUUM](5_5.png)

### Task 3.4 — Enable predictive optimization

Enable predictive optimization on the `governance_lab` schema so that future `VACUUM` and `OPTIMIZE` maintenance runs automatically without manual scheduling.

🤖 **Genie Code tip:** Ask: *"How do I enable predictive optimization on a Unity Catalog schema in Databricks?"*  
*Hint:* Use `ALTER SCHEMA ... ENABLE PREDICTIVE OPTIMIZATION`.

![Enable Predictive Optimization](6_5.png)

---

## Exercise 4: Query Data Lineage Programmatically

📋 **Before running this exercise:** If you haven't already done so, pause here and follow the Catalog Explorer lineage steps in the lab setup instructions to explore lineage visually in the UI. Then return here to query lineage data using SQL.

Unity Catalog captures all read and write events and stores them in the `system.access.table_lineage` system table. This lets you programmatically answer questions like: *"Which tables in our automotive catalog were most queried this week?"*

### Task 4.1 — Find recently accessed tables in your catalog

Query `system.access.table_lineage` to find all write events (source tables) within the past 7 days that relate to `automotive_catalog`. Return the table name and the count of distinct events, ordered by the most active tables first.

🤖 **Genie Code tip:** Ask: *"How do I query system.access.table_lineage to find recently accessed Unity Catalog tables in Databricks?"*  
*Hint:* Filter on `source_table_full_name LIKE 'automotive_catalog%'` and group by table name.

![Query Table Lineage](7_5.png)

### Task 4.2 — View the history of vehicle_telemetry

Use `DESCRIBE HISTORY` to see the full Delta Lake operation history for `vehicle_telemetry`. Identify the version where the deletion occurred (`DELETE` operation).

🤖 **Genie Code tip:** Ask: *"How do I view the full version history of a Delta Lake table in Databricks SQL?"*

![Describe History Query](8_4.png)

---

## Exercise 5: Audit Logging

AutoSphere's compliance team needs to prove to regulators that only authorised users accessed customer data during the past 30 days. The team also wants to be alerted whenever permissions are modified on the Unity Catalog metastore.

You will use the `system.access.audit` system table to answer both of these requirements.

### Task 5.1 — Find who accessed the customer_registrations table

Query `system.access.audit` to list all users who accessed `automotive_catalog.governance_lab.customer_registrations` in the past 30 days. Include the action name and the time of each access, ordered by most recent first.

🤖 **Genie Code tip:** Ask: *"How do I query the Databricks audit log system table to find who accessed a specific Unity Catalog table?"*  
*Hint:* Filter on `request_params.full_name_arg` and `action_name IN ('getTable', 'createTable')`. Use `user_identity.email` for the user column.

![Audit Log Table Access](9_3.png)

### Task 5.2 — Detect permission changes across your catalog

Query `system.access.audit` to list all permission change events (`updatePermissions` action) on securable objects in `automotive_catalog` during the past 30 days. Include who made the change and what object was affected.

🤖 **Genie Code tip:** Ask: *"How do I find all permission changes in the Databricks audit log for a specific Unity Catalog catalog?"*  
*Hint:* Filter on `service_name = 'unityCatalog'`, `action_name = 'updatePermissions'`, and `request_params.securable_full_name LIKE 'automotive_catalog%'`.

![Audit Log Permission Changes](10_3.png)