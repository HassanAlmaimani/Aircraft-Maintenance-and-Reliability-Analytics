# Aircraft Maintenance & Reliability Analytics

A Power BI case study focused on **aircraft maintenance, fleet reliability, downtime, and component performance** in a fictional MRO environment.

This project was built as a practical end-to-end analytics exercise covering data profiling, ETL, dimensional modeling, DAX, KPI design, and dashboard development.

> **Note:** All data used in this project is synthetic and does not represent Saudia, Saudia Technic, or any real airline or maintenance organization.

---

## Project Objective

The goal of this project is to help a fictional maintenance and engineering team answer questions such as:

* Which aircraft generate the most maintenance burden?
* Which aircraft have the highest unscheduled maintenance rate?
* Which components contribute most to aircraft downtime?
* How does aircraft utilization affect reliability comparisons?
* What proportion of maintenance is scheduled versus unscheduled?
* Which maintenance events and components are associated with repeat faults?
* Are work orders being completed on time and within planned labor hours?

The final solution uses a Power BI semantic model and dashboard to translate raw operational records into decision-support metrics and reliability insights.

---

## Tools Used

* **Microsoft Power BI Desktop**
* **Power Query**
* **DAX**
* **Dimensional / Star Schema Modeling**
* **CSV source files**
* **Excel** for documentation and data dictionary
* **SQL concepts** for joins, aggregation, filtering, and relational modeling
* **Microsoft Fabric concepts** as a potential future deployment architecture

---

## Dataset

The project uses a synthetic MRO dataset covering approximately two years of maintenance activity.

### Dataset Scope

| Dataset            | Records | Description                                    |
| ------------------ | ------: | ---------------------------------------------- |
| Aircraft           |      24 | Fleet master data                              |
| Components         |      30 | Aircraft component master                      |
| Technicians        |      25 | Maintenance technician records                 |
| Maintenance Events |   1,224 | Scheduled and unscheduled maintenance activity |
| Work Orders        |   1,338 | Maintenance work execution records             |
| Flight Utilization |     576 | Monthly aircraft flight-hour and cycle data    |

The analysis period covers:

**July 2024 – June 2026**

The raw data intentionally contains realistic data-quality problems such as:

* missing values
* duplicated records
* inconsistent text formatting
* mixed date formats
* negative or invalid numerical values
* incomplete foreign keys
* inconsistent categorical values

These issues were included to simulate a realistic analytics workflow rather than working with perfectly clean data.

---

## Project Workflow

```text
Raw CSV Files
      ↓
Data Profiling
      ↓
Power Query ETL
      ↓
Cleaned Data Layer
      ↓
Star Schema / Semantic Model
      ↓
DAX Measures & KPIs
      ↓
Power BI Dashboard
      ↓
Insights & Recommendations
```

---

## Data Cleaning & ETL

Data preparation was completed using **Power Query inside Power BI Desktop**.

The queries were organized into three layers:

```text
01 - Staging
02 - Clean
03 - Model
```

### Staging Layer

The staging layer preserves the original source files with minimal modification.

Examples:

```text
stg_aircraft
stg_components
stg_technicians
stg_maintenance_events
stg_work_orders
stg_flight_utilization
```

These queries are not loaded into the final Power BI model.

### Clean Layer

The clean layer contains transformation and validation logic.

Key transformations included:

* trimming and cleaning text fields
* standardizing categorical values
* correcting data types
* handling mixed date formats
* detecting duplicate maintenance events
* identifying invalid negative repair hours
* preserving unknown values instead of automatically replacing them with zero
* validating work-order date logic
* checking composite keys
* performing referential-integrity checks

Where values could not be safely corrected, quality flags were used instead of silently guessing.

For example:

```text
RepairHoursQuality
ComponentDataQuality
DowntimeQuality
WorkOrderDateQuality
```

This approach preserves traceability and prevents invalid data from silently affecting KPIs.

---

## Data Model

The final Power BI model follows a **star-schema / fact-constellation design**.

The project contains three fact tables because the source data represents three different business processes and grains.

### Fact Tables

**FactMaintenanceEvent**

Grain:

> One row per maintenance event

Measures include:

* downtime hours
* repair hours
* labor hours
* maintenance cost

---

**FactFlightUtilization**

Grain:

> One row per aircraft per month

Measures include:

* flight hours
* flight cycles
* flights operated

This table provides the exposure denominator required for normalized reliability metrics.

---

**FactWorkOrder**

Grain:

> One row per maintenance work order

Measures include:

* planned hours
* actual hours
* completion status
* priority

Aircraft and component context were merged into the work-order fact to avoid unnecessary fact-to-fact relationships.

---

### Dimension Tables

```text
DimAircraft
DimComponent
DimTechnician
DimDate
```

