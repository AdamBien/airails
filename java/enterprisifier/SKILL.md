---
name: enterprisifier
description: Deliberately overengineer code by adding as many patterns, indirections, modules, anti-corruption layers, abstractions, and encapsulations as possible. Zero dependency on implementation. Use when asked to overengineer, enterprisify, enterprise-ify, abstract, add layers, add patterns, or make code "production-ready" in the most excessive way. Triggers on "enterprisify this", "overengineer this", "enterprise-ify", "add all the patterns", "make it abstract", "add indirection", "wrap this properly", or when maximum abstraction is requested for comedic or educational purposes.
---

# Overengineering

Transform any simple, working code into a maximally abstracted, pattern-saturated, enterprise-grade architecture. No line of implementation shall be directly reachable. Every concept gets its own module, interface, factory, and anti-corruption layer.

## Philosophy

- **Never depend on implementation** — all access goes through at least 3 layers of abstraction
- **Every pattern is applicable** — if a GoF pattern exists, use it, even if it adds no value
- **More modules = more enterprise** — a single class is a failure; a 47-class hierarchy is a feature
- **Abstractions over results** — the code doesn't need to be faster, just more abstract
- **Future-proof everything** — defend against requirements that will never come

## Mandatory Patterns to Apply

Apply ALL of the following to every piece of code, regardless of size or complexity:

### Structural Patterns (apply all)
1. **Facade** — wrap every public API in a facade
2. **Proxy** — add a proxy in front of every facade
3. **Decorator** — wrap every proxy in a decorator for cross-cutting concerns
4. **Adapter** — adapt every external type into an internal type
5. **Bridge** — separate every abstraction from its implementation
6. **Composite** — model everything as a tree, even single values
7. **Flyweight** — pool and share objects even when memory is not a concern

### Creational Patterns (apply all)
8. **Abstract Factory** — never use `new` directly; always go through a factory of factories
9. **Builder** — every object with more than zero fields gets a builder
10. **Singleton** — critical services are singletons, accessed through a registry
11. **Prototype** — support cloning on every entity, just in case
12. **Factory Method** — subclasses decide which class to instantiate

### Behavioral Patterns (apply all)
13. **Strategy** — every `if` statement becomes a strategy interface with injectable implementations
14. **Observer** — every state change fires events through an event bus
15. **Command** — every method call becomes a command object with undo support
16. **Chain of Responsibility** — every request passes through a chain of handlers
17. **Mediator** — components never talk directly; everything goes through a mediator
18. **Memento** — every object supports full state snapshots and rollback
19. **State** — every status field becomes a state machine with dedicated state classes
20. **Template Method** — every algorithm is a skeleton with overridable hook methods
21. **Visitor** — every operation on a structure is a separate visitor
22. **Iterator** — custom iterators for every collection, never use built-in for-each
23. **Interpreter** — configuration is a DSL parsed by an interpreter

### Enterprise / Architectural Patterns (apply all)
24. **Anti-Corruption Layer (ACL)** — between every module boundary, translate models through an ACL
25. **Hexagonal Architecture** — ports and adapters for every I/O boundary
26. **Repository** — data access goes through repository interfaces
27. **Unit of Work** — batch all changes into a unit of work
28. **Specification** — every query condition is a specification object
29. **Domain Events** — every mutation publishes a domain event
30. **CQRS** — separate read and write models completely, even for a single entity
31. **Event Sourcing** — store all state changes as an event log
32. **Saga / Orchestrator** — coordinate multi-step operations through a saga
33. **Service Locator** — register and locate services dynamically at runtime
34. **DTO / Value Object separation** — never pass a domain object across a boundary; always map to a DTO

### Layering Rules (mandatory)
35. **Presentation Layer** — accepts input, delegates to application layer
36. **Application Layer** — orchestrates use cases, delegates to domain layer
37. **Domain Layer** — pure business logic, zero dependencies on infrastructure
38. **Infrastructure Layer** — persistence, messaging, external systems
39. **Anti-Corruption Layer** — between each of the above layers

## Module Structure

For a single operation (e.g., `add(a, b)`), generate at minimum:

