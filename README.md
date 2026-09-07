# Aircraft Maintenance & Reliability Analytics

A Power BI case study focused on **aircraft maintenance, fleet reliability, downtime, and component performance** in a fictional MRO environment.

This project was built as a practical end-to-end analytics exercise covering data profiling, ETL, dimensional modeling, DAX, KPI design, and dashboard development.

> **Note:** All data used in this project is synthetic and does not represent Saudia, Saudia Technic, or any real airline or maintenance organization.

---

## Live Dashboard

**[View the interactive Power BI dashboard here](https://app.powerbi.com/view?r=eyJrIjoiODk0ODlkMWYtOGM3ZC00N2EyLTk4YTAtZGYyZmU1MDU2YWUwIiwidCI6IjJkMzE5NGUzLTE2NTQtNDZiZC1iYWUyLWFkMzdiYTExYjBhZSIsImMiOjl9)**

---

## Dashboard Preview

### Executive Fleet Overview

<img width="2158" height="1185" alt="image" src="https://github.com/user-attachments/assets/64e03286-f191-489b-8b80-4836fe5bf33b" />


---

### Aircraft Reliability Analysis

<img width="2158" height="1186" alt="image" src="https://github.com/user-attachments/assets/278276c1-4887-46f5-b8e7-6daaa723f4fd" />


---

### Component & Fault Analysis

<img width="2158" height="1183" alt="image" src="https://github.com/user-attachments/assets/02138981-861f-44de-8a26-c0f6a21367b3" />


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

### Data Model Preview

<img width="800" height="518" alt="image" src="https://github.com/user-attachments/assets/adc17c08-5411-4001-a394-a089119a5015" />


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

**FactFlightUtilization**

Grain:

> One row per aircraft per month

Measures include:

* flight hours
* flight cycles
* flights operated

This table provides the exposure denominator required for normalized reliability metrics.

**FactWorkOrder**

Grain:

> One row per maintenance work order

Measures include:

* planned hours
* actual hours
* completion status
* priority

Aircraft and component context were merged into the work-order fact to avoid unnecessary fact-to-fact relationships.

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

### Unscheduled Maintenance %

```DAX
Unscheduled Maintenance % =
DIVIDE(
    [Unscheduled Maintenance Events],
    [Maintenance Events]
)
```

### MTTR

Mean Time to Repair was defined as average repair time for valid unscheduled maintenance events.

```text
MTTR =
Unscheduled Repair Hours
÷
Valid Unscheduled Repair Events
```

### MTBF

Because the dataset does not contain a dedicated failure indicator, **unscheduled maintenance events were used as a proxy for failures**.

```text
MTBF =
Flight Hours
÷
Unscheduled Maintenance Events
```

This assumption is documented explicitly and should not be interpreted as a certified aviation reliability definition.

### Unscheduled Events per 1,000 Flight Hours

```DAX
Unscheduled Events per 1K Flight Hours =
DIVIDE(
    [Unscheduled Maintenance Events],
    [Total Flight Hours]
) * 1000
```

### Downtime per 1,000 Flight Hours

```DAX
Downtime per 1K Flight Hours =
DIVIDE(
    [Total Downtime Hours],
    [Total Flight Hours]
) * 1000
```

### Maintenance Cost per Flight Hour

```DAX
Maintenance Cost per Flight Hour =
DIVIDE(
    [Total Maintenance Cost],
    [Total Flight Hours]
)
```

### Repeat Fault Rate

A repeat fault was defined as:

> The same aircraft experiencing another event involving the same component within 30 days of a previous event.

---

## Power BI Dashboard

The report contains three main pages.

### 1. Executive Fleet Overview

<img width="2158" height="1185" alt="image" src="https://github.com/user-attachments/assets/d9c7b375-ffc7-4190-9945-0eb79b35aca0" />


Purpose:

> Provide maintenance leadership with a quick view of overall fleet performance.

Headline KPIs include:

* Maintenance Events
* Unscheduled Maintenance %
* MTBF
* MTTR
* Total Downtime
* Total Maintenance Cost

---

### 2. Aircraft Reliability Analysis

<img width="2158" height="1186" alt="image" src="https://github.com/user-attachments/assets/cf5201aa-8568-429f-a60b-507874970fa9" />


Purpose:

> Identify aircraft with disproportionately high reliability or maintenance burden.

Key metrics include:

* MTBF
* MTTR
* Unscheduled Events / 1K Flight Hours
* Downtime / 1K Flight Hours
* Maintenance Cost / Flight Hour
* Repeat Fault Rate

---

### 3. Component & Fault Analysis

<img width="2158" height="1183" alt="image" src="https://github.com/user-attachments/assets/776795aa-bcd2-4478-826f-c30c42d55b63" />


Purpose:

> Identify the components and fault categories driving maintenance workload and downtime.

Visuals include:

* most frequently maintained components
* components driving the most downtime
* maintenance events by fault category
* repeat-fault rate by component manufacturer
* component frequency vs downtime scatter plot

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

### 1. Raw maintenance counts can be misleading

Aircraft with more maintenance events are not automatically less reliable.

Some aircraft simply operate more flight hours.

Normalizing events by flight utilization produced a more meaningful aircraft comparison.

### 2. Maintenance frequency and operational impact are different

Some components appeared frequently but caused relatively little aircraft downtime.

Other components occurred less often but generated substantially greater downtime when they failed.

### 3. Aircraft-level reliability issues can be isolated

The dashboard identified aircraft/component combinations with elevated unscheduled maintenance and repeat-fault behavior.

These should be treated as areas for further engineering investigation rather than immediate proof of a specific root cause.

### 4. Work-order completion and labor estimation tell different stories

Work orders may perform well against completion deadlines while still exceeding planned labor hours.

This suggests maintenance scheduling and labor estimation should be analyzed separately.

---

## Recommendations

1. **Prioritize aircraft with elevated normalized unscheduled-maintenance rates** rather than relying only on raw event counts.
2. **Investigate high-downtime component categories** even when their event frequency is relatively low.
3. **Review recurring aircraft/component fault combinations** for potential reliability issues.
4. **Track maintenance burden relative to aircraft utilization** using flight hours or cycles.
5. **Review work-order labor estimates** where actual labor consistently exceeds planned hours.
6. **Use component manufacturer trends as investigation signals**, while avoiding causal conclusions without controlling for component mix and operational exposure.

---

## Repository Structure

```text
aircraft-maintenance-analytics/
│
├── README.md
│
├── data/
│   ├── raw/
│   └── reference_clean/
│
├── documentation/
│   └── data_dictionary.xlsx
│
├── powerbi/
│   └── Aircraft_Maintenance_Analytics.pbix
│
├── screenshots/
│   ├── executive_overview.png
│   ├── aircraft_reliability.png
│   ├── component_fault_analysis.png
│   └── data_model.png
│
├── sql/
└── python/
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

---

## Dashboard Access

**[Open the interactive Power BI report]([PASTE-YOUR-POWER-BI-LINK-HERE](https://app.powerbi.com/view?r=eyJrIjoiODk0ODlkMWYtOGM3ZC00N2EyLTk4YTAtZGYyZmU1MDU2YWUwIiwidCI6IjJkMzE5NGUzLTE2NTQtNDZiZC1iYWUyLWFkMzdiYTExYjBhZSIsImMiOjl9))**

---

## Disclaimer

This project is an independent educational and portfolio project.

All airline, aircraft, maintenance, technician, supplier, cost, and operational records are fictional and synthetically generated.

The project is not affiliated with or endorsed by Saudia, Saudia Technic, Airbus, Boeing, or any other aviation organization.
