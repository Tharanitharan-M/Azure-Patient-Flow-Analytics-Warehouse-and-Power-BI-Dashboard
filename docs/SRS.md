# Software Requirements Specification (SRS)
## Real Time Patient Flow and Bed Occupancy Analytics Platform

**Document Owner:** Tharanitharan Muthuthirumaran  
**Version:** 0.1  
**Last Updated:** February 2026

---

## 1. Overview

### 1.1 Purpose
This document describes the requirements for a real time analytics platform that helps hospitals monitor patient flow and bed occupancy. The goal is to take data from multiple hospital systems, clean it, organize it into a reporting friendly model, and expose it through Power BI dashboards that refresh frequently.

This SRS is meant to be practical. It focuses on what is needed to build a working pipeline for analytics, plus what could be added later to make it closer to a production platform.

### 1.2 Background and Problem
Midwest Health Alliance runs seven hospitals and needs better visibility into patient movement, especially during busy periods. Right now, data is scattered across registration systems, EHR extracts, and department reference files. Without a centralized view, it is hard to identify bottlenecks early or understand capacity constraints across departments like Emergency, ICU, and Surgery.

### 1.3 Goals
- Near real time visibility into admissions, discharges, and occupancy trends
- Department level bottleneck detection
- Demographic insights by gender and age group
- Automated orchestration so refreshes are repeatable
- Basic reliability features so failures are visible and do not silently break dashboards

### 1.4 Stakeholders
**Primary business stakeholders**
- COO and operations leadership, want high level monitoring and decision support
- Department managers, want visibility into their unit and bottlenecks
- Bed management team, wants occupancy and throughput signals

**Technical stakeholders**
- Data engineering team, responsible for ingestion, processing, and reliability
- BI developer or analyst, responsible for Power BI dashboards and KPI definitions
- IT security or compliance lead, responsible for access control and auditing

### 1.5 Definitions
- **Bronze:** raw data as received, minimal changes
- **Silver:** cleaned and standardized data
- **Gold:** curated data for analytics and reporting
- **SCD2:** storing history when dimension attributes change, by keeping multiple versions of the same entity
- **Star schema:** a modeling approach using fact tables for events and dimension tables for context

---

## 2. Scope

### 2.1 In Scope
- Ingest real time admission and discharge events (or simulated events) into a landing layer
- Ingest daily batch EHR extracts
- Ingest department metadata such as capacity and staffing counts
- Build Bronze, Silver, Gold layers
- Apply data quality rules in Silver
- Build a star schema style reporting model in the curated layer
- Support analytics queries using an Azure SQL serving layer (Synapse in the reference design, Azure SQL is acceptable depending on implementation)
- Build a Power BI dashboard with core KPIs
- Orchestrate data refresh using Azure Data Factory
- Produce basic documentation: architecture overview, data model, KPI definitions, and run steps

### 2.2 Out of Scope (for this initial version)
- Production grade monitoring with full alert routing to on call rotations
- Enterprise identity integration and full RBAC matrix across multiple user groups
- Full HIPAA compliant deployment and legal compliance processes
- Advanced ML predictions, such as forecasting occupancy or staffing needs
- Real integrations with actual hospital systems (this version may use synthetic or sample data)

### 2.3 Assumptions
- Data used for this implementation is synthetic or de identified and contains no real PHI
- Refresh frequency depends on available services and cost limits
- “Near real time” in this context means frequent refreshes that are good enough for operational awareness, not necessarily sub second streaming analytics

### 2.4 Constraints
- Limited time and limited budget for cloud resources
- The platform must be explainable and reproducible from the repo and docs
- The solution should prioritize clarity and reliability over complicated optimizations

---

## 3. System Overview and High Level Architecture

### 3.1 Logical Flow
1. **Ingestion**
   - Real time events arrive from registration systems or a simulator
   - Daily EHR extract files arrive as batch data
   - Department metadata is ingested as reference data

2. **Storage and Processing**
   - Bronze stores raw files or raw tables
   - Silver cleans and standardizes records, fixes common data issues, and applies validation rules
   - Gold produces curated tables for analytics, including a star schema model

3. **Serving and Reporting**
   - Curated Gold tables are served through Synapse or Azure SQL for reporting queries
   - Power BI dashboards connect to the serving layer and refresh on a schedule

4. **Orchestration**
   - Azure Data Factory coordinates batch ingestion, processing jobs, and Gold refresh steps

---

## 4. Functional Requirements

### 4.1 Data Sources

**FR 1. Real time registration events**
- The system shall ingest admission and discharge events as they occur
- Each event should include at minimum: patient identifier, department, event type, and timestamp
- The system should tolerate missing optional fields without failing the pipeline

**FR 2. Daily EHR extracts**
- The system shall ingest daily EHR extracts as batch files
- The system shall validate file presence and basic schema requirements before processing

**FR 3. Department metadata**
- The system shall ingest department metadata such as capacity and staffing counts
- Department metadata should be treated as reference data and used to compute occupancy percent and workload metrics

---

### 4.2 Medallion Layers

**FR 4. Bronze layer**
- The system shall store raw events and raw extracts in the Bronze layer
- Bronze data should be stored in a way that supports replay and reprocessing

