# Customer Master Change Data Capture (CDC) Pipeline using Databricks MERGE

## Document Information

| Field         | Value                        |
| ------------- | ---------------------------- |
| Project       | Customer Master CDC Pipeline |
| Pipeline Type | Batch ETL                    |
| Processing    | Change Data Capture (CDC)    |
| Storage       | Delta Lake                   |
| Platform      | Databricks                   |
| Language      | PySpark & SQL                |

---

# 1. Objective

The objective of this pipeline is to synchronize the **Customer Master** table with the latest customer changes received from the source system.

The pipeline processes customer **INSERT**, **UPDATE**, and **DELETE** operations using the Databricks **MERGE INTO** command. When multiple changes exist for the same customer, only the latest record based on the **updated_at** timestamp is applied.

---

# 2. Business Scenario

The source application sends only the customer records that have changed instead of sending the complete customer master table every day.

The pipeline must process these changes and keep the Customer Master table synchronized with the source system.

---

# 3. Source Datasets

## Customer Master (Target Table)

| Column Name         | Description                                     |
| ------------------- | ----------------------------------------------- |
| customer_id         | Unique customer identifier                      |
| name                | Customer name                                   |
| city                | Customer city                                   |
| status              | Customer status                                 |
| updated_at          | Last update timestamp from source               |
| ingestion_timestamp | Time when the record was loaded into Databricks |
| source_file_name    | Source file name                                |

---

## Customer CDC (Incoming File)

| Column Name         | Description                |
| ------------------- | -------------------------- |
| customer_id         | Unique customer identifier |
| name                | Customer name              |
| city                | Customer city              |
| status              | Customer status            |
| operation           | INSERT, UPDATE or DELETE   |
| updated_at          | Timestamp of the change    |
| ingestion_timestamp | File ingestion timestamp   |
| source_file_name    | Source file name           |

---

# 4. Pipeline Workflow

```
Customer Master (Target)
            │
            ▼
Read Existing Customer Master
            │
            ▼
Read Incoming CDC File
            │
            ▼
Validate Incoming Records
            │
            ▼
Filter Invalid Records
            │
            ▼
Deduplicate CDC Records
(Latest updated_at Wins)
            │
            ▼
MERGE INTO Customer Master
            │
     ┌──────┼────────┐
     │      │        │
 INSERT  UPDATE   DELETE
     │      │        │
     └──────┼────────┘
            │
            ▼
Updated Customer Master
```

---

# 5. Business Rules

* customer_id is the primary key.
* Only INSERT, UPDATE and DELETE operations are allowed.
* If multiple records are received for the same customer, the record with the latest updated_at timestamp is processed.
* Records with invalid operations are rejected.
* Records with missing customer_id are rejected.
* Audit columns are maintained for data lineage and traceability.

---

# 6. Data Validation Rules

The following validations are performed before processing the CDC data.

* customer_id should not be null.
* operation should be one of INSERT, UPDATE or DELETE.
* Duplicate customer records are resolved using the latest updated_at timestamp.
* Invalid records are separated for further analysis.

---

# 7. Processing Logic

### Step 1

Load the Customer Master Delta table.

---

### Step 2

Load the incoming CDC file into a staging DataFrame.

---

### Step 3

Validate the incoming CDC data.

Remove records with:

* Missing customer_id
* Invalid operation values
* Invalid timestamps (if applicable)

---

### Step 4

Deduplicate the CDC data.

If multiple changes are available for the same customer, keep only the latest record based on updated_at.

---

### Step 5

Execute MERGE INTO.

Processing logic:

* If operation = INSERT and customer does not exist, insert the record.
* If operation = UPDATE and customer exists, update the existing record.
* If operation = DELETE and customer exists, delete the record.
* Ignore outdated records.

---

### Step 6

Verify the final Customer Master table after the merge operation.

---

# 8. Expected Output

After successful execution:

* New customers are inserted.
* Existing customer information is updated.
* Deleted customers are removed.
* Duplicate CDC records are resolved.
* Customer Master remains synchronized with the source system.

---

# 9. Databricks Features Used

* Delta Lake
* Delta Tables
* MERGE INTO
* Change Data Capture (CDC)
* Upsert
* Window Functions
* ROW_NUMBER()
* Data Validation
* Data Deduplication
* Audit Columns

---

# 10. Folder Structure

```

Assignment_02_CDC/
│
├── Bronze/
│   ├── 01_Load_Customer_Master
│   └── 02_Load_Customer_CDC
│
├── Silver/
│   ├── 03_Clean_Customer_Master
│   ├── 04_Clean_Customer_CDC
│   └── 05_Deduplicate_CDC
│
├── Gold/
│   ├── 06_Merge_Customer_Master
│   └── 07_Verify_Output
│
└── Playbook.md
```

---

# 11. Success Criteria

The pipeline is considered successful when:

* All valid INSERT records are added to the Customer Master table.
* All valid UPDATE records correctly modify existing customers.
* All valid DELETE records remove existing customers.
* Only the latest change for each customer is applied.
* Invalid records are excluded from processing.
* The Customer Master table accurately reflects the latest state of the source system.

---

# 12. Conclusion

This playbook describes the implementation of a Customer Master Change Data Capture (CDC) pipeline using Databricks and Delta Lake. By leveraging the MERGE INTO command, the pipeline efficiently processes inserts, updates, and deletes while ensuring data consistency, maintaining audit information, and applying the latest customer changes. This approach represents a standard industry practice for building reliable and scalable data engineering pipelines.
