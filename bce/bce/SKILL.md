---
name: bce
description: Generic, composable architecture rules for the Boundary-Control-Entity (BCE/ECB) pattern — business components, layer responsibilities, package structure, and cross-component relationships. Technology-neutral; meant to be composed with language- or framework-specific skills (e.g. microprofile-server, web-components, aws-cdk, java-cli-app). Use when creating, generating, scaffolding, writing, or reviewing code organized as business components with boundary/control/entity layers. Triggers on "BCE", "ECB", "Boundary-Control-Entity", "business component", "BC layout", "BC structure", "boundary layer", "control layer", "entity layer", or requests to organize, package, refactor, or review code along BCE lines.
---

## Pattern

- structure code using the Boundary-Control-Entity (BCE/ECB) pattern
- the unit of organization is the business component (BC); each BC owns a responsibility and is composed of one or more layers
- the three layers are boundary, control, entity; each has a distinct responsibility (see layer sections below)
- do not explain the BCE pattern in generated documentation

## Package / Directory Structure

- package or directory path: `[ORGANIZATION].[PROJECT].[BC].[boundary|control|entity]`
- the top-level package reflects the application responsibility or name
- business components are direct children of the top-level package and named after their responsibilities
- the `boundary`, `control`, `entity` segments are only allowed inside a business component
- name packages after their domain responsibilities, not technical concerns

## Business Components (BC)

- a BC may represent a domain concept or a shared concern; both are valid when the responsibility has a name and is reused
- a BC does not need every layer; a BC may consist of only a control layer when its responsibility is procedural and consumed by other BCs
- not every BC needs a dedicated boundary; control contents may be consumed directly when no facade is justified
- prefer a dedicated BC over the root application package when a shared concern carries domain or protocol semantics, exposes more than one operation, or is expected to grow
- reserve the root application package for trivial single-class plumbing with no business semantics and no protocol coupling
- create new BCs with minimal logic and essential fields only

## Boundary Layer

- keep coarse-grained classes in the boundary
- place facades that adapt external protocols, transports, or UI events to internal operations in the boundary
- boundary classes are the only entry points called from outside the system; external actors (UI, protocols, transports, tests) never reach control or entity directly
- cross-cutting concerns that wrap an operation (transactions, authorization checks, request/response mapping) belong in the boundary, not in control or entity

## Control Layer

- implement procedural business logic in the control layer
- prefer stateless, function-like units for procedural logic
- control may be called by the boundary of the same BC or directly by other BCs; the boundary is not a gate for cross-BC calls

## Entity Layer

- maintain domain objects, data classes, and entities in the entity layer
- entities maintain state and corresponding behavior; they are not anemic data holders
- model value objects as enums or equivalent closed sets where the language supports them
- direct references between entities from independent BCs are allowed, but always aim for Maximal Cohesion and Minimal Coupling between BCs
- if a relation exists in the persistent store (e.g. foreign key), the entities must carry a corresponding reference (id field or association); the schema is the source of truth
- excessive cross-BC references or shared configuration is a refactoring signal — split, merge, or rebalance the BCs to restore cohesion

## Naming

- name classes, modules, and files after their responsibilities
- avoid meaningless suffixes (e.g. `*Impl`, `*Service`, `*Manager`, `*Creator`)
- a class or module name must not end with `Control`
- reserve protocol- or pattern-specific suffixes (e.g. `Resource`, `Factory`, `Builder`) for elements that actually fulfill that role

## Documentation

- document only domain-specific packages where the purpose is not self-evident
- when documenting a top-level package or BC, describe design decisions and responsibilities — not contents
- in Java, use `package-info.java` with JavaDoc to document packages
- do not write documentation that restates the BCE pattern itself

## Composition with Other Skills

- this skill defines only the architectural pattern; language, framework, build, testing, and protocol rules come from the composed skill (e.g. `microprofile-server`, `web-components`, `aws-cdk`, `java-cli-app`)
- when a composed skill adds a layer rule (e.g. "health checks belong in boundary", "JAX-RS resources are boundary classes"), apply it on top of the BCE rules above; the composed skill always specializes, never contradicts
