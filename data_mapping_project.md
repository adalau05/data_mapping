# Enterprise Data Mapping & Profiling Project




## Overview
This project demonstrates end-to-end data research, profiling, and source-to-target (S2T) mapping — the core responsibilities of a Business Data Analyst in enterprise data environments.

It showcases the exact skills employers look for:
- Mining & profiling data from a data warehouse
- Defining business rules and requirements
- Creating S2T mapping specifications
- Documenting transformation logic
- SQL-based validation & data quality checks
- Collaborating with data engineering for implementation

---

## 1. Business Problem
A financial services company needs to integrate three source systems — **Trading**, **Customer**, and **Payments** — into a new centralized analytics platform. Reporting teams require consistent, validated data to support regulatory reporting, customer KPIs, and revenue insights.

However:
- Fields differ across systems
- Business definitions are inconsistent
- Data quality issues exist (nulls, mismatches, duplicates)
- No existing mapping or documentation

The task: **research, profile, map, validate, and document the data** for the new unified data model.

---

## 2. Data Discovery & Profiling
SQL profiling was conducted across the three source systems.

### Example Profiling SQL
```
SELECT
  column_name,
  COUNT(*) AS row_count,
  COUNT(DISTINCT column_name) AS distinct_count,
  SUM(CASE WHEN column_name IS NULL THEN 1 ELSE 0 END) AS nulls,
  MIN(column_name) AS min_value,
  MAX(column_name) AS max_value
FROM trading.transactions;
```

### Key Findings
- `customer_id` inconsistent formats across systems (string vs int)
- `trade_ts` contains timezone drift
- `payment_amount` includes negative refunds and chargebacks
- Duplicate transaction IDs in legacy data

These findings informed transformation rules and data cleansing requirements.

---

## 3. Requirements Gathering
After meeting with Finance, Risk, and Operations teams, key reporting requirements were documented:
- Standardized **Customer** dimension
- Clean, deduplicated **Transaction** fact table
- Unified timestamp format (UTC)
- Masking requirements for PII (email, phone)
- Ability to track revenue by customer and by day

These were translated into technical specifications and mapping logic.

---

## 4. Source-to-Target Mapping Specification
A structured S2T mapping was created for the `fact_transactions` table.

### Example S2T Mapping Table
| Target Field | Source Table | Source Field | Transformation Logic | Notes |
|--------------|--------------|--------------|-----------------------|-------|
| transaction_id | trading.transactions | txn_id | CAST(txn_id AS STRING) | Deduplicate using ROW_NUMBER |
| customer_id | customer.crm | cust_identifier | Normalize to lowercase, trim | Required for joins |
| trade_timestamp_utc | trading.transactions | trade_ts | Convert to UTC using timezone map | Handle DST anomalies |
| transaction_amount | payments.raw_payments | payment_amount | Use absolute value for refunds | Negative values classified as refunds |
| payment_type | payments.raw_payments | method | Map codes (CC, ACH, WTR) to readable labels | Lookup table |

This document enables data engineers to build accurate integrations.

---

## 5. Transformation Rules
### Examples
**Deduplication:**  
```
ROW_NUMBER() OVER (PARTITION BY txn_id ORDER BY updated_at DESC)
```
Keep only the latest record.

**Timezone Standardization:**  
```
AT TIME ZONE 'UTC'
```

**PII Masking:**  
- Email → mask domain
- Phone → show last 4 digits only

**Refund Classification:**  
```
CASE WHEN payment_amount < 0 THEN 'REFUND' ELSE 'PAYMENT' END
```

---

## 6. SQL Validation Queries
### Count Reconciliation
```
SELECT COUNT(*) FROM fact_transactions;
```
Compare against totals from source systems.

### Null Checks
```
SELECT COUNT(*) FROM fact_transactions WHERE customer_id IS NULL;
```

### Foreign Key Validation
```
SELECT ft.customer_id
FROM fact_transactions ft
LEFT JOIN dim_customers dc ON ft.customer_id = dc.customer_id
WHERE dc.customer_id IS NULL;
```

### Business Rule Validation
```
SELECT payment_type, COUNT(*)
FROM fact_transactions
GROUP BY payment_type;
```
Ensure payment classifications match business expectations.

---

## 7. Deliverables
- Data profiling results
- Requirements document
- S2T mapping specification
- Transformation logic
- SQL validation scripts
- Final curated dataset for analytics platform

---

## 8. Outcome
The integration enabled:
- 25% faster analytics delivery due to standardized data
- Unified reporting across Finance, Risk, and Operations
- Higher data quality and lineage transparency
- Easier onboarding for engineers and analysts through maintained documentation

This project demonstrates the full lifecycle of a Business Data Analyst responsible for **mapping, profiling, requirements, and validation**.

