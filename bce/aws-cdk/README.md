# aws-cdk

An [AIrails.dev](https://airails.dev) skill for creating and reviewing Java AWS CDK v2 infrastructure code organized with the Boundary-Control-Entity pattern.

## Scope

- CDK v2 project structure composing with the [`/bce`](https://bce.design) skill — business components named after the AWS capability they provision (`s3`, `cloudfront`, `certificate`, `route53`, `codebuild`, `cognito`, `iam`, `stepfunctions`), with `boundary`, `control`, `entity` only inside a BC
- The app→stack→construct split: stacks (`boundary`) are thin CloudFormation deployment boundaries; reusable resource factories (`control`) hold the configuration; immutable specs and value objects live in `entity`
- Region pinning through a central `Stacks` interface, typed `Configuration` records, and context-driven environment selection
- High-level L2/L3 constructs, `Builder.create` syntax, stable PascalCase logical IDs, and `grant*` least-privilege permissions
- Synthesis-based tests plus parameterized tests for pure `entity` logic

Not for TypeScript/Python CDK, Terraform, Pulumi, or non-AWS infrastructure.

## Composition

This skill composes with [`bce`](../bce), which provides the technology-neutral architecture pattern, and [`java-conventions`](../../java/java-conventions), which provides all language-level Java 25 rules. `aws-cdk` adds only the AWS CDK authoring context on top — project layout, the app/stack/construct split, region pinning, typed configuration, permissions, and synthesis tests. It also composes with [`ears-tests`](../ears-tests) when a business component carries EARS requirements in its `package-info.java`.

## Reference material

- [`references/naming-conventions.md`](references/naming-conventions.md) — the full CloudFormation, CDK, and Java naming tables (casing for logical IDs, stack names, construct IDs, tags, physical names)
- [`references/project-blueprint.md`](references/project-blueprint.md) — a domain-neutral CDK app walked through file by file, as a concrete scaffolding template (copy the shapes, not the example domain)

## Usage

Activated automatically when creating, scaffolding, extending, or reviewing AWS CDK code in Java. The rules in [SKILL.md](SKILL.md) guide code generation and review decisions.

## Test

```
create a CDK app that provisions a DynamoDB table and a Lambda REST API for an orders service
```
