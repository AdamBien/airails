---
name: bce-diagrams
description: Create high-level overview diagrams showing interactions between business components (BCs), subsystems, services, or systems. Use when asked to create architecture overviews, BC interaction diagrams, subsystem diagrams, service interaction diagrams, system landscapes, or integration maps. Triggers on "diagram", "overview diagram", "BC interaction", "business component diagram", "subsystem diagram", "service interaction", "component interaction", "architecture overview", "system diagram", "integration diagram", or requests to visualize how BCs, subsystems, or services communicate. Not for detailed class diagrams or sequence diagrams.
argument-hint: "[services or components to visualize]"
---

# BCE Overview Diagrams

Create diagrams that show interactions and dependencies between business components (BCs), subsystems, and external services at a high level of abstraction. Based on BCE architecture (see [bce.design](https://bce.design)).

## BCE Terminology

- **Business Component (BC)**, **boundary**, **control**, **entity** — as defined by `/bce`; the stack skill names what fulfills each layer.
- **Subsystem** — a grouping of related BCs at a higher level of abstraction.
## Workflow

1. **Gather information** - Identify BCs, subsystems, and external services to visualize, from the first applicable source in "Information Gathering".
2. **Choose detail level and format** - Settle each from the request and its target first; ask only what remains open, in a single AskUserQuestion call, and skip the call when nothing does:
   - **Detail level**: settled when the request names it (e.g. "BC overview", "show the layers"). Otherwise ask: "Between components (Recommended)" (each BC is a single opaque node) or "Within components" (show the layers inside each BC)
   - **Format**: settled when the request names Mermaid or draw.io, or when the diagram is destined for a README.md in a GitHub project (Mermaid). Otherwise ask: "Mermaid (Recommended)" (text-based, version-control friendly, renders natively in GitHub/GitLab) or "draw.io" (visual editor, richer styling, exportable to PNG/SVG)
3. **Invoke the chosen skill** - Use the Skill tool to invoke either `mermaid` or `drawio` with the identified BCs, subsystems, interactions, and chosen detail level.

## Diagram Content Guidelines

### Between components

- Show BCs, subsystems, and external services — not classes, methods, or fields
- Represent each BC as a single node labeled by its responsibility: `Orders`, `Payments`, `Users`
- Group BCs into subsystems when they share a domain: `subgraph Billing [Orders, Payments, Invoicing]`
- Focus on interactions between BCs: which BC uses which BC, what data flows where
- Label arrows with the interaction type when not obvious: `-->|REST|`, `-->|async|`
- Show external dependencies (third-party APIs, databases, message brokers) as separate nodes

### Within components

- Represent each BC as a subgraph containing only the layers it has — a control-only BC shows a single control node; never draw a placeholder for a missing layer
- Show internal interactions: boundary → control → entity
- Show cross-BC interactions between the layers involved: a boundary or control uses another BC's control or entity, an entity only another BC's entity; never draw an edge into another BC's boundary
- External dependencies connect to the control that calls them, never to a boundary; inbound callers from outside the system (clients, UI, other systems) connect to a boundary

## Information Gathering

Take the BCs and their interactions from the first source that applies:

1. **User's description** — the components and interactions named in the request.
2. **Declared wiring** — a `## Components` section in the base package's doc (the `/sbce` system doc). Draw exactly the declared edges; never add an edge inferred from code. In "within components" mode, read from code only which layers a declared edge connects.
3. **Code scan** — only when neither exists:
   - Scan the package or directory structure (`[ORGANIZATION].[PROJECT].[BC].[boundary|control|entity]`, see `/bce`) for BCs: direct children of the top-level package that contain a `boundary`, `control`, or `entity` package or directory
   - Identify cross-BC interactions: references from one BC's code to another BC's control or entity, and messaging between BCs; the stack skill names how a reference appears in code
   - Determine external dependencies (third-party APIs, cloud services, databases)