Shared dimensions allow filters such as aircraft, model, date, and component to propagate consistently across multiple fact tables.

Relationships use:

```text
1 : many
Dimension → Fact
Single-direction filtering
```

This reduces ambiguity and helps prevent incorrect aggregations.

---

## Why Separate Maintenance and Utilization?

Maintenance events and flight utilization exist at different grains.

Maintenance:

```text
One row = one event
```

Utilization:

```text
One row = one aircraft/month
```

Combining them into a single flat table could duplicate monthly flight-hour values across multiple maintenance events and produce incorrect totals.

Keeping them separate allows measures such as:

```text
Unscheduled Events per 1,000 Flight Hours
```

to be calculated correctly using shared Aircraft and Date dimensions.

---

## Key KPIs

### Maintenance Events

```DAX
Maintenance Events =
DISTINCTCOUNT(FactMaintenanceEvent[EventID])
```

---

### Unscheduled Maintenance %

```DAX
Unscheduled Maintenance % =
DIVIDE(
    [Unscheduled Maintenance Events],
    [Maintenance Events]
)
```

---

### MTTR

Mean Time to Repair was defined as average repair time for valid unscheduled maintenance events.

```text
MTTR =
Unscheduled Repair Hours
÷
Valid Unscheduled Repair Events
```

---

### MTBF

Because the dataset does not contain a dedicated failure indicator, **unscheduled maintenance events were used as a proxy for failures**.

```text
MTBF =
Flight Hours
÷
Unscheduled Maintenance Events
```

This assumption is documented explicitly and should not be interpreted as a certified aviation reliability definition.

---

### Unscheduled Events per 1,000 Flight Hours

```DAX
Unscheduled Events per 1K Flight Hours =
DIVIDE(
    [Unscheduled Maintenance Events],
    [Total Flight Hours]
) * 1000
```

This metric allows aircraft with different levels of utilization to be compared more fairly.

---

### Downtime per 1,000 Flight Hours

```DAX
Downtime per 1K Flight Hours =
DIVIDE(
    [Total Downtime Hours],
    [Total Flight Hours]
) * 1000
```

---

### Maintenance Cost per Flight Hour

```DAX
Maintenance Cost per Flight Hour =
DIVIDE(
    [Total Maintenance Cost],
    [Total Flight Hours]
)
```

---

### Repeat Fault Rate

A repeat fault was defined as:

> The same aircraft experiencing another event involving the same component within 30 days of a previous event.

This allows recurring aircraft/component combinations to be identified for investigation.

---

## Power BI Dashboard

The report contains three main pages.

---

### 1. Executive Fleet Overview

Purpose:

> Provide maintenance leadership with a quick view of overall fleet performance.

Headline KPIs include:

* Maintenance Events
* Unscheduled Maintenance %
* MTBF
* MTTR
* Total Downtime
* Total Maintenance Cost

Visuals include:

* unscheduled maintenance trend
* scheduled vs unscheduled maintenance mix
* top aircraft by maintenance downtime
* monthly maintenance cost

---

### 2. Aircraft Reliability Analysis

Purpose:

> Identify aircraft with disproportionately high reliability or maintenance burden.

Key metrics include:

* MTBF
* MTTR
* Unscheduled Events / 1K Flight Hours
* Downtime / 1K Flight Hours
* Maintenance Cost / Flight Hour
* Repeat Fault Rate

Visuals include:

* aircraft reliability ranking
* maintenance cost vs downtime scatter plot
* flight utilization vs unscheduled maintenance
* monthly aircraft downtime trend

---

### 3. Component & Fault Analysis

Purpose:

> Identify the components and fault categories driving maintenance workload and downtime.

Visuals include:

* most frequently maintained components
* components driving the most downtime
* maintenance events by fault category
* repeat-fault rate by component manufacturer
* component frequency vs downtime scatter plot

This page helps distinguish between:

> frequently occurring but low-impact faults

and:

> less frequent but operationally disruptive faults

---

## Key Results

Across the synthetic fleet, the analysis produced approximately:

| KPI                     |             Result |
| ----------------------- | -----------------: |
| Maintenance Events      |              1,224 |
| Unscheduled Maintenance |              57.7% |
| MTBF Proxy              | 248.6 flight hours |
| MTTR                    |          4.4 hours |
| Total Downtime          |      5,569.5 hours |
| Maintenance Cost        |            ~$22.1M |

The results also highlighted several important analytical lessons.

### 1. Raw maintenance counts can be misleading

Aircraft with more maintenance events are not automatically less reliable.

Some aircraft simply operate more flight hours.

Normalizing events by flight utilization produced a more meaningful aircraft comparison.

---

### 2. Maintenance frequency and operational impact are different

Some components appeared frequently but caused relatively little aircraft downtime.

