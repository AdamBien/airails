---
name: aws-cdk
description: Create and review Java AWS CDK v2 infrastructure code organized with the BCE (Boundary-Control-Entity) pattern — scaffold the project, wire the app entry point, pin regions in a central Stacks interface, model business components named after AWS services, and keep stacks thin over reusable constructs. Use whenever generating, scaffolding, writing, extending, or reviewing AWS CDK infrastructure-as-code in Java: new CDK apps, stacks, constructs, IAM grants, Step Functions, S3/CloudFront/DynamoDB/Cognito/Route53/CodePipeline resources, or CloudFormation authored in Java. Triggers on "AWS CDK", "CDK v2", "CDK app", "CDK stack", "CDK construct", "infrastructure as code in Java", "CloudFormation in Java", "provision AWS in Java", "Step Functions", or any request to model AWS resources programmatically in Java. Not for TypeScript/Python CDK, Terraform, Pulumi, or non-AWS infrastructure.
---

Create Java AWS CDK v2 infrastructure the same way a business application is built: as BCE business components, not as a flat pile of resources. A construct is a reusable capability; a stack is a deployment boundary. Keep that split and the code stays readable, testable, and refactorable.

## Composition

- compose with `bce` for the architecture pattern (business components, boundary/control/entity, package structure) — this skill specializes it for the AWS CDK context and never contradicts it
- compose with `java-conventions` for all language-level rules (modern Java 25 syntax, `var`, records, pattern matching, text blocks, naming, visibility, streams, exceptions)
- this skill adds only what is specific to authoring CDK: project layout, the app→stack→construct split, region pinning, typed configuration, `grant*` permissions, and synthesis-based tests

## What this skill produces

A Maven, Java 25 CDK v2 project whose packages are BCE business components. The entry-point class loads typed configuration, wires stacks, and calls `app.synth()`. Each business component is named after the AWS capability it provisions (`s3`, `cloudfront`, `certificate`, `route53`, `codebuild`, `cognito`, `iam`, `stepfunctions`). Stacks live in `boundary`; the reusable resource factories and helpers live in `control`; immutable specs and value objects live in `entity`.

A generic, domain-neutral blueprint — the BCE layout, build files, and one worked business component per layer — is in `references/project-blueprint.md`. Read it when scaffolding a new project or when you need a concrete template for a stack, a control factory, an entity, or the tests; copy the shapes, not the example domain. Read `references/naming-conventions.md` for the full CloudFormation / CDK / Java naming tables — casing rules for logical IDs, stack names, tags, and construct IDs.

## Scaffolding a new project

