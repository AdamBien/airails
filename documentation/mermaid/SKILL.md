---
name: mermaid
description: Generate Mermaid overview diagrams for architecture and component visualization. Use when asked to create, generate, or draw Mermaid diagrams, architecture overviews, component diagrams, dependency graphs, or system visualizations. Triggers on "mermaid diagram", "architecture diagram", "component overview", "dependency graph", "draw a diagram", or requests to visualize system structure. Not for sequence diagrams or class diagrams.
---

# Mermaid Overview Diagrams

## Rules

- prefer flowchart (graph) with TD or LR direction
- wrap in ` ```mermaid ` fenced code block
- use stadium-shaped nodes: `([text])`
- use meaningful node IDs reflecting component names
- node IDs are global — identical IDs merge into one node; prefix a layer node's ID with its BC: `OrdersBoundary`, `PaymentsControl`
- use `-->` for dependencies, `-.->` for optional/async dependencies
- label arrows only when ambiguous: `Gateway -->|routes| Service`
- visualize only high-level concepts — omit classes, methods, implementation details
- use subgraphs for logical grouping (subsystems)
- write diagram into the target file (e.g. README.md) when context is clear

## BCE Shape Mapping

| Element | Mermaid syntax | Style class |
|---|---|---|
| Business Component (BC) | `Orders([Orders])` | `bc` |
| Business Component with its layers shown | `subgraph Orders` | `bc` |
| Subsystem | `subgraph Billing` | `subsystem` |
| External service | `PayGW([Payment Gateway])` | `ext` |
| Boundary layer | `OrdersBoundary([Boundary])` | `boundary` |
| Control layer | `OrdersControl([Control])` | `control` |
| Entity layer | `OrdersEntity([Entity])` | `entity` |

## Style Classes

Define these `classDef` entries at the end of the graph and apply with `class` or `:::`; a subgraph takes its class through `class`:

```
classDef bc fill:#dae8fc,stroke:#6c8ebf,color:#000
classDef subsystem fill:#f5f5f5,stroke:#666666,color:#000
classDef ext fill:#fff2cc,stroke:#d6b656,color:#000,stroke-dasharray:5 5
classDef boundary fill:#d5e8d4,stroke:#82b366,color:#000
classDef control fill:#e1d5e7,stroke:#9673a6,color:#000
classDef entity fill:#fff2cc,stroke:#d6b656,color:#000
```

## Color Palette

- BC blue: fill `#dae8fc`, border `#6c8ebf`
- Subsystem gray: fill `#f5f5f5`, border `#666666`
- External yellow: fill `#fff2cc`, border `#d6b656`
- Boundary green: fill `#d5e8d4`, border `#82b366`
- Control purple: fill `#e1d5e7`, border `#9673a6`
- Entity yellow: fill `#fff2cc`, border `#d6b656`

## Example — Between Components

```mermaid
graph LR
    subgraph Billing
        Orders([Orders]) -->|charges| Payments([Payments])
    end
    Orders -.->|async| Notifications([Notifications])
    Payments -->|REST| PayGW([Payment Gateway])

    classDef bc fill:#dae8fc,stroke:#6c8ebf,color:#000
    classDef subsystem fill:#f5f5f5,stroke:#666666,color:#000
    classDef ext fill:#fff2cc,stroke:#d6b656,color:#000,stroke-dasharray:5 5
    class Orders,Payments,Notifications bc
    class Billing subsystem
    class PayGW ext
```

## Example — Within Components

```mermaid
graph LR
    subgraph Orders
        OrdersBoundary([Boundary]) --> OrdersControl([Control])
        OrdersControl --> OrdersEntity([Entity])
    end
    subgraph Payments
        PaymentsControl([Control]) --> PaymentsEntity([Entity])
    end
    OrdersControl -->|charges| PaymentsControl
    PaymentsControl -->|REST| PayGW([Payment Gateway])

    classDef bc fill:#dae8fc,stroke:#6c8ebf,color:#000
    classDef ext fill:#fff2cc,stroke:#d6b656,color:#000,stroke-dasharray:5 5
    classDef boundary fill:#d5e8d4,stroke:#82b366,color:#000
    classDef control fill:#e1d5e7,stroke:#9673a6,color:#000
    classDef entity fill:#fff2cc,stroke:#d6b656,color:#000
    class Orders,Payments bc
    class OrdersBoundary boundary
    class OrdersControl,PaymentsControl control
    class OrdersEntity,PaymentsEntity entity
    class PayGW ext
```
