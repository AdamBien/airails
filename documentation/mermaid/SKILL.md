---
name: mermaid
description: Generate Mermaid overview diagrams for architecture and component visualization. Use when asked to create, generate, or draw Mermaid diagrams, architecture overviews, component diagrams, dependency graphs, or system visualizations. Triggers on "mermaid diagram", "architecture diagram", "component overview", "dependency graph", "draw a diagram", or requests to visualize system structure. Not for detailed class diagrams, sequence diagrams, or UML.
---

# Mermaid Overview Diagrams

## Rules
- prefer flowchart (graph) with TD or LR direction
- wrap in ` ```mermaid ` fenced code block
- use stadium-shaped nodes: `([text])`
- use meaningful node IDs reflecting component names
- use `-->` for dependencies, `-.->` for optional dependencies
- label arrows only when ambiguous: `Gateway -->|routes| Service`
- visualize only high-level concepts — omit classes, methods, implementation details
- use subgraphs sparingly for logical grouping
- write diagram into the target file (e.g. README.md) when context is clear

## Example

```mermaid
graph LR
    Gateway([API Gateway]) -->|routes| OrderService([Order Service])
    OrderService --> OrderDB([Order DB])
    OrderService -.-> NotificationService([Notification Service])
```
