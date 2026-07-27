# Project blueprint — a BCE CDK app

A domain-neutral walkthrough of a Java AWS CDK v2 app structured as BCE business components. The example domain (an `orders` service with a DynamoDB data store and a Lambda-backed API) is illustrative — **copy the shapes, not the domain**. Any AWS capability maps onto the same layout: a thin stack in `boundary`, reusable resource factories in `control`, immutable data in `entity`.

The two business components also demonstrate both naming rules: `dynamodb` provisions a single service and is named after it; `ordering` spans Lambda and API Gateway and is therefore named after its domain responsibility, never after a technical layer (`api`, `data`).

A compilable starter template is available at [AdamBien/aws-cdk-plain](https://github.com/AdamBien/aws-cdk-plain) — clone it to start a project, then grow it into the shapes below. When template and blueprint disagree, this blueprint governs.

## Contents

- [Project layout](#project-layout)
- [Build files](#build-files-pomxml--cdkjson)
- [Application entry point](#application-entry-point)
- [Stacks interface — region pinning](#stacks-interface--region-pinning)
- [Configuration — typed records](#configuration--typed-records)
- [Boundary — a thin stack](#boundary--a-thin-stack)
- [Control — reusable factories and permissions](#control--reusable-factories-and-permissions)
- [Entity — immutable data](#entity--immutable-data)
- [Tests — synthesis and parameterized](#tests--synthesis-and-parameterized)
- [Helper scripts](#helper-scripts)

## Project layout

Packages are BCE business components — named after the single AWS capability they provision, or after their domain responsibility when they span multiple services. `boundary` holds stacks, `control` holds reusable factories, `entity` holds immutable data. A component uses only the layers it needs — a stateless factory is a `control`-only component.

```
src/main/java/
  com/acme/                                     ← organization root
    CDKApp.java                                 ← app entry point (appName constant lives here)
    orders/                                     ← application package [organization].[application]
      Configuration.java                        ← typed config records (shared concern)
      Stacks.java                               ← pinned StackProps per region
      dynamodb/boundary/DynamoDBStack.java      ← stateful deployment boundary (single service)
      dynamodb/control/Tables.java              ← DynamoDB table factory
      ordering/boundary/OrderingStack.java      ← application deployment boundary (multi-service)
      ordering/control/Functions.java           ← Lambda + grants factory
      ordering/entity/RouteKey.java             ← value object
      iam/Roles.java                            ← shared IAM (no BCE substructure needed)
src/test/java/
  com/acme/orders/
    dynamodb/boundary/DynamoDBStackTest.java    ← synthesis test
    ordering/entity/RouteKeyTest.java           ← parameterized test
```

Business components (`dynamodb`, `ordering`, `iam`) are direct children of the **application package** `com.acme.orders` (`[organization].[application]`), not of the organization. `CDKApp` sits at the organization root (keeping it in the root is fine); the deployment name `acme-orders` is carried by its `appName` constant — distinct from the `orders` application package segment.

## Build files (pom.xml + cdk.json)

`pom.xml` — Maven, Java 25, `exec-maven-plugin` pointing at the app class; only `aws-cdk-lib` + `constructs` at runtime, JUnit 5 + AssertJ for tests:

```xml
<project ...>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>orders-infrastructure</artifactId>
  <version>0.1</version>
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>3.1.0</version>
        <configuration>
          <mainClass>com.acme.CDKApp</mainClass>
        </configuration>
      </plugin>
    </plugins>
  </build>
  <dependencies>
    <dependency>
      <groupId>software.amazon.awscdk</groupId>
      <artifactId>aws-cdk-lib</artifactId>
      <version>2.261.0</version>
    </dependency>
    <dependency>
      <groupId>software.constructs</groupId>
      <artifactId>constructs</artifactId>
      <version>10.6.0</version>
    </dependency>
    <!-- junit-jupiter-api/engine/params + assertj-core, scope test -->
  </dependencies>
  <properties>
    <maven.compiler.source>25</maven.compiler.source>
    <maven.compiler.target>25</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
</project>
```

`cdk.json` — the CLI drives Maven; enable cross-stack references only if a stack consumes another region's resource:

```json
{
  "app": "mvn -e -q compile exec:java",
  "context": { "@aws-cdk/core:defaultCrossStackReferences": "both" }
}
```

## Application entry point

The entry point is the only place that reads context/env and configuration. It tags the app, instantiates stacks in dependency order, passes **resource interfaces** (never stack instances) from producer to consumer, and synthesizes.

```java
public interface CDKApp {
    String appName = "acme-orders";   // application identity — not encoded in a package or the class name

    static void main(String... args) {
        var app = new App();
        var environmentName = context(app, "env", "dev");   // reads CDK context
        var configuration = Configuration.of(appName, environmentName);

        Tags.of(app).add("acme:application-id", "orders");
        Tags.of(app).add("acme:environment-type", environmentName);
        Tags.of(app).add("acme:managed-by", "cdk");

        var dynamoDBStack = new DynamoDBStack(app, configuration);
        new OrderingStack(app, configuration, dynamoDBStack.getOrdersTable());  // pass ITable, not the stack
        app.synth();
    }

    static String context(App app, String key, String fallback) {
        var value = (String) app.getNode().tryGetContext(key);
        return (value == null || value.isBlank()) ? fallback : value;
    }
}
```

## Stacks interface — region pinning

One `StackProps` constant per region; never inline `StackProps` in a stack. Set `crossRegionReferences(true)` only on a stack that consumes another region's resource (a classic case: a global service pinned to `us-east-1` whose output is consumed by a stack in the primary region).

```java
public interface Stacks {
    StackProps PRIMARY = StackProps.builder()
            .env(Environment.builder().region("eu-central-1").build())
            .build();

    StackProps US_EAST_1 = StackProps.builder()
            .crossRegionReferences(true)
            .env(Environment.builder().region("us-east-1").build())
            .build();
}
```

## Configuration — typed records

A `Configuration` interface groups immutable records and `static` factory methods that parse the raw source (properties, context, env). Records carry domain behavior — derived names, normalization, copy-with methods — rather than anemic getters. A stack never reaches back to the raw source; it receives a record.

```java
public interface Configuration {
    record OrdersConfiguration(String appName, String environmentName) {
        // "acme-orders-ordering-dev"
        String stackName(String component) {
            return "%s-%s-%s".formatted(appName, component, environmentName);
        }
    }

    static OrdersConfiguration of(String appName, String environmentName) {
        return new OrdersConfiguration(appName, environmentName);
    }
}
```

## Boundary — a thin stack

A stack derives its CloudFormation name from configuration, delegates resource creation to `control` factories, wires cross-stack dependencies, and exposes only interfaces via getters. It holds almost no resource configuration itself. Stateful stacks are isolated and their resources retained.

```java
public class DynamoDBStack extends Stack {
    static String component = "dynamodb";
    ITable ordersTable;

    public DynamoDBStack(Construct scope, OrdersConfiguration configuration) {
        super(scope, "DynamoDB",                                    // stable construct id
                Stacks.PRIMARY.toBuilder()
                        .stackName(configuration.stackName(component)).build());
        this.ordersTable = Tables.createOrdersTable(this);
    }

    public ITable getOrdersTable() { return ordersTable; }          // expose the interface
}
```

The consuming stack accepts the interface and passes it into its own factory — CDK derives the cross-stack reference from the object:

```java
public class OrderingStack extends Stack {
    static String component = "ordering";

    public OrderingStack(Construct scope, OrdersConfiguration configuration, ITable ordersTable) {
        super(scope, "Ordering",
                Stacks.PRIMARY.toBuilder()
                        .stackName(configuration.stackName(component)).build());
        var handler = Functions.createApiHandler(this, ordersTable);
        CfnOutput.Builder.create(this, "ApiHandlerArn")
                .value(handler.getFunctionArn()).build();
    }
}
```

## Control — reusable factories and permissions

Stateless factories as `interface` + `static` methods: `Builder.create`, stable PascalCase logical ids, no explicit physical name (let CloudFormation generate it), `RemovalPolicy.RETAIN` for stateful resources.

```java
public interface Tables {
    static ITable createOrdersTable(Construct scope) {
        return Table.Builder.create(scope, "OrdersTable")            // logical id = deployment identity
                .partitionKey(Attribute.builder()
                        .name("orderId").type(AttributeType.STRING).build())
                .billingMode(BillingMode.PAY_PER_REQUEST)
                .removalPolicy(RemovalPolicy.RETAIN)                 // protect data on stack delete
                .build();                                            // no tableName → generated
    }
}
```

Permissions use `grant*` on the target construct — least privilege, no hand-written policies. Accept and return **interfaces** so consumers depend on behavior, not the concrete class:

```java
public interface Functions {
    static IFunction createApiHandler(Construct scope, ITable ordersTable) {
        var handler = Function.Builder.create(scope, "ApiHandler")
                .runtime(Runtime.JAVA_21)
                .handler("com.acme.Handler")
                .code(Code.fromAsset("target/handler.jar"))
                .build();
        ordersTable.grantReadWriteData(handler);                    // scoped, least-privilege
        return handler;
    }
}
```

Fall back to an explicit `PolicyStatement.Builder` / `Role.Builder` only when no `grant*` expresses the permission, and keep such roles in a dedicated `iam` component. Give Cognito its own stack.

## Entity — immutable data

`entity` produces data, never resources — value objects, closed sets, and pure spec builders. Being pure, this logic is trivially unit-testable without synthesis.

```java
public enum RouteKey {
    GET_ORDERS("GET", "/orders"),
    CREATE_ORDER("POST", "/orders");

    final String method;
    final String path;

    RouteKey(String method, String path) { this.method = method; this.path = path; }

    // "GET /orders" — the API Gateway route key format
    public String value() { return "%s %s".formatted(method, path); }
}
```

## Tests — synthesis and parameterized

**Synthesis test** — build an `App`, instantiate the stack, `synth()`, and assert the template has the expected resources (extend with property, count, and grant assertions where they matter):

```java
class DynamoDBStackTest {
    static final ObjectMapper JSON =
            new ObjectMapper().configure(SerializationFeature.INDENT_OUTPUT, true);

    @Test
    void ordersTableIsSynthesized() {
        var app = new App();
        var configuration = Configuration.of("acme-orders", "test");
        var stack = new DynamoDBStack(app, configuration);
        var template = JSON.valueToTree(
                app.synth().getStackArtifact(stack.getArtifactId()).getTemplate());
        assertThat(template.get("Resources")).isNotEmpty();
    }
}
```

**Parameterized test** for pure `entity` logic — one labelled row per requirement, no synthesis needed. When the component carries EARS requirements in its `package-info.java`, compose with `ears-tests` to generate the rows:

```java
class RouteKeyTest {
    static Stream<Arguments> routeKeyValues() {
        return Stream.of(
                Arguments.of(RouteKey.GET_ORDERS, "GET /orders"),
                Arguments.of(RouteKey.CREATE_ORDER, "POST /orders"));
    }

    @ParameterizedTest(name = "{0} → {1}")
    @MethodSource
    void routeKeyValues(RouteKey key, String expected) {
        assertThat(key.value()).isEqualTo(expected);
    }
}
```

## Helper scripts

```sh
# buildAndDeploy.sh
mvn -DskipTests clean package && cdk deploy --all --context env=dev
# buildAndDeployDontAsk.sh
mvn -DskipTests clean package && cdk deploy --all --context env=dev --require-approval=never
# destroy.sh
cdk destroy --all --context env=dev
```

Bootstrap each pinned region once before the first deploy:

```sh
cdk bootstrap aws://ACCOUNT/eu-central-1
cdk bootstrap aws://ACCOUNT/us-east-1   # only if a stack is pinned there
```
