**English** | [繁體中文](README_ZH-TW.md)

# BOM Management Platform | Lowest-Cost BOM Data and Decision Platform

Starting from business use cases and user needs, the platform turns the cross-functional data, business rules, and workflows required by the lowest-cost BOM model into a governed and traceable platform that supports day-to-day operations and ongoing decision-making.

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

### 1. Clarify use cases and define data rules

For each type of input, I first clarified its role in BOM decisions, the responsible business function, its update frequency, and the controls it required. I then designed a management process based on the characteristics of the data:

| Input | Business and data definition | Management approach |
|---|---|---|
| Raw-material quantity limits | Represent procurement's estimate of the quantity available for purchase each month and serve as supply constraints in the model | Procurement maintains the limits in Power Apps. The data is written to SharePoint and then synchronized to the on-premises SQL database |
| Raw-material composition | Defines the composition data used by the model to calculate the material mix for each steel grade | The data is maintained in Power Apps. Changes must be approved by the relevant functions before being synchronized to the SQL database |
| Raw-material prices | Provide the price basis for comparing different material combinations in the model | Standardized prices are generated on a daily schedule based on pricing definitions provided by procurement |
| Production plan | Defines the planned production volume and calculation scope for each steel grade | The production planning team uploads a standardized Excel file to SharePoint, and Power Automate converts it into structured data |

These workflows do more than move data. They embed data formats, ownership, and validation rules into daily operations, creating a reliable input foundation for the model.

### 2. Build a cross-system data flow

The Microsoft 365 platform is used to support cross-functional data exchange. Power Apps provides the parameter maintenance interface, while SharePoint and standardized Excel files capture input from each function. Power Automate controls the workflows and extracts the data, and the On-premises Data Gateway synchronizes it to the on-premises SQL database.

### 3. Apply the same model to different decision-making processes

Weekly planning and real-time shop-floor calculations use the same lowest-cost BOM model, but they use different production plans, triggers, and delivery methods based on the use case:

| Use case | Decision need | Execution and delivery |
|---|---|---|
| Weekly planning | Give procurement visibility into raw-material demand for the next three months and support the weekly review of model constraints | Power Automate calls a Flask server on a Windows VM to run the model. The Excel output is returned to SharePoint and published to Teams |
| Real-time shop-floor production | Generate a new BOM for each steel grade immediately when the production schedule changes | MES calls a batch file through an API to run the model. The results are written to a staging table, and MES is notified when they are ready to retrieve |

This design allows the same analytical logic to support both material planning and real-time operations, preventing different use cases from developing inconsistent calculation methods.

### 4. Build a BOM version data model, analytical validation, and exception management

Each model run generates a unique time-based key. A predefined primary-key structure links the raw-material quantity limits, composition, prices, production plan, and BOM output used in that run. The data is stored in `BOM_Record`, allowing every result to be traced back to its complete input version and supporting result reproduction, version comparison, and issue investigation.

Power BI uses the version data in `BOM_Record` to show changes in weekly consumption forecasts and compare theoretical BOMs with actual material usage. This allows users to identify differences among the plan, model results, and actual execution.

An execution log is also stored for every run. If the program encounters an error or the model cannot find a feasible solution, a Power Automate webhook sends a message to Teams so that the team can respond quickly and use the version data to investigate the issue.

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
    J --> M[Power BI: version, forecast, and theoretical-vs-actual analysis]
    I -. Error or no feasible solution .-> N[Webhook alert to Teams]
```

I was responsible for clarifying the use cases and defining the data and business rules, as well as designing and developing the parameter workflows, data model, Power Platform automation, on-premises integration, dual execution modes, MES integration, Power BI analysis, and execution monitoring. The core lowest-cost BOM model, raw-material price calculation, and cost management were handled by other team members.

## Technology

| Capability | Technology | Use in the project |
|---|---|---|
| User collaboration and workflows | Power Apps, SharePoint, Excel, Teams | Parameter maintenance, production-plan submission, approvals, reviews, and result publishing |
| Data flow and automation | Power Automate, On-premises Data Gateway, webhook | Cloud-to-on-premises data synchronization, workflow control, model triggering, and exception notifications |
| Application and system integration | Flask, REST API, batch file, MES | Model integration, scheduled execution, and real-time shop-floor calculations |
| Data processing, modeling, and execution | Python, SQL Server, Windows VM | Data processing, standardized inputs, version data model, model execution, and log management |
| Analysis and decision support | Power BI | Consumption forecasts, version changes, and theoretical-versus-actual material usage analysis |

This case study presents only de-identified business context, data flows, platform architecture, and individual contributions. It does not include company source data, actual parameters, material numbers, formulas, connection details, or details of the core model.
