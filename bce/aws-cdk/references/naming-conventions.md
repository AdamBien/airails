# AWS CDK / CloudFormation naming conventions (Java)

Governing rule: **CloudFormation logical IDs and CDK construct paths are deployment identity — keep them stable.** Do not rename a deployed construct ID or logical ID for cosmetic consistency; it changes the synthesized resource identity and can force a replacement.

## Contents

- [Consolidated casing standard](#consolidated-casing-standard)
- [CloudFormation and resource naming](#cloudformation-and-resource-naming)
- [CDK element naming (Java)](#cdk-element-naming-java)
- [Structural conventions (app / stage / stack / construct)](#structural-conventions-app--stage--stack--construct)

## Consolidated casing standard

| Casing | Use for | Examples |
| --- | --- | --- |
| **PascalCase** | CloudFormation logical IDs; CDK classes, records, interfaces, and construct IDs | `OrdersTable`, `OrdersApiStack`, `VpcId` |
| **lowerCamelCase** | Java fields, variables, methods | `ordersTable`, `environmentName`, `getOrdersTable()` |
| **UPPER_SNAKE_CASE** | Constants and enum members | `DEFAULT_RETENTION_DAYS`, `PAY_PER_REQUEST` |
| **lowercase-kebab-case** | Stack names, necessary physical names, template filenames, tag-key components | `acme-orders-api-prod`, `orders-api.yaml` |
| **namespace:lowercase-kebab-case** | Organization-controlled tag keys | `acme:application-id` |

Acronyms are ordinary words in Pascal/camel case: `VpcId`, `ApiUrl`, `JsonPolicy`, `iamRole` — never `VPCID`, `APIURL`, `JSONPolicy`.

## CloudFormation and resource naming

| Element | Convention | Example | Key rule |
| --- | --- | --- | --- |
| Stack name | `<org>-<workload>-<component>-<environment>`, lowercase kebab | `acme-orders-api-prod` | Start with a letter; letters/digits/hyphens only; ≤128 chars. |
| Resource logical ID | PascalCase describing business purpose | `OrdersTable`, `ApiExecutionRole` | Alphanumeric only; unique per template; stable deployment identity. |
| Output logical ID | PascalCase subject + attribute | `ApiUrl`, `OrdersTableArn` | Avoid generic `Output1`. |
| Export name | Stack name + value | `${AWS::StackName}-ApiUrl` | Unique per account+Region. |
| Physical resource name | Usually omit; when required, lowercase kebab, service-valid | `acme-orders-events-prod` | Pattern `<org>-<workload>-<component>-<env>[-<region>][-<qualifier>]`; service restrictions win. |
| Tag key | Org prefix + lowercase kebab | `acme:application-id`, `acme:cost-center` | Consistent namespace; never the reserved `aws:` prefix. |
| Tag value | Consistent lowercase vocabulary | `orders`, `prod`, `cdk` | Standardize allowed values for env/ownership/management. |

## CDK element naming (Java)

| Element | Convention | Example |
| --- | --- | --- |
| Package | `[organization].[application].[bc].[layer]`; BCs are direct children of the application package | `airhacks.website.cloudfront.boundary` |
| Application package | `[organization].[application]`, reflects the app name; holds `Configuration` / `Stacks` | `airhacks.website` |
| Entry-point class | Always `CDKApp`, at the root (application or organization package) | `CDKApp` |
| Deployment name | `appName` constant used to derive stack names — not the `[application]` segment | `aws-cicd-website-cdk` |
| Stack class | PascalCase ending in `Stack` | `OrdersApiStack` |
| Reusable construct / factory | PascalCase business capability | `OrdersApi`, `Buckets`, `Route53` |
| Configuration record | PascalCase, often nested in a `Configuration` interface | `DomainEntriesConfiguration`, `BuildConfiguration` |
| Field | lowerCamelCase | `ordersTable` |
| Method | lowerCamelCase | `getOrdersTable()` |
| Constant / enum member | UPPER_SNAKE_CASE | `DEFAULT_RETENTION_DAYS`, `PAY_PER_REQUEST` |
| Construct ID (2nd `Builder.create` arg) | Stable PascalCase string | `"OrdersTable"`, `"CloudFrontDistribution"` |
| Primary L1 construct ID inside a wrapper | `"Resource"` | `"Resource"` |
| Behavioral interface (CDK library) | `I` + PascalCase | `IBucket`, `ITable`, `IDistribution` |
| Source filename | Same as the public class | `OrdersApiStack.java` |

Prefer capability names over service-only names for constructs where a capability fits (`EncryptedDataStore` over `S3`), but naming a plain resource factory after its service (`Buckets`, `Route53`) is idiomatic in infrastructure-only projects.

## Structural conventions (app / stage / stack / construct)

| Area | Convention | Avoid |
| --- | --- | --- |
| Application entry point | Environment discovery + top-level composition in one class | Reading env vars throughout stacks/constructs |
| Stage (optional) | A `Stage` groups the stacks of one environment | Treating a stage as a reusable component |
| Stack boundaries | Define by deployment lifecycle, ownership, failure domain | One stack per AWS service (`S3Stack`, `IamStack`) |
| Stack responsibility | Compose constructs, connect dependencies, apply settings | Putting all resource configuration in the stack |
| Construct responsibility | Model one cohesive capability | Grouping unrelated resources |
| Construct granularity | Group resources that operate as one unit | One construct per individual resource |
| Abstraction level | Prefer L2/L3 | `Cfn*` L1 without a specific need |
| Configuration | Immutable typed props/records | Hidden mutable state, hardcoded env values |
| Resource references | Pass CDK interfaces (`ITable`, `IBucket`) | Passing names/ARNs/IDs as strings |
| Public API | Expose only what consumers need | Exposing every internal child resource |
| Physical names | Let CloudFormation generate them | Naming every resource explicitly |
| Stateful resources | Isolate + `RemovalPolicy.RETAIN` | Databases in volatile stacks unprotected |
| Cross-stack references | Pass resource objects explicitly | Copying ARNs/physical names by hand |
| Environment handling | Account/Region at app or stage boundary | Looking up account/Region inside constructs |
| Synthesis | Deterministic, side-effect free | SDK calls, timestamps, randomness during synth |
| Testing | Assert on the synthesized template | Snapshot-only tests |

### Recommended hierarchy

```
App                              selects accounts, Regions, environments
└── (optional) Stage             groups the stacks of one environment
    ├── DataStack (boundary)     stateful data lifecycle boundary
    │   └── data control/entity  reusable data capability + domain data
    ├── ApiStack (boundary)      application deployment boundary
    │   └── api control          API Gateway, functions, grants, alarms
    └── MonitoringStack          independently owned observability
```

### Core design rule

**Constructs (control) build reusable capabilities; stacks (boundary) define deployable boundaries.** Most resource configuration lives in cohesive `control` factories; stacks stay thin and focus on composition, environment configuration, dependencies, and lifecycle.