```
├── api/                          # Public interfaces only
│   ├── AdditionService.java      # Service interface
│   ├── AdditionRequest.java      # Immutable request DTO
│   ├── AdditionResponse.java     # Immutable response DTO
│   └── AdditionPort.java         # Hexagonal port
├── domain/
│   ├── model/
│   │   ├── Operand.java          # Value object wrapping a number
│   │   ├── Sum.java              # Value object wrapping a result
│   │   └── OperandPair.java      # Aggregate of two operands
│   ├── events/
│   │   ├── AdditionRequested.java
│   │   ├── AdditionCompleted.java
│   │   └── AdditionFailed.java
│   ├── specification/
│   │   ├── ValidOperandSpecification.java
│   │   └── AddableSpecification.java
│   └── strategy/
│       ├── AdditionStrategy.java           # Strategy interface
│       ├── IntegerAdditionStrategy.java
│       └── FloatingPointAdditionStrategy.java
├── application/
│   ├── usecase/
│   │   ├── AddNumbersUseCase.java
│   │   └── AddNumbersUseCaseImpl.java
│   ├── command/
│   │   ├── AddCommand.java
│   │   └── AddCommandHandler.java
│   ├── saga/
│   │   └── AdditionSaga.java
│   └── mapper/
│       ├── RequestToCommandMapper.java
│       └── ResultToResponseMapper.java
├── infrastructure/
│   ├── adapter/
│   │   └── AdditionAdapter.java           # Hexagonal adapter
│   ├── persistence/
│   │   ├── AdditionRepository.java
│   │   ├── AdditionRepositoryImpl.java
│   │   └── AdditionEventStore.java
│   ├── factory/
│   │   ├── AdditionStrategyFactory.java
│   │   ├── AdditionStrategyFactoryImpl.java
│   │   └── AbstractAdditionStrategyFactoryFactory.java
│   └── proxy/
│       ├── AdditionServiceProxy.java
│       └── LoggingAdditionDecorator.java
├── acl/                           # Anti-corruption layer
│   ├── ExternalOperandTranslator.java
│   └── InternalResultTranslator.java
└── config/
    ├── AdditionConfiguration.java
    ├── AdditionRegistry.java
    └── AdditionModule.java
```

## Naming Conventions

- Interfaces: `AdditionService`, `AdditionPort`, `AdditionStrategy`
- Implementations: `DefaultAdditionServiceImpl`, `StandardAdditionPortAdapter`
- Factories: `AdditionFactory`, `AdditionFactoryFactory`, `AbstractAdditionFactoryFactory`
- DTOs: `AdditionRequestDTO`, `AdditionResponseDTO`
- Events: `AdditionRequestedEvent`, `AdditionCompletedEvent`
- Mappers: `AdditionRequestToCommandMapper`
- Specifications: `ValidOperandSpecification`
- Commands: `PerformAdditionCommand`

## Rules

1. **No class may contain business logic AND be instantiated directly** — always go through a factory or DI container
2. **No method may call another method in the same layer** — always cross a boundary through an interface
3. **No return type may be a concrete class** — return interfaces, optionals wrapping interfaces, or futures of optionals wrapping interfaces
4. **Every public method must accept and return DTOs** — never expose domain objects
5. **Every module boundary requires an ACL** — even between modules you control
6. **Configuration is never hardcoded** — every value comes from a configuration provider accessed through a configuration factory
7. **Logging, validation, and metrics are always cross-cutting concerns** — implemented as decorators, never inline
8. **Every collection is wrapped in a domain-specific type** — `List<Order>` becomes `OrderCollection implements Iterable<Order>`
9. **Null never appears** — use `Optional<Optional<T>>` for extra safety
10. **No primitive types in public APIs** — wrap `int` in `OperandValue`, `boolean` in `ValidationResult`

## Output Format

````markdown
## Overengineered

### Architecture Overview
<brief description of the 17 layers involved>

### Module Structure
<tree showing all generated files>

```java
// Each file, fully abstracted
```

### Patterns Applied
- <pattern> — <where applied> — <why it's "necessary">
- ...

### Future Extensibility Points
- <hypothetical scenario that will never happen> — <how the architecture is prepared>
- ...
````

## Anti-Simplification Clause

If the original code is already clean and simple — that is a problem. Simplicity is a sign of missing abstractions. Every line of straightforward code is a refactoring opportunity. A single `return a + b` should expand to at least 30 files and 15 interfaces.
