---
name: bce
description: Generic, composable architecture rules for the Boundary-Control-Entity (BCE/ECB) pattern — business components, layer responsibilities, package structure, and cross-component relationships. Technology-neutral; meant to be composed with language- or framework-specific skills (e.g. microprofile-server, web-components, aws-cdk, java-cli-app). Use when creating, generating, scaffolding, writing, or reviewing code organized as business components with boundary/control/entity layers. Triggers on "BCE", "ECB", "Boundary-Control-Entity", "business component", "BC layout", "BC structure", "boundary layer", "control layer", "entity layer", or requests to organize, package, refactor, or review code along BCE lines.
---

## Pattern

- derived from Ivar Jacobson's Boundary-Control-Entity (OOSE, 1992)
- the unit of organization is the business component (BC): it owns one responsibility and consists of the layers boundary, control, entity — only those it needs
- dependencies point downward only: boundary may use control and entity, control may use entity; never upward

## Package / Directory Structure

- path: `[ORGANIZATION].[PROJECT].[BC].[boundary|control|entity]`; the top-level package reflects the application's name or responsibility
- BCs are direct children of the top-level package; `boundary`, `control`, `entity` appear only inside a BC
- reserve the top-level package for trivial single-class plumbing; anything with domain or protocol semantics, more than one operation, or expected growth is a BC

## Business Components (BC)

- a BC is a domain concept, or a shared concern with a name that is reused by other BCs
- a control-only BC is valid when its responsibility is procedural; a boundary exists only when a facade is justified
- BCs may depend on each other in both directions, including entity-to-entity references; aim for maximal cohesion and minimal coupling — excessive cross-BC references or shared configuration are a signal to split, merge, or rebalance
- create new BCs with minimal logic and essential fields only

## Boundary Layer

- boundary classes are coarse-grained facades that adapt external protocols, transports, or UI events to internal operations
- they are the only entry points from outside the system; UI, protocols, transports, and system tests never reach control or entity directly
- cross-cutting concerns that wrap an operation (transactions, authorization checks, request/response mapping) live here, never in control or entity

## Control Layer

- procedural business logic in stateless, function-like units
- callable by the BC's own boundary and directly by other BCs
- a control method is public when called from outside the control package, package-private (or the language equivalent) otherwise; visibility does not express a cross-BC contract

## Entity Layer

- domain objects that hold state and the behavior on that state; never anemic data holders
- model enumerable value objects as enums or the language's closed-set equivalent

## Naming

- name packages, classes, modules, and files after their responsibilities, never after technical concerns
- no meaningless suffixes: `*Impl`, `*Service`, `*Manager`, `*Creator`, `*Control`
- protocol or pattern suffixes (`Resource`, `Factory`, `Builder`) only on elements that fulfill that role

## Documentation

- a BC's package doc states its responsibility and design decisions, never its contents; the stack skill names the file and format
- never restate the BCE pattern in generated documentation

## Composition with Other Skills

- this skill defines only the pattern; language, framework, build, testing, persistence, and protocol rules come from the composed skill (e.g. `microprofile-server`, `web-components`, `aws-cdk`, `java-cli-app`)
- a composed skill specializes these rules (e.g. "JAX-RS resources are boundary classes"), never contradicts them
