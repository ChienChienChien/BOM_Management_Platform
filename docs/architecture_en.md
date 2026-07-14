[繁體中文](architecture.md) | **English**

# Bill of Materials (BOM) Planning and Governance | System Architecture

## End-to-End Architecture

```mermaid
flowchart TB
    subgraph Owners[Parameters and Business Inputs]
        A["Industrial engineering and QA: material composition"]
        B["Procurement: prices and purchasing limits"]
        C["Logistics: production plan"]
        D["Steelmaking: shop-floor schedule and real-time demand"]
    end

    E["Parameter management and approval: application and workflow"]
    F["Parameter database and version history"]
    G["BOM orchestration: integration, scheduling, real-time trigger"]
    H["Team-developed core BOM model"]
    I["Results and input snapshots: versions, parameters, validation"]

    subgraph Outputs[Publishing and Use]
        J["BOM1: weekly release"]
        K["BOM2: real-time shop-floor response"]
        L["Downstream: cost control and inventory forecasting"]
    end

    A --> E
    B --> E
    C --> E
    D --> G
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
    I --> K
    J --> L
    K --> L
    I -. "Exception feedback and version validation" .-> E
```

## Component Responsibilities

| Component | Responsibility |
|---|---|
| Parameters and business inputs | Domain owners provide composition, prices, purchasing limits, production plans, and shop-floor schedules |
| Parameter management and approval | Provides the maintenance interface and manages requests, approvals, notifications, and production updates |
| Parameter database and version history | Stores approved parameters, before-and-after changes, approval status, and history |
| BOM orchestration | Integrates and validates inputs, handles scheduled or real-time requests, and invokes the core model |
| Core BOM model | Produces material-use results from approved inputs; led by another team member |
| Results and input snapshots | Stores model outputs together with the parameters, plans, and schedules used for each run |
| Publishing and use | Releases BOM1 and BOM2 and supplies downstream cost and inventory forecasting |

## Parameter Change Workflow

```mermaid
stateDiagram-v2
    state "Change Requested" as Request
    state "Owner Approval" as Approval
    state "Update Approved Parameters" as Update
    state "Run BOM" as Run
    state "Validate Results" as Validate
    state "Publish Version" as Publish
    state "Store History" as Record

    [*] --> Request
    Request --> Approval: Submit change
    Approval --> Request: Return for revision
    Approval --> Update: Approve
    Update --> Run: Apply approved parameters
    Run --> Validate: Generate trial result
    Validate --> Run: Exception found
    Validate --> Publish: Validation passed
    Publish --> Record: Release approved result
    Record --> [*]
```

## BOM1 and BOM2

| Item | BOM1 | BOM2 |
|---|---|---|
| Primary use | Procurement and medium-term planning | Real-time shop-floor material decisions |
| Update model | Scheduled weekly | Triggered by the shop-floor system |
| Main inputs | Approved parameters and production plan | Approved parameters, shop-floor schedule, and real-time demand |
| Shared controls | Parameter version, input snapshot, result history, validation, and exception alerts | Parameter version, input snapshot, result history, validation, and exception alerts |

## Traceability Model

```mermaid
flowchart LR
    A["Published BOM version"] --> B["Calculation run"]
    B --> C["Parameter version"]
    B --> D["Production plan or schedule"]
    B --> E["Model inputs and validation results"]
    C --> F["Change and approval history"]
```

This relationship allows users to trace an unexpected result back to the calculation run, parameter version, plan or schedule, validation evidence, and source change. The platform therefore retains the conditions behind each result, not only the output file.

## Diagram Notes

- Solid arrows indicate the main data and process flow.
- The dashed arrow indicates validation and exception feedback.
- All system and field names are de-identified.
