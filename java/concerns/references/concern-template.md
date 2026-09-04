# Concern.java Template

One file in the project root package. Adjust the package name; keep the shape. The reference
implementation lives in [zsmith](https://github.com/AdamBien/zsmith).

```java
package airhacks.myapp;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import static java.lang.annotation.ElementType.PACKAGE;
import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.SOURCE;

/// Why this type or package exists, when the reason is a technical concern rather than a
/// business component's responsibility. Marks only what would not exist without the concern —
/// a domain class that emits an event participates in observability, it is not observability.
/// An import says what a class touches; this says what it is for.
///
/// On a package when the whole business component serves the concern, on a type when it is
/// scattered across components that serve something else. Never the same kind twice for one
/// class; a type may carry a kind its package does not declare.
///
/// Retained in source only: every consumer — the reader, the agent, javadoc — reads the
/// source, and the jar stays free of the metadata.
@Documented
@Retention(SOURCE)
@Target({TYPE, PACKAGE})
public @interface Concern {

    Kind value();

    /// A kind is admitted only when its members are few and would not exist without the
    /// concern. Each kind states what does not qualify — the exclusions carry the information.
    enum Kind {

        /// Exists to record what the process did — whatever the transport: JFR events, OTEL
        /// spans and metrics, dedicated diagnostic loggers, and the components that capture
        /// and read them back. The emitting API is not the criterion. A class that emits or
        /// logs in passing is not this.
        OBSERVABILITY,
        /// Exists to communicate with a system that has its own lifecycle and failure modes —
        /// another process, a remote service, or a foreign in-process engine. Ownership is not
        /// the criterion: the project's own server counts. Not this: the class that owns the
        /// shared HttpClient, or a class that merely triggers the call.
        EXTERNAL_SYSTEM
    }
}
```

## Type-Level Marker

For a member scattered in a BC that serves something else — here a JFR event entity; an OTEL
exporter or a provider transport is marked the same way:

```java
package airhacks.myapp.orders.entity;

import jdk.jfr.Category;
import jdk.jfr.Event;
import jdk.jfr.Label;

import airhacks.myapp.Concern;
import static airhacks.myapp.Concern.Kind.OBSERVABILITY;

@Concern(OBSERVABILITY)
@Label("Order Placed")
@Category({"myapp", "orders"})
public class OrderPlacedEvent extends Event {
    ...
}
```

## Package-Level Marker

For a BC that wholly serves the concern — the annotation sits above the `package` declaration
in `package-info.java`, its imports below:

```java
/// Answers what each run cost and where it went wrong.
@Concern(OBSERVABILITY)
package airhacks.myapp.telemetry;

import airhacks.myapp.Concern;
import static airhacks.myapp.Concern.Kind.OBSERVABILITY;
```

No type inside `airhacks.myapp.telemetry` repeats `OBSERVABILITY`; a type there may still
carry a different kind.