**FR 5. Silver layer**
- The system shall clean and standardize records in Silver
- The system shall address common dirty data issues:
  - Missing timestamps when possible, or flagging records that cannot be corrected
  - Duplicate patient ids or duplicate events, using deterministic rules for de duplication
  - Invalid timestamp ordering such as discharge before admission

**FR 6. Gold layer**
- The system shall create curated analytics tables in Gold
- Gold tables should be designed for reporting performance and clarity

---

### 4.3 Schema Evolution

**FR 7. Schema evolution handling**
- The system should tolerate the introduction of new attributes in incoming data
- New attributes should not break existing pipelines or dashboards
- Schema changes should be documented, and there should be a strategy for how new columns are surfaced to Silver and Gold

---

### 4.4 Data Modeling

**FR 8. Star schema**
- The system shall store analytics tables in a star schema format
- The star schema shall include:
  - A fact table representing patient visit or patient movement events
  - Dimension tables representing patient, department, date, and hospital or location

**FR 9. SCD Type 2**
- The system should preserve history for patient and department attributes using an SCD2 approach
- The system should record effective start and end dates and an active flag for current records

Note: If full SCD2 is not implemented in the first version, the data model should still be documented and added as a planned enhancement.

---

### 4.5 Analytics and Dashboarding

**FR 10. Analytics query layer**
- The system shall expose curated tables through a SQL based serving layer
- Queries should support filtering by hospital, department, time range, and demographics

**FR 11. Power BI dashboard**
- The system shall provide a Power BI dashboard that supports operational monitoring
- The dashboard shall include at minimum:
  - Current occupancy percent by department
  - Gender based occupancy distribution
  - Average length of stay per patient

---

### 4.6 Orchestration and Automation

**FR 12. Batch orchestration**
- The system shall use Azure Data Factory to schedule ingestion of daily EHR extracts
- The system shall run processing steps in the correct order, Bronze then Silver then Gold

**FR 13. Real time triggers**
- The system should support frequent refresh of curated tables, either via triggers or a short interval schedule
- Failure of a step should stop dependent steps and surface an error

---

### 4.7 Data Quality and Validation Outputs

**FR 14. Data quality rules**
- The system shall simulate realistic dirty data issues in the input and demonstrate how they are handled in Silver
- The system shall produce validation outputs that show counts of issues found and fixed or flagged

Examples of checks:
- Missing admission timestamps
- Duplicate patient ids or duplicate events
- Discharge timestamp earlier than admission timestamp

---

### 4.8 Security and Access

**FR 15. Role based access**
- The system should support role based access control at least at the level of:
  - Executive view across all hospitals
  - Department manager view restricted to one department
- If full RBAC is not implemented in the first version, the design and planned approach should be documented

---

## 5. Non Functional Requirements

### 5.1 Performance
- Dashboards should load within a reasonable time for common filters such as department and date range
- The serving layer should support aggregation queries across large volumes without timing out

### 5.2 Reliability
- Pipeline failures should be visible and should not silently produce stale dashboards
- Processing steps should be repeatable and idempotent where possible, meaning re running a job should not corrupt results

### 5.3 Maintainability
- The system should be documented so another person can set it up using the repo
- Naming conventions should be consistent across layers and tables

### 5.4 Security and Privacy
- The solution should avoid storing real personally identifiable patient information
- Access credentials and secrets must not be committed to the repository

### 5.5 Cost Awareness
- The solution should use cost conscious service tiers where possible
- The design should avoid unnecessary always on compute if not needed

---

## 6. Data Model Summary

### 6.1 Proposed Tables
This is a suggested star schema structure. Exact naming may differ based on implementation.

**Dimensions**
- DimDate
- DimHospital
- DimDepartment
- DimPatient (or a de identified patient dimension)
- DimDemographics (optional, can be derived from patient attributes)

**Facts**
- FactPatientFlow or FactVisits
  - Keys to Date, Hospital, Department, Patient
  - Metrics such as occupancy count contribution, LOS minutes, wait minutes, admission and discharge flags

### 6.2 SCD2 Columns (for Patient and Department history)
If SCD2 is implemented:
- natural_key
- surrogate_key
- effective_start_date
- effective_end_date
- is_current

---

## 7. Acceptance Criteria

The project is considered successful if:

1. A working pipeline exists that ingests raw data and produces Bronze, Silver, and Gold outputs.
2. ADF can run the pipeline steps automatically, at least for daily batch ingestion and Gold refresh.
3. Power BI can connect to the curated tables and refresh to provide operational views.
4. Data quality checks demonstrate handling of missing, duplicate, and invalid timestamp issues.
5. Basic documentation exists for architecture, data model, KPI definitions, and setup steps.
6. The system can tolerate basic schema changes without breaking the pipeline, or at minimum documents the strategy clearly.

---

## 8. Future Enhancements
These items are valuable but may be implemented after the initial version:

- Full SCD2 implementation for patient and department dimensions
- Stronger schema evolution handling with versioning and migration scripts
- Full RBAC matrix aligned to hospital org structure
- Automated alerting using Azure Monitor and notification channels
- Cost controls and environment separation for dev, test, and prod
- Additional dashboards for staffing demand and predictive analytics

<!-- ---

## 9. Appendix
- Repository structure and run steps are documented in `docs/06_runbook.md`
- KPI definitions are documented in `docs/05_kpi_definitions.md`
- Data model diagrams and table definitions are documented in `docs/04_data_model.md` -->