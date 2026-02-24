---
name: mermaid
description: Generate Mermaid overview diagrams for architecture and component visualization. Use when asked to create, generate, or draw Mermaid diagrams, architecture overviews, component diagrams, dependency graphs, or system visualizations. Triggers on "mermaid diagram", "architecture diagram", "component overview", "dependency graph", "draw a diagram", or requests to visualize system structure.
---

# Mermaid Overview Diagrams

## Diagram Type
- prefer flowchart (graph) for component overviews
- use TD (top-down) or LR (left-right) direction

## Output
- wrap diagram in a ` ```mermaid ` fenced code block for GitHub markdown rendering

## Style
- use stadium-shaped nodes: `([text])`

## Content
- visualize only high-level concepts and modules
- show dependencies with arrows: `API --> Database`
- omit implementation details, classes, and methods
- use subgraphs sparingly for logical grouping
- limit diagram to essential relationships

## Arrows
- use `-->` for dependencies
- use `-.->` for optional/weak dependencies
- label arrows only when relationship type is ambiguous: `Gateway -->|routes| Service`

## Node IDs
- use meaningful, descriptive node IDs: `UserService`, `OrderDB`, `PaymentGateway`
- avoid generic IDs like `A`, `B`, `C` or `node1`, `node2`
- node ID should reflect the component name
- keep IDs concise but self-explanatory
