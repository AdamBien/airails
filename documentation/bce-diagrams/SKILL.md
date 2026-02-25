---
name: bce-diagrams
description: Create high-level overview diagrams showing interactions between business components (BCs), subsystems, services, or systems. Use when asked to create architecture overviews, BC interaction diagrams, subsystem diagrams, service interaction diagrams, system landscapes, or integration maps. Triggers on "diagram", "overview diagram", "BC interaction", "business component diagram", "subsystem diagram", "service interaction", "component interaction", "architecture overview", "system diagram", "integration diagram", or requests to visualize how BCs, subsystems, or services communicate. Not for detailed class diagrams or sequence diagrams.
argument-hint: "[services or components to visualize]"
---

# BCE Overview Diagrams

Create diagrams that show interactions and dependencies between business components (BCs), subsystems, and external services at a high level of abstraction. Based on BCE architecture (see [bce.design](https://bce.design)).

## BCE Terminology

- **Business Component (BC)** — a feature-based unit grouping related boundary, control, and entity packages. Named after its domain responsibility (e.g., `orders`, `payments`, `speakers`).
- **Boundary** — the BC's external interface: JAX-RS resources, facades, health checks.
- **Subsystem** — a grouping of related BCs at a higher level of abstraction.
- In "between components" mode, show BCs and subsystems as opaque nodes. In "within components" mode, expand BCs to show their boundary, control, and entity layers.

## Workflow

1. **Gather information** - Identify BCs, subsystems, and external services to visualize. Analyze the codebase package structure (`[org].[project].[bc].[boundary|control|entity]`) or use the user's description.
2. **Choose detail level and format** - Ask the user using a single AskUserQuestion call with two questions:
   - **Detail level**: "Between components" (each BC is a single opaque node) or "Within components" (show boundary, control, entity layers inside each BC)
   - **Format**: "Mermaid (Recommended)" (text-based, version-control friendly, renders natively in GitHub/GitLab — default for README.md in GitHub projects) or "draw.io" (visual editor, richer styling, exportable to PNG/SVG)
3. **Invoke the chosen skill** - Use the Skill tool to invoke either `mermaid` or `drawio` with the identified BCs, subsystems, interactions, and chosen detail level. When the diagram is destined for a README.md in a GitHub project, default to Mermaid without asking.

## Diagram Content Guidelines

### Between components (default)

- Show BCs, subsystems, and external services — not classes, methods, or fields
- Represent each BC as a single node labeled by its responsibility: `Orders`, `Payments`, `Users`
- Group BCs into subsystems when they share a domain: `subgraph Billing [Orders, Payments, Invoicing]`
- Focus on interactions between BCs: which boundary calls which boundary, what data flows where
- Label arrows with the interaction type when not obvious: `-->|REST|`, `-->|async|`, `-->|JPA|`
- Show external dependencies (third-party APIs, databases, message brokers) as separate nodes

### Within components

- Represent each BC as a subgraph containing its boundary, control, and entity layers
- Show internal interactions: boundary → control → entity
- Show cross-BC interactions at the boundary level: one BC's boundary calls another BC's boundary
- External dependencies connect to the boundary or control layer that uses them

## Information Gathering

When the user does not specify components, analyze the codebase:
- Scan the package structure for BC packages (children of the top-level application package containing boundary/control/entity sub-packages)
- Identify cross-BC interactions via injected references, REST client calls, or messaging
- Determine external dependencies (third-party APIs, cloud services, databases)
