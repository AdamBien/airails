---
name: bce
description: Generic, composable architecture rules for the Boundary-Control-Entity (BCE/ECB) pattern — business components, layer responsibilities, package structure, and cross-component relationships. Technology-neutral; meant to be composed with language- or framework-specific skills (e.g. microprofile-server, web-components, aws-cdk, java-cli-app). Use when creating, generating, scaffolding, writing, or reviewing code organized as business components with boundary/control/entity layers. Triggers on "BCE", "ECB", "Boundary-Control-Entity", "business component", "BC layout", "BC structure", "boundary layer", "control layer", "entity layer", or requests to organize, package, refactor, or review code along BCE lines.
---

## Pattern

- derived from Ivar Jacobson's Boundary-Control-Entity (OOSE, 1992), also written Entity-Control-Boundary (ECB)
- Jacobson's stereotypes classify the analysis objects of a single use case; here they are the layers of a business component and map to packages — the reading documented at bce.design
- the unit of organization is the business component (BC): it owns one responsibility and consists of the layers boundary, control, entity — only those it needs
- dependencies point downward only: boundary may use control and entity, control may use entity; never upward

## Package / Directory Structure

- path: `[ORGANIZATION].[PROJECT].[BC].[boundary|control|entity]`; the top-level package reflects the application's name or responsibility
- BCs are direct children of the top-level package; `boundary`, `control`, `entity` appear only inside a BC — unlike package-by-layer, where technical packages (`controllers`, `services`, `repositories`, `components`) form the top level and one responsibility is spread across them
- reserve the top-level package for trivial single-class plumbing; anything with domain or protocol semantics, more than one operation, or expected growth is a BC

## Business Components (BC)

- a BC is a domain concept, or a shared concern with a name that is reused by other BCs
- a control-only BC is valid when its responsibility is procedural; a boundary exists only when a facade is justified — unlike a layered stack, where every feature gets every layer and a missing one is filled with a pass-through
- BCs may depend on each other in both directions, including entity-to-entity references (unlike a strictly acyclic module graph, or DDD aggregates that reference each other by id only); aim for maximal cohesion and minimal coupling — excessive cross-BC references or shared configuration are a signal to split, merge, or rebalance
- create new BCs with minimal logic and essential fields only

## Boundary Layer

- boundary classes are coarse-grained facades that adapt external protocols, transports, or UI events to internal operations
- they are the only entry points from outside the system; UI, protocols, transports, and system tests never reach control or entity directly
- cross-cutting concerns that wrap an operation (transactions, authorization checks, request/response mapping) live here, never in control or entity — unlike a service layer, which declares them on the services

## Control Layer

- procedural business logic in stateless, function-like units
- callable by the BC's own boundary and directly by other BCs — unlike a modular monolith, where a module is reached only through its facade or API package
- a control method is public when called from outside the control package, package-private (or the language equivalent) otherwise; visibility does not express a cross-BC contract
- outbound integration is a control concern: a client to an external system (remote API, message producer, third-party service) is owned and wrapped by a control in the BC that needs it; the boundary remains the inbound perimeter and never calls out itself; unlike ports-and-adapters, the client is not a driven adapter behind a port in a separate `adapter` or `infrastructure` package; the stack skill names the client technology

## Entity Layer

- domain objects that hold state and the behavior on that state; never anemic data holders — unlike the transaction-script style, where entities are getters and setters and their logic sits in services
- model enumerable value objects as enums or the language's closed-set equivalent

## Naming

- name packages, classes, modules, and files after their responsibilities, never after technical concerns
- no generic or technical BC names: `util`, `common`, `model`, `dto`, `services`, `helpers`, `api`, `impl`, `core`, `app`, `base`, `shared`, `domain`, `data`, `logic`; a shared concern becomes a BC only under the name the other BCs use for it (e.g. `pricing`, not `common`)
- no meaningless suffixes: `*Impl`, `*Service`, `*Manager`, `*Creator`, `*Control`
- protocol or pattern suffixes (`Resource`, `Factory`, `Builder`) only on elements that fulfill that role

## Documentation

- every BC has a package doc; it states the BC's responsibility and design decisions, never its contents; the stack skill names the file and format
- never restate the BCE pattern in generated documentation

## Composition with Other Skills

- this skill defines only the pattern; language, framework, build, testing, persistence, and protocol rules come from the composed skill (e.g. `microprofile-server`, `web-components`, `aws-cdk`, `java-cli-app`)
- a composed skill specializes these rules (e.g. "JAX-RS resources are boundary classes"), never contradicts them