A clone of [AdamBien/aws-cdk-plain](https://github.com/AdamBien/aws-cdk-plain) is a valid starting point; otherwise generate the files directly. Either way, work in this order — it mirrors the dependency direction of synthesis:

1. **`pom.xml` + `cdk.json`** — Maven with `aws-cdk-lib` and `constructs`, Java 25 source/target, the `exec-maven-plugin` pointing `mainClass` at the app class. `cdk.json` sets `"app": "mvn -e -q compile exec:java"`. Templates in `references/project-blueprint.md`. Ask before adding any dependency beyond these.
2. **Application entry point** — the `CDKApp` class at the root. It reads CDK context, loads configuration, applies app-level tags, instantiates stacks in dependency order, and calls `app.synth()`. Its `appName` constant is the deployment name used to derive stack names.
3. **`Stacks` interface** — pinned `StackProps` constants, one per region (see Regions below).
4. **`Configuration`** — typed `record`s parsed from a properties/context source (see Configuration below).
5. **Business components** — one package per AWS capability, each with the layers it needs.
6. **Tests** — synthesize each stack and assert on the template; drive `entity` build/spec logic with parameterized tests.

Do not generate speculative resources. Create each business component with the minimal set of resources its responsibility requires, and grow from there.

## Package structure (BCE)

- package path: `[organization].[application].[businessComponent].[boundary|control|entity]` — the same shape `microprofile-server` uses
- the application package `[organization].[application]` (e.g. `airhacks.website`) is the BCE top level: it reflects the application name and holds the shared `Configuration` and `Stacks`. Business components are its **direct children** — never placed directly under the organization
- the `CDKApp` entry-point class sits at the root — in the application package, or at the organization root just above it, as your projects do
- business components are named after the AWS capability they provision (`s3`, `dynamodb`, `lambda`, `apigateway`, `cloudfront`, `route53`, `cognito`); when a component spans **multiple** services, name it after its domain responsibility instead of any single service
- the `appName` constant (e.g. `aws-cicd-website-cdk`) is the deployment name used to derive stack names — a separate concern from the `[application]` package segment
- a component uses only the layers it needs — a stateless resource factory is a `control`-only component (e.g. `s3.control.Buckets`, `route53.control.Route53`); no boundary is required to reach it
- `boundary`, `control`, `entity` segments appear only inside a business component

### Layer responsibilities in a CDK project

- **boundary** — the stack classes (`extends Stack`). A stack is the CloudFormation deployment boundary: it composes constructs, connects cross-stack dependencies, applies tags, and declares outputs. Every stack except the entry-point app lives in a `boundary` package.
- **control** — reusable, stateless resource factories and wiring helpers (bucket factories, Route53 alias-record setup, a CodeBuild publishing project). Prefer `interface` with `static` methods. These are the CDK "constructs" in the capability sense — the reusable behavior.
- **entity** — immutable data that describes infrastructure but creates no resources: build specs, typed value objects, configuration records local to a component (e.g. a `BuildSpec` factory, a `GitRepository` record).

IAM (roles, policy statements) belongs in a dedicated `iam` component. Cognito always gets its own stack.

## Application entry point

- name the entry-point class `CDKApp` and keep it at the root — the application package `[organization].[application]`, or the organization root just above it
- the `appName` constant here (e.g. `aws-cicd-website-cdk`) is the deployment name used to derive CloudFormation stack names — distinct from the `[application]` package segment
- read environment-specific values from CDK context (`app.getNode().tryGetContext("...")`), never hardcode them and never read context or `System.getenv` from inside a stack
- load configuration once here, pass typed records down
- apply organization/environment tags at the app scope with `Tags.of(app).add(key, value)` so every resource inherits them
- instantiate stacks in dependency order and pass **resource interfaces** (not stack instances) from producer to consumer, then call `app.synth()`

## Regions and StackProps

- maintain a central `Stacks` interface exposing pinned `StackProps` constants, one per region — never inline `StackProps` in each stack
- set account/Region at this boundary via `Environment.builder().region(...)`
- enable `crossRegionReferences(true)` only on stacks that consume a resource from a different region (e.g. CloudFront in one region consuming an ACM certificate pinned to `us-east-1`)
- bootstrap each pinned region once: `cdk bootstrap aws://ACCOUNT/REGION`

## Configuration

- model configuration as immutable Java `record`s (a `Configuration` interface with nested records and `static` factory methods is a clean home)
- parse raw properties/context in one place and surface typed records; give records domain behavior (derived names, normalization, `withX` copy methods) rather than leaving them anemic
- pass configuration records into stack constructors — a stack never reaches back to the raw property source

## Stacks (boundary)

- every stack class ends with `Stack`; never ends with `Construct`
- pass the `Construct`/scope as the **first** constructor parameter, followed by typed configuration and the resource interfaces it consumes
- stacks are **independent**: never pass a `Stack` instance to another stack. Pass only the interfaces consumers need (`IBucket`, `ITable`, `IFunction`, `IDistribution`, `ICertificate`). CDK derives the CloudFormation cross-stack reference from the object
- keep stacks thin — they compose constructs and connect dependencies; the resource configuration itself lives in `control` factories
- expose only what other stacks need via package-visible getters; keep internals encapsulated
- derive the CloudFormation stack name from configuration (e.g. `appName-normalizedDomain-component`); set it via the `StackProps` you pass to `super(...)`

## Constructs (control)

- prefer `Builder.create(scope, "LogicalId").…build()` over constructor invocation
- use the **highest-level** construct available (L2/L3); drop to `Cfn*` (L1) only when no L2/L3 exposes the capability, and give the wrapped principal L1 resource the logical id `"Resource"`
- **prove the absence of an L2/L3 before writing any `Cfn*` usage** — memory of what a CDK package contains is reliably stale, because services gain L2s in minor `aws-cdk-lib` releases (bedrockagentcore shipped a full L2 family — `Runtime`, `Gateway`, `Memory` — while only the `Cfn*` layer was widely known). Enumerate the service package in the project's resolved `aws-cdk-lib` jar and look for non-`Cfn` classes: `unzip -l ~/.m2/repository/software/amazon/awscdk/aws-cdk-lib/<version>/aws-cdk-lib-<version>.jar | grep 'services/<service>/'`. Listing the whole package matters: `javap` on the `Cfn*` class you expect will succeed whether or not an L2 exists, so a targeted lookup only confirms the assumption it started from. If the L1 is genuinely alone, say so where the `Cfn*` construct is introduced (commit message or PR), so the next pass re-checks instead of inheriting the choice
- give every resource a **stable, semantic PascalCase logical id** — it becomes the CloudFormation logical id and is deployment identity. Never derive ids from timestamps, randomness, or list positions; do not rename deployed ids for cosmetic reasons
- let CloudFormation generate physical names — omit `bucketName`, `tableName`, `roleName` unless an operator-facing name is genuinely required
- accept and return **interfaces** (`IBucket`, `ITable`) in factory signatures; return the interface type from getters so consumers depend on behavior, not the concrete class
- protect stateful resources: `RemovalPolicy.RETAIN` for buckets/tables that hold data, isolated in their own stack
- expose outputs with `CfnOutput.Builder.create(...)`; attach component-level tags with `Tags.of(scope).add(...)`

## IAM and permissions

- prefer `grant*` methods on the target construct (`bucket.grantReadWrite(role)`, `table.grantReadData(fn)`, `distribution.grantCreateInvalidation(role)`, `logGroup.grantWrite(role)`) — they generate least-privilege, correctly-scoped policies
- fall back to an explicit `PolicyStatement.Builder` / `Role.Builder` only when no `grant*` expresses the permission
- keep roles and policy statements in the dedicated `iam` component
- always give Cognito its own stack

## AWS Step Functions

- use **JSONata** as the query language
- define workflows with `IChainable` and `DefinitionBody.fromChainable(...)`
- give every `StateMachine` a descriptive name reflecting its business purpose

## Synthesis and efficiency

- keep synthesis deterministic and side-effect free: no AWS SDK calls, no randomness, no timestamps during construction — the same source and context must synthesize an equivalent template
- commit `cdk.context.json` when it holds required lookup results, so CI runs reproduce
- minimize runtime AWS API calls in any Lambda/handler code you generate; S3 `PUT`/`DELETE` are idempotent — do not check existence first unless business logic requires it

## Java conventions for CDK

Language-level rules (syntax, naming, visibility, structure) come from `java-conventions`. CDK-specific deltas only:

- a stack class must end in `Stack` — the exception to the no-suffix rule; `*Construct` remains forbidden
- prefer `interface` with `static` methods for stateless `control` factories and helpers
- for synth-time diagnostics a tiny `Log` control over `System.Logger` (as in the reference) is fine; `System.out.println` is acceptable for one-off output

## Testing

- test the synthesized contract, not a snapshot: build an `App`, instantiate the stack(s), call `app.synth()`, and assert the stack artifact's template has the expected `Resources` (and, where it matters, properties, counts, and grants)
- drive `entity` logic (specs, ordering, derived names) with **parameterized** JUnit 5 tests — one labelled row per requirement — since that logic is pure and cheaply verifiable without synthesis
- the blueprint shows both test styles
- when the project carries EARS requirements in a `package-info.java`, compose with `ears-tests` to generate the parameterized rows
- keep tests in the same package as the code under test to use package-private access

## Build and deployment

- Maven, Java 25 source/target; core dependencies `software.amazon.awscdk:aws-cdk-lib` and `software.constructs:constructs`; JUnit 5 + AssertJ for tests
- entry point run via `exec-maven-plugin` with `mainClass` set to the app class; `cdk.json` `"app"` invokes `mvn -e -q compile exec:java`
- deploy multi-stack apps with `cdk deploy --all --context key=value`; a bare `cdk deploy` errors when the app synthesizes more than one stack
- provide `buildAndDeploy.sh` (`mvn -DskipTests clean package && cdk deploy --all`) and `destroy.sh` (`cdk destroy`) helper scripts

## README

The README's map is the **deployables**, not the business components: every stack by the CloudFormation name an operator types, what it creates, and whose credentials apply it. A reader arrives to deploy — the BCE structure is authoring shape, visible in the packages.

- one bullet per stack: the deployed stack name (or its pattern, e.g. `app-workload-[ACCOUNT_ID]-stack` for a per-account stack), what it provisions, and a link to its `package-info` spec when the project carries one
- state the deploy order whenever stacks must be applied in sequence, and name what carries a value from one to the next — a cross-account or cross-region value is configuration, never a resolved reference
- compose with `readme` for the rest of the document; when `sbce` also applies, omit its generated BC map and Components diagram (insert no markers) rather than carrying two maps of one system

## Reference files

- `references/naming-conventions.md` — full CloudFormation, CDK, and Java naming tables (casing for logical IDs, stack names, construct IDs, tags, physical names). Read when unsure how to name any element.
- `references/project-blueprint.md` — a domain-neutral CDK app walked through file by file: `pom.xml`, `cdk.json`, app entry point, `Stacks`, `Configuration` records, a boundary stack, `control` factories, an `entity`, and the two test styles. Read when scaffolding or when you need a concrete template; copy the shapes, not the example domain.
- [AdamBien/aws-cdk-plain](https://github.com/AdamBien/aws-cdk-plain) — a compilable starter template (`CDKApp`, `Configuration`, a placeholder stack) for clone-and-go project starts. It is a seed, not the worked example; when it and this skill or the blueprint disagree, the blueprint governs.
