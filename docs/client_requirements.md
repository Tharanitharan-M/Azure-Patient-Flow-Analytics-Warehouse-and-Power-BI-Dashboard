# Client Request (Simulated)
## Real Time Data Platform for Patient Flow and Bed Occupancy Analytics

**From:** Dr. Emily Carter, Chief Operations Officer, Midwest Health Alliance (MHA)  
**To:** Data Engineering Solutions Partner  
**Date:** February 2026  
**Purpose:** Request for a real time analytics platform to improve patient flow visibility across MHA hospitals

---

## 1. Business Background
Midwest Health Alliance (MHA) is a network of seven hospitals across the Midwest. Our operations teams are frequently under pressure during high demand periods, especially seasonal surges such as flu outbreaks. Today we do not have one centralized, real time view of patient movement across departments. Data exists across multiple systems and is not easily usable for operational decision making.

As a result, it is difficult to answer simple questions quickly, such as which departments are over capacity, where bottlenecks are forming, and how patient demographics relate to demand patterns.

---

## 2. Business Objectives
We would like a platform that supports near real time operational decisions and daily planning. The main goals are:

1. Monitor admissions and discharges to reduce waiting time and improve throughput.
2. Identify department level bottlenecks, including Emergency, Surgery, and ICU.
3. Provide demographic KPIs, including gender based and age based views.
4. Improve reliability through automated alerts and visibility when data pipelines fail.

---

## 3. Functional Requirements

### 3.1 Data Sources
The solution should integrate three sources of data:

1. **Real time registration events**  
   Streaming patient admission and discharge events from hospital registration systems.

2. **Daily EHR extracts**  
   Batch extracts from Electronic Health Records for clinical context and completeness.

3. **Department metadata**  
   Reference data such as department capacity and staffing, used for occupancy and workload metrics.

---

### 3.2 Data Processing and Storage
1. The platform should follow a **Medallion architecture**, with Bronze, Silver, and Gold layers.  
   - Bronze: raw ingested data as received  
   - Silver: cleaned and standardized data  
   - Gold: curated tables for analytics and dashboards

2. The platform should support **schema evolution**, meaning new patient attributes should not cause downtime.

3. The platform should implement **Slowly Changing Dimension Type 2 (SCD2)** for patient and department history, so we can analyze how attributes change over time.

4. The curated analytics layer should follow a **star schema** design for reporting performance and clarity.

---

### 3.3 Analytics and Dashboards
1. Analytics queries should be supported using **Azure Synapse** as the serving layer for reporting queries.
2. Dashboards should be built in **Power BI** and include at least the following KPIs:
   - Current occupancy percent by department
   - Gender based occupancy distribution
   - Average length of stay per patient

---

### 3.4 Orchestration and Automation
Use **Azure Data Factory (ADF)** to automate and coordinate the workflow:
1. Daily batch ingestion from EHR extracts
2. Triggers or scheduling for real time processing steps
3. Gold layer refreshes so dashboards stay up to date

---

### 3.5 Data Quality Requirements
The platform should handle realistic operational data issues in the Silver layer. Examples include:
1. Missing admission times
2. Duplicate patient IDs
3. Incorrect timestamps, such as discharge earlier than admission

We would like validation outputs or reports that indicate what was corrected or flagged.

---

### 3.6 Security and Compliance
The system should support **role based access control (RBAC)** so that users from different departments see appropriate data access. This includes at minimum separating operational views by department.

---

## 4. Deliverables
The engagement should produce:
1. A functional Azure based data pipeline that ingests, cleans, and publishes analytics tables
2. A Power BI dashboard connected to the analytics serving layer
3. Data quality and validation outputs
4. Full documentation including architecture overview, data model, and a Git repository with reproducible steps

---

## 5. Success Criteria
The solution will be considered successful if:
1. Dashboards refresh frequently enough to support near real time decision making
2. Pipelines run automatically without manual intervention using ADF
3. Schema changes do not cause downtime or broken dashboards