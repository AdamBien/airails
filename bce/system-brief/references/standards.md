# Standards behind the system brief

The brief is written for a product owner and uses plain section names. Each section, however,
adopts the definition of one established standard, so the skill reasons with terms it already
knows and the stakeholders never see them. This file states which standard owns which section,
the term adopted, and the definition the skill applies. Where two standards compete, the one
with the sharper definition wins; the other is named for orientation only.

## Section ownership

| Brief section | Standard | Adopted term | Definition applied |
|---|---|---|---|
| System · mission | OMG Business Motivation Model (BMM) | Mission | the ongoing operational activity of the enterprise that makes the vision a reality; stated from the system's side, one sentence |
| System · vision | BMM | Vision | the desired future state, an outcome; it names no means and no feature |
| Actors · primary | UML 2.5 Use Cases; Cockburn, *Writing Effective Use Cases* | Actor, primary actor | an entity outside the Subject that interacts with it to reach a goal; primary when the goal is theirs |
| Actors · external | UML 2.5 Use Cases; Cockburn | supporting actor | an entity outside the Subject the Subject relies on to deliver the goal; here always another system |
| the system itself | UML 2.5 Use Cases | Subject | the system under consideration; the boundary the Actors stand outside of |
| Capabilities | ArchiMate 3.2, Strategy layer | Capability | an ability that an active structure element, here the system, possesses; described by what it does, never by how; one capability map per system |
| Capabilities · altitude | Cockburn goal levels | kite / cloud level | a capability line is a summary-level goal; sea-level goals are use cases and belong to `/capability-brief` |
| Interactions | Evans, *Domain-Driven Design*, context map | upstream / downstream | a dependency between two bounded contexts with a direction and what crosses it; ArchiMate `serving` relationship for orientation |
| Language | Evans, DDD | ubiquitous language | one meaning per noun within the system; a noun with a second meaning marks a context boundary and is a decision |
| Invariants | ArchiMate 3.2, Motivation layer | Principle | a general rule that holds across the whole architecture, stated as intent, not as a requirement of one part; BMM Business Policy for orientation |
| Success measures | ArchiMate 3.2, Motivation layer | Outcome | an end result that has been achieved, observable, attributable to the system; BMM Objective when it carries a target value |
| Out of scope | ISO/IEC/IEEE 29148, Stakeholder Requirements Specification | system boundary, exclusions | what lies outside the Subject, named with the reason it was excluded |
| Decisions | ISO/IEC/IEEE 42010 | architecture decision, rationale | a choice made against named alternatives, with the reason; the rejected alternative is part of the record |
| Open issues | ISO/IEC/IEEE 42010 | concern | an interest in the system held by a named stakeholder, unresolved; recorded in the stakeholder's words |
| the whole document | ISO/IEC/IEEE 29148 | Stakeholder Requirements Specification (StRS) | purpose, scope, stakeholders, boundary, external interfaces, exclusions; the brief is the StRS compressed to inception size |

## Stakeholder versus actor

ArchiMate separates the **Stakeholder**, who holds concerns and is interviewed, from the
**Business Actor**, who performs behaviour. The skill keeps that line: *stakeholders* answer the
interview, sign off the carving and own the open issues; *actors* appear in the brief as the
people and systems that use or support the Subject. A stakeholder may also be an actor; the two
words are never swapped.

## What the standards do not supply

The following are the skill's own rules; no adopted standard defines them.

- the `new` / `extends` / `existing` tags on a capability line
- one lowercase word as a capability name, and one capability equals one business component
- build order with a stated reason for the first capability (TOGAF transition architectures are the nearest notion)
- the `record: <choice> _(rejected: …)_` line form, a compressed Architecture Decision Record
- the interview protocol: fill from the stakeholders' words, read back, propose the carving with the rejected alternative, park what nobody can answer
- the status of the brief as inception input, not a source of truth

## Considered and not adopted

- DoDAF / UAF capability views (CV-1, CV-2, CV-4) fit the sections but carry a defence vocabulary and are known less widely
- BIZBOK capability map: the closest in spirit, but the guide is not openly published, so its definitions are second-hand
- TOGAF: its method would leak into the skill; only the transition-architecture notion is borrowed for build order
- SysML v2: stakeholder, concern and use case fit, but the vocabulary is too new to be relied on
- UML alone: it supplies Actor and Subject only; it has no mission, vision, capability or decision

## Sources

- ArchiMate 3.2 Specification, The Open Group, chapters 6 (Motivation) and 7 (Strategy)
- Business Motivation Model 1.3, Object Management Group
- OMG Unified Modeling Language 2.5.1, clause 18 Use Cases
- Alistair Cockburn, *Writing Effective Use Cases*, 2001: actor roles and goal levels
- Eric Evans, *Domain-Driven Design*, 2003: ubiquitous language, bounded context, context map
- ISO/IEC/IEEE 42010:2022, Software, systems and enterprise — Architecture description
- ISO/IEC/IEEE 29148:2018, Systems and software engineering — Life cycle processes — Requirements engineering