Other components occurred less often but generated substantially greater downtime when they failed.

This shows why prioritization should consider both:

```text
Frequency
+
Operational Impact
```

rather than event count alone.

---

### 3. Aircraft-level reliability issues can be isolated

The dashboard identified aircraft/component combinations with elevated unscheduled maintenance and repeat-fault behavior.

These should be treated as areas for further engineering investigation rather than immediate proof of a specific root cause.

---

### 4. Work-order completion and labor estimation tell different stories

Work orders may perform well against completion deadlines while still exceeding planned labor hours.

This suggests maintenance scheduling and labor estimation should be analyzed separately.

A team can be:

> operationally on time

while still experiencing:

> planning or productivity variance.

---

## Recommendations

Based on the analysis, the fictional maintenance organization should consider:

1. **Prioritizing aircraft with elevated normalized unscheduled-maintenance rates** rather than relying only on raw event counts.

2. **Investigating high-downtime component categories** even when their event frequency is relatively low.

3. **Reviewing recurring aircraft/component fault combinations** for potential reliability or maintenance-process issues.

4. **Tracking maintenance burden relative to aircraft utilization** using flight hours or cycles.

5. **Reviewing work-order labor estimates** where actual labor consistently exceeds planned hours.

6. **Using component manufacturer trends as investigation signals**, while avoiding causal conclusions without controlling for component mix and operational exposure.

---

## Analytical Limitations

This case study intentionally includes several limitations.

### Synthetic Data

The dataset is artificially generated and should only be treated as an analytical training environment.

### MTBF Definition

Unscheduled maintenance events are used as a proxy for failures.

True engineering reliability programs would require more precise failure definitions and engineering records.

### Availability

True aircraft technical availability was not calculated because the dataset does not contain complete scheduled-availability or out-of-service-hour definitions.

### Causality

The dashboard identifies patterns and correlations, not proven causal relationships.

For example:

> a manufacturer with a high repeat-fault rate should be investigated

does **not** automatically mean:

> the manufacturer produces lower-quality equipment.

Differences in component type, aircraft usage, severity, or sample size may explain part of the result.

---

## Skills Demonstrated

This project demonstrates practical experience with:

### Data Preparation

* Power Query
* ETL workflows
* data profiling
* data quality validation
* missing-value handling
* duplicate detection
* date standardization
* business-rule validation

### Data Modeling

* star schemas
* fact and dimension tables
* table grain
* cardinality
* one-to-many relationships
* filter direction
* conformed dimensions
* role-playing date relationships

### DAX

* measures
* calculated columns
* `CALCULATE`
* `DIVIDE`
* `FILTER`
* `DISTINCTCOUNT`
* `ALL`
* `USERELATIONSHIP`
* filter context
* time intelligence

### Analytics

* KPI design
* reliability analysis
* normalization by operational exposure
* root-cause investigation
* repeat-fault analysis
* cost analysis
* maintenance workload analysis
* operational performance measurement

### Visualization

* executive KPI reporting
* ranking analysis
* trend analysis
* scatter plots
* drill-down investigation
* slicers and cross-filtering
* business-focused dashboard design

---

## Repository Structure

```text
aircraft-maintenance-analytics/
│
├── README.md
│
├── data/
│   ├── raw/
│   │   ├── aircraft.csv
│   │   ├── components.csv
│   │   ├── technicians.csv
│   │   ├── maintenance_events.csv
│   │   ├── work_orders.csv
│   │   └── flight_utilization.csv
│   │
│   └── reference_clean/
│
├── documentation/
│   ├── data_dictionary.xlsx
│   ├── PROJECT_BRIEF.md
│   ├── ANALYST_HANDOFF.md
│   ├── RELATIONSHIPS.md
│   └── DATA_QUALITY_CHALLENGE.md
│
├── powerbi/
│   └── Aircraft_Maintenance_Analytics.pbix
│
├── sql/
│
├── python/
│
└── screenshots/
    ├── executive_overview.png
    ├── aircraft_reliability.png
    └── component_fault_analysis.png
```

---

## Project Takeaway

The main lesson from this project was that a Power BI solution is not primarily about creating charts.

The more important work happens before visualization:

```text
Understand the business problem
        ↓
Assess data quality
        ↓
Define the grain
        ↓
Build the data model
        ↓
Define defensible KPIs
        ↓
Validate calculations
        ↓
Visualize and investigate
        ↓
Translate findings into actions
```

By following this process, the final dashboard becomes a decision-support tool rather than simply a collection of visuals.

---

## Disclaimer

This project is an independent educational and portfolio project.

All airline, aircraft, maintenance, technician, supplier, cost, and operational records are fictional and synthetically generated.

The project is not affiliated with or endorsed by Saudia, Saudia Technic, Airbus, Boeing, or any other aviation organization.
