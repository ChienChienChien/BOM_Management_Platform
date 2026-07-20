**English** | [繁體中文](README_ZH-TW.md)

# BOM Management Platform | Lowest-Cost BOM Data and Decision Platform

Starting from business contexts and user needs, this platform transforms the cross-functional data, business rules, and workflows required by a lowest-cost BOM model into a governed, traceable operational platform that supports ongoing decision-making.

## Purpose

In stainless-steel manufacturing, raw materials account for approximately 70% to 80% of total cost. Fluctuations in raw-material prices change the optimal material mix for each steel grade, directly affecting procurement planning and cost competitiveness.

The lowest-cost bill of materials (BOM) model relies on four categories of critical inputs: raw-material procurement limits, material composition, raw-material prices, and production plans. These inputs are owned by different functions and vary in update frequency, data format, validation method, and usage context. This project therefore established consistent data definitions, business rules, and collaboration workflows, using standardized and automated controls to manage model inputs and create a trusted data foundation for stable decision support.

Built from the ground up, the BOM Management Platform translates each function's domain judgment into data definitions, maintenance rules, and automated workflows. It makes participating functions accountable for the data they own and enables them to review BOM results together, embedding lowest-cost BOM decision logic into day-to-day operations.

## Outcomes

- **Established a shared data and rule foundation:** Standardized the definitions, sources, ownership, and validation methods for the four critical inputs, creating the single source of truth used for BOM calculation.
- **Applied analytical outputs to two decision contexts:** Produces weekly steel-grade BOMs and aggregated raw-material demand for the next three months to support procurement planning and weekly reviews, while also enabling real-time recalculation when shop-floor schedules change.
- **Made every analysis reproducible and traceable:** Assigns a unique run key to each execution and links the corresponding procurement limits, composition, prices, production plan, and BOM outputs, allowing the full input state behind each result to be reconstructed.
- **Established validation and continuous-improvement mechanisms:** Uses Power BI to compare changes in weekly consumption forecasts and analyze differences between theoretical BOMs and actual material usage, helping business users evaluate gaps between model outputs and operations.
- **Created a stable operational process:** Supports more than 50 raw materials with a monthly raw-material cost base of approximately NT$1 billion, with immediate notifications when a program error occurs or the model cannot find a feasible solution.

The platform's primary value goes beyond workflow automation. It connects data, analytical models, and cross-functional accountability within a shared decision process, allowing the lowest-cost BOM to be continuously used, validated, and improved.

## Approach

### 1. Clarify usage contexts and define data rules

For each input category, the project first identified its role in BOM decisions, business owner, update frequency, and necessary controls, then designed a management method suited to the data:

| Input | Business and data definition | Management approach |
|---|---|---|
| Raw-material procurement limits | Reflect procurement's assessment of the quantities available for purchase in each month and serve as supply constraints for the model | Maintained by procurement through Power Apps, written to SharePoint, and synchronized to the on-premises SQL database |
| Material composition | Defines the composition data used by the model when calculating the material mix for each steel grade | Maintained through Power Apps; changes require approval from relevant functions before synchronization to the SQL database |
| Raw-material prices | Provide the price basis for comparing alternative material combinations | Generated daily through a scheduled process based on price definitions provided by procurement |
| Production plan | Defines planned production quantities and the calculation scope for each steel grade | Uploaded by production planning to SharePoint in a standardized Excel format and transformed into structured data by Power Automate |

These workflows do more than move data. They embed formatting standards, maintenance ownership, and validation rules into daily operations, creating a trusted input foundation for the model.

### 2. Build a cross-system data flow

The platform uses Microsoft 365 to support cross-functional data exchange. Power Apps serves as the parameter-maintenance interface, SharePoint and standardized Excel files capture input from business functions, and Power Automate manages workflow control and data extraction. The On-premises Data Gateway then synchronizes the data to the on-premises SQL database.

### 3. Apply the same model to different decision workflows

Weekly planning and real-time production calculations use the same lowest-cost BOM model but apply different production plans, triggers, and delivery methods according to the user context:

| Usage context | Decision need | Execution and delivery |
|---|---|---|
| Weekly planning | Give procurement visibility into raw-material demand for the next three months and support weekly reviews of model constraints | Power Automate calls a Flask service on a Windows VM to execute the model; Excel results are returned to SharePoint and published to Teams |
| Real-time shop-floor production | Recalculate steel-grade BOMs immediately when production schedules change | MES calls a batch script through an API to execute the model; results are written to a staging table and MES is notified when they are ready for retrieval |

This design allows one analytical logic to support both material planning and real-time operations, avoiding inconsistent calculation methods across different usage contexts.

### 4. Build a BOM version data model, analytical validation, and exception management

Each model run generates a unique, time-based run key. Predefined primary-key relationships link the corresponding raw-material procurement limits, composition, prices, production plan, and BOM outputs. These records are stored in `BOM_Record`, allowing every analytical result to be traced to its complete input version and supporting reproducibility, version comparison, and issue investigation.

Power BI uses the versioned data in `BOM_Record` to show changes in weekly consumption forecasts and compare theoretical BOMs with actual material usage, helping business users identify differences among plans, model outputs, and actual execution.

Execution logs are also retained for every calculation. If a program error occurs or the model cannot find a feasible solution, a Power Automate webhook sends a notification to the relevant Teams channel, enabling timely monitoring and version-based root-cause analysis.

## Architecture

```mermaid
flowchart TB
    subgraph Source[Business Data and Ownership]
        A[Procurement: raw-material limits and prices]
        B[Quality Assurance and Industrial Engineering: material composition]
        C[Production plan]
    end

    A --> D[Data maintenance, format control, and approval]
    B --> D
    C --> D
    D --> E[Power Apps, SharePoint, and Power Automate]
    E --> F[(On-premises SQL database: standardized inputs)]

    subgraph UseCase[Analytical Model Usage Contexts]
        G[Weekly planning: Power Automate and Flask]
        H[MES shop-floor scheduling: API and batch script]
    end

    G --> I[Windows VM: lowest-cost BOM model]
    H --> I
    F --> I

    I --> J[(BOM_Record: input versions and analytical results)]
    I --> K[Planning output: Excel, SharePoint, and Teams]
    I --> L[Real-time output: staging table and MES]
    J --> M[Power BI: version, forecast, and plan-versus-actual analysis]
    I -. Error or infeasible solution .-> N[Webhook to Teams alert]
```

I was responsible for clarifying usage contexts, defining data and business rules, and designing and developing the parameter workflows, data model, Power Platform automation, on-premises integration, dual-mode execution, MES integration, Power BI analysis, and execution monitoring. The core lowest-cost BOM model, raw-material price calculation, and cost management were owned by other team members.

## Technology

| Capability layer | Technology | Use in the project |
|---|---|---|
| User collaboration and workflow | Power Apps, SharePoint, Excel, Teams | Parameter maintenance, production-plan submission, approval, review, and result publishing |
| Data flow and automation | Power Automate, On-premises Data Gateway, webhook | Cloud-to-on-premises data synchronization, workflow control, model triggering, and exception notification |
| Application and system integration | Flask, REST API, batch script, MES | Model integration, scheduled execution, and real-time shop-floor calculation |
| Data processing, modeling, and execution | Python, SQL Server, Windows VM | Data processing, standardized inputs, versioned data model, model execution, and log management |
| Analytics and decision support | Power BI | Consumption forecasting, version comparison, and theoretical-versus-actual material usage analysis |

This case study presents only de-identified business context, data flows, platform architecture, and individual contributions. It excludes proprietary company data, actual parameters, material numbers, formulas, connection details, and core model logic.
