**English** | [繁體中文](README_ZH-TW.md)

# BOM Management Platform | Lowest-Cost BOM Data and Decision Platform

Transform the cross-functional data, business rules, and usage workflows required by the lowest-cost BOM model into a governed, traceable day-to-day operations platform that continuously supports decision-making.

## Purpose

Raw materials account for approximately 70% to 80% of the total cost of stainless-steel manufacturing industry. Fluctuations in raw-material prices change the optimal material mix for each steel grade, directly affecting procurement planning and cost competitiveness.

The lowest-cost BOM model relies on four major inputs: raw-material quantity limits, raw-material composition, raw-material prices, and production plans. These inputs are owned by different functions and vary in update frequency, data format, validation method, and use case. In order to manage the input parameters and build reliable data for the lowest-cost BOM model, we established standardized and automated processe by defining consistent data definitions, business rules, and collaboration workflows.

We bulit the BOM Management Platform from the ground up. By translating each function's professional judgment into data definitions, maintenance rules, and automated workflows, and making each function accountable for the data it owns, the result of the lowest-cost BOM model is taken seriously by all related functions. This brings the decision-making based on the lowest-cost BOM model become critical and merge into daily operations.

## Outcomes

- **Established a foundation of data and rules across functions:** Standardized the definitions, sources, ownership, and validation methods for the four major inputs, creating a single source of truth for BOM calculations.
- **Brought BOM results into two decision-making scenarios:** Weekly runs produce steel-grade BOMs and aggregated material demand for the next three months, supporting procurement planning and weekly reviews. The shop floor can also recalculate BOMs immediately when schedules change instead of waiting for the next weekly run.
- **Made every BOM reproducible and traceable:** Each run generates a unique key that links the raw-material quantity limits, composition, prices, production plan, and BOM output used in that run, making it possible to reconstruct the exact data conditions behind the result.
- **Established a process for validation and continuous improvement:** Using Power BI tracks changes in weekly consumption forecasts and compares theoretical BOMs with actual material usage, allowing users to review gaps between model results and operations.
- **Management Scale:** The platform supports more than 50 raw materials and a monthly raw-material cost base of approximately NT$1 billion

The platform’s greatest value goes beyond workflow automation. It connects data, models, and cross-functional roles and responsibilities into a consensus-driven decision-making process, enabling the lowest-cost BOM to be continuously adopted, validated, and improved.

## Approach

### 1. Input definitions and management approach

Clarify the definitions, business owners, use case, and update frequency of the four major input, then design tailored management approaches based on the characteristics of each input.

| Input | Management approach |
|---|---|
| Raw-material quantity limits | Procurement maintains the limits in Power Apps. The data is written to SharePoint and then synchronized to the on-premises SQL database |
| Raw-material composition | The data is maintained in Power Apps. Changes must be approved by the relevant functions before being synchronized to the SQL database |
| Raw-material prices | Standardized prices are generated on a daily schedule based on pricing definitions provided by procurement |
| Production plan | The production planning team uploads a standardized Excel file to SharePoint, and Power Automate converts it into structured data |

These management workflow do more than move data — they embed data standards, maintenance ownership, and validation rules into day-to-day operations, creating a reliable input foundation for the model.

### 2. Build cross-system data flows

Establish a cross-functional data exchange process using the Microsoft 365 platform.

Use Power Apps, Excel, and SharePoint as interfaces for maintaining parameters; Power Automate for workflow orchestration and data extraction; and the On-premises Data Gateway to synchronize data with the on-premises SQL database.

### 3. Deploy the lowest-cost BOM model across different use cases

| Use case | Need | Execution and delivery |
|---|---|---|
| Weekly planning | Raw material requirements for the next three months | Power Automate calls a Flask server on a Windows VM to run the model. The Excel output is pushed to SharePoint and published to Teams |
| Real-time shop-floor production | Generate updated BOMs for each steel grade in response to scheduling changes | MES calls a batch file through an API to run the model. The results are written to a staging table, and MES is notified when they are ready to retrieve |

This design enables the same calculation logic to support both material planning and real-time operations, preventing different use cases from developing inconsistent calculation methods.

### 4. BOM Versioning, Tracking, Analytics, and Exception Management

Each model run generates a unique, time-based key. Using a standardized primary-key structure, it links the raw-material quantity limits, composition, prices, production plan, and BOM output used in that run. These records are stored in `BOM_Record`, ensuring that every analysis result can be traced back to its complete set of input versions and supporting reproducibility, version comparison, and issue investigation.

Using `BOM_Record` as the data source, Power BI visualizes changes in monthly raw-material demand and compares the theoretical BOM with actual material inputs, supporting ongoing monitoring, optimization, and process improvement of the lowest-cost BOM model.

If an error occurs or the model has no feasible solution in use cases of **Approach 3**, the system sends a notification to Teams through a Power Automate webhook, enabling the team to respond promptly and analyze the issue using the versioned data.

## Architecture

```mermaid
flowchart TB
    subgraph Source[Business Data and Ownership]
        A[Procurement: raw-material quantity limits and prices]
        B[Quality Assurance and Industrial Engineering: raw-material composition]
        C[Production plan]
    end

    A --> D[Data maintenance, format control, and approval]
    B --> D
    C --> D
    D --> E[Power Apps, SharePoint, and Power Automate]
    E --> F[(On-premises SQL database: standardized input data)]

    subgraph UseCase[Analytical Model Use Cases]
        G[Weekly planning: Power Automate and Flask]
        H[MES shop-floor scheduling: API and batch]
    end

    G --> I[Windows VM: lowest-cost BOM model]
    H --> I
    F --> I

    I --> J[(BOM_Record: input versions and analytical results)]
    I --> K[Planning results: Excel, SharePoint, and Teams]
    I --> L[Real-time results: staging table and MES]
    J --> M[Power BI: raw-material demand, and theoretical-vs-actual analysis]
    I -. Error or no feasible solution .-> N[Webhook alert to Teams]
```

*Note: The core lowest-cost BOM model, raw-material price calculations, and cost management were owned by other members of the team.*

## Skills

| Capability | Skill | Use in the project |
|---|---|---|
| User Collaboration and Interface Design | Power Apps, SharePoint, Excel | Parameter maintenance, production-plan submission |
| Data Workflows and Automation | Power Automate、On-premises Data Gateway、Webhook、Teams | Cloud-to-on-premises data synchronization, workflow orchestration, model triggering, and exception notifications |
| Data Processing and Application Integration | Flask、Python、SQL Server、Windows Server | Model Integration, scheduled Execution, and Real-Time Shop-Floor calculations |
| Analytics and Decision Support | Power BI | raw-material demand, and theoretical-versus-actual material usage analysis |

This case study presents only de-identified business context, data flows, platform architecture, and individual contributions. It does not include company source data, actual parameters, material numbers, formulas, connection details, or details of the core model.
