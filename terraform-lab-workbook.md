# Terraform for AWS Ops Engineering

## 4-Week Portfolio Lab

**Starting point:** ~6 years software engineering, strong C#/.NET, working knowledge of Docker/Kubernetes, essentially zero Terraform.

**Schedule:** 4 weeks × 5 days × 1 hour/day = **20 hours**

**Primary goals:**

1. Become comfortable writing and explaining Terraform from scratch.
2. Understand Terraform as an engineering workflow, not merely HCL syntax.
3. Gain practical AWS infrastructure experience.
4. Build a public GitHub portfolio demonstrating Infrastructure as Code.
5. Become prepared for Terraform/AWS/DevOps review questions.
6. Finish with a realistic serverless/event-driven AWS project

### The final portfolio project

You will build a fictional **Operations Event Processing Platform**:

```
                     ┌──────────────────┐
                     │   HTTP Client     │
                     │ curl / test app   │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │   API Gateway    │
                     │     HTTP API     │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │ Ingest Lambda    │
                     │    Python        │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │      SQS         │
                     │   Event Queue    │
                     └───────┬──────────┘
                             │
                    failures │
                             ▼
                     ┌──────────────────┐
                     │       DLQ        │
                     └──────────────────┘

                             │
                             ▼
                     ┌──────────────────┐
                     │ Process Lambda   │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │    DynamoDB      │
                     │   Event State    │
                     └──────────────────┘

                     CloudWatch Logs
                     + basic alarms
                     + Terraform tests
```

AWS describes HTTP APIs as a lightweight API product that can integrate directly with Lambda, while SQS supports dead-letter queues for messages that repeatedly fail processing. DynamoDB provides a fully managed serverless datastore appropriate for operational workloads.

* * *

# 1. Rules for the course

## Rule 1: Everything goes into GitHub

Create a public repository from Day 1:

```
terraform-ops-suite-lab
```

Don't wait until the project is finished and dump the code into GitHub.

Your commit history should demonstrate progression:

```
feat: add first terraform configuration
feat: provision secure S3 bucket
feat: add reusable networking module
feat: introduce remote state backend
ci: add terraform validation workflow
ci: add AWS OIDC authentication
test: add terraform module tests
security: add checkov and tflint
feat: add event processing pipeline
docs: add architecture decision records
fix: correct SQS visibility timeout
```

* * *

# 2. Cost strategy

The target is **$0–$20**, with the strong possibility of spending $0.

AWS currently offers new customers up to $200 in credits over six months on its Free Plan. HCP Terraform currently has a Free plan supporting up to 500 managed resources, and public repositories using standard GitHub-hosted Actions runners do not incur GitHub Actions charges.

Still, treat AWS as capable of charging money.

### Deliberately avoid

```
NAT Gateway
EC2 instances
RDS
EKS
Load Balancers
OpenSearch
CloudFront
large data transfers
```

The first five are unnecessary for demonstrating Terraform competence and can turn a $0 lab into a bill.

You will use primarily:

```
S3
IAM
VPC
Lambda
API Gateway HTTP API
SQS
DynamoDB
CloudWatch
```

You will also create VPC networking resources without deploying expensive compute into them.

### Cost discipline

Every resource gets tags such as:

```
tags = {
  Project     = "terraform-ops-suite"
  Environment = var.environment
  ManagedBy   = "terraform"
  Owner       = "learning"
}
```

Create an AWS budget/billing alert before deploying the larger labs.

At the end of every lab:

```
terraform plan
terraform destroy
```

unless the resource is intentionally reused by the next lab.

* * *

# 3. Version and tooling

As of October 2026, HashiCorp's release page shows **Terraform 1.16.5 as the latest stable 1.16 release**, while 1.17 releases are still in pre-release status. Use the current stable release rather than a beta.

Install:

```
Terraform 1.16.5
AWS CLI
Git
VS Code
GitHub account
AWS account
```

Optional:

```
tflint
Checkov
jq
Make
```

HashiCorp recommends formatting and validating Terraform before committing, and specifically recommends using TFLint for organization-specific coding rules.

* * *

# 4. GitHub repository structure

Build toward this:

```
terraform-ops-suite-lab/
│
├── README.md
├── LICENSE
├── .gitignore
├── Makefile
│
├── docs/
│   ├── architecture.md
│   ├── troubleshooting.md
│   ├── review-notes.md
│   └── adr/
│       ├── 001-terraform.md
│       ├── 002-remote-state.md
│       ├── 003-serverless-architecture.md
│       └── 004-oidc.md
│
├── labs/
│   ├── 01-terraform-basics/
│   ├── 02-aws-s3/
│   ├── 03-expressions-and-data/
│   ├── 04-networking/
│   ├── 05-modules/
│   ├── 06-state/
│   ├── 07-github-actions/
│   └── 08-testing-security/
│
├── modules/
│   ├── s3-bucket/
│   ├── network/
│   ├── queue/
│   ├── lambda/
│   └── ops-pipeline/
│
├── environments/
│   ├── dev/
│   └── prod/
│
├── app/
│   ├── ingest/
│   │   └── lambda_function.py
│   └── processor/
│       └── lambda_function.py
│
└── .github/
    └── workflows/
        ├── terraform-pr.yml
        ├── terraform-plan.yml
        └── terraform-apply.yml
```

Do **not** put Terraform state into Git.

Your `.gitignore` should ignore:

```
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
```

But **do commit** `**.terraform.lock.hcl**`.

That last point is a useful review detail.

* * *

# WEEK 1 — Terraform Fundamentals

## Objective

By Friday, you should understand:

```
HCL
providers
resources
variables
outputs
locals
data sources
expressions
for_each
count
dependency graph
plan/apply/destroy
state
```

You should no longer feel like Terraform is "magic."

* * *

## DAY 1 — Infrastructure as Code + Terraform Workflow

### Reading — 15 minutes

Read:

**HashiCorp: Get Started – AWS**

The official tutorial series walks through installation, configuration, authentication, `init`, `plan`, `apply`, variables, outputs, modules, destroy, and HCP Terraform.

Focus especially on:

```
terraform init
terraform plan
terraform apply
terraform destroy
```

### Lab — 40 minutes

Create the GitHub repo.

Install Terraform.

Run:

```
terraform version
```

Create:

```
labs/01-terraform-basics/
```

Write a configuration using the `random` provider.

Create something simple such as a randomly generated project name.

Run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
terraform show
terraform destroy
```

Then run:

```
terraform console
```

Experiment with:

```
upper("company")
lower("TERRAFORM")
length(["aws", "lambda", "sqs"])
```

### Commit

```
feat: add terraform fundamentals lab
```

### Knowledge check

**Explain:** What does `terraform init` actually initialize?

**Review:** Why doesn't Terraform simply inspect AWS every time instead of maintaining state?

* * *

# DAY 2 — HCL, Variables, Outputs and Types

### Reading

Read HashiCorp's variable documentation and configuration language material. Terraform variables can have types, descriptions, validation rules, and sensitivity settings.

### Lab

Build a configurable "application configuration" module.

Use:

```
string
number
bool
list
set
map
object
```

Create:

```
variables.tf
locals.tf
outputs.tf
main.tf
```

Add input validation:

```
variable "environment" {
  type        = string
  description = "Deployment environment."

  validation {
    condition     = contains(["dev", "prod"], var.environment)
    error_message = "Environment must be dev or prod."
  }
}
```

Experiment with:

```
terraform plan -var="environment=dev"
terraform plan -var="environment=test"
```

Observe the validation failure.

### Knowledge check

**Explain:** Why are Terraform types important?

**Review:** What's the difference between a variable, a local value, and an output?

* * *

# DAY 3 — Your First Real AWS Infrastructure

## Lab 2 — Secure S3 Bucket

### Reading

Run through the relevant portion of HashiCorp's AWS getting-started tutorial.

### Lab

Create an S3 bucket with Terraform.

Include:

```
bucket
versioning
server-side encryption
public access block
tags
output
```

Your goal isn't simply:

```
resource "aws_s3_bucket" "example" {}
```

Your goal is to demonstrate that you understand secure infrastructure.

Run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Then inspect the resources in the AWS console.

Change a configuration value.

Run:

```
terraform plan
```

Before applying, explain every proposed change.

### Knowledge check

**Explain:** What information does Terraform get from the provider during `plan`?

**Review:** What's the difference between `plan` and `apply`?

* * *

# DAY 4 — Expressions, Data Sources, count and for_each

### Reading

Read:

* Terraform `for` expressions
* `for_each`
* Data sources
* Meta-arguments

Terraform data sources read information without creating or modifying the referenced object. `for_each` creates multiple resource instances based on a map or set.

### Lab 3

Create several S3 buckets based on:

```
variable "buckets" {
  type = map(string)

  default = {
    logs    = "application logs"
    uploads = "application uploads"
    archive = "application archive"
  }
}
```

Use:

```
for_each = var.buckets
```

Then use a data source such as:

```
data "aws_caller_identity" "current" {}
```

Build outputs from the resulting resources.

Experiment with both:

```
count
for_each
```

Then intentionally change the order of a `count` list and observe the consequences.

### Knowledge check

**Review:** When would you choose `for_each` instead of `count`?

**Review:** What is a data source, and when would you use one instead of a resource?

* * *

# DAY 5 — Terraform State and Drift

### Reading

Read Terraform's State documentation.

Terraform state maps Terraform resource instances to real infrastructure and stores information Terraform uses to determine what changes are required. Remote state is recommended for teams.

### Lab

Take your S3 infrastructure.

Run:

```
terraform state list
terraform state show aws_s3_bucket.example
terraform show
```

Now manually modify the bucket in AWS.

For example, alter a tag.

Then:

```
terraform plan
```

Observe Terraform detecting the difference.

Create a deliberate state-management exercise:

```
terraform state list
terraform state show <resource>
```

Then use:

```
terraform graph
```

and inspect the dependency graph.

### Week 1 checkpoint

You should now be able to explain:

```
HCL
provider
resource
data source
variable
local
output
state
drift
plan
apply
destroy
count
for_each
dependency graph
```

If you cannot explain those without looking at documentation, repeat Days 4–5 before continuing.

* * *

# WEEK 2 — AWS Infrastructure + Reusable Terraform

## Objective

This week shifts from "I can write Terraform" to:

> "I can design infrastructure with Terraform."

* * *

# DAY 6 — AWS Networking Fundamentals

### Reading

Read AWS VPC basics.

A VPC is a logically isolated network. Subnets live inside a VPC and route tables determine where traffic is directed.

### Lab 4 — Build a Network Module

Create:

```
modules/network/
```

Terraform should create:

```
VPC
2 subnets
Internet Gateway
route table
route associations
security group
```

Do NOT create a NAT Gateway.

Diagram it in your README.

Your goal is not to become a network engineer. Your goal is to be able to answer:

```
What is a subnet?
What makes a subnet public?
What is a route table?
What is an Internet Gateway?
What does a security group do?
Why might an application need a private subnet?
What does NAT do?
```

This is particularly worthwhile because you've identified networking as one of your weaker areas.

### Knowledge check

Draw a VPC from memory.

Then explain the path of traffic:

```
EC2 → subnet → route table → internet gateway → internet
```

* * *

# DAY 7 — IAM and SQS

### Reading

Read about:

```
AWS IAM roles
IAM policies
SQS
dead-letter queues
```

AWS describes a DLQ as a separate SQS queue that receives messages that couldn't be successfully processed.

### Lab

Create:

```
SQS queue
SQS dead-letter queue
redrive policy
IAM policy
IAM role
```

Do this entirely with Terraform.

Start thinking in terms of:

> "What is the minimum permission this application needs?"

Instead of:

```
Action: "*"
Resource: "*"
```

Aim toward resource-specific permissions.

### Knowledge check

**Review:** What's the difference between an IAM user and an IAM role?

**Review:** Why is least privilege important for Terraform and CI/CD?

* * *

# DAY 8 — Terraform Modules

### Reading

Read HashiCorp's module documentation and standard module structure.

HashiCorp recommends reusable modules expose inputs and outputs and warns against creating unnecessary abstractions.

### Lab 5 — Refactor

Take your previous configuration and create:

```
modules/
    s3-bucket/
    network/
    queue/
```

Your root configuration becomes a composition layer:

```
module "network" {
  source = "../../modules/network"

  environment = var.environment
}
```

Do the same for S3 and SQS.

Create README files for each module.

### Recruiter signal

A recruiter or hiring manager seeing:

```
modules/
examples/
README.md
variables.tf
outputs.tf
```

is very different from seeing one giant `main.tf`.

### Knowledge check

**Review:** What makes a good Terraform module?

**Review:** When should you NOT create a module?

* * *

# DAY 9 — Environments and Configuration

### Lab

Create:

```
environments/
    dev/
    prod/
```

Use:

```
dev.tfvars
prod.tfvars
```

Experiment with:

```
terraform plan -var-file=dev.tfvars
terraform plan -var-file=prod.tfvars
```

Learn the difference between:

```
configuration
variables
tfvars
modules
state
workspaces
```

Do a short experiment with Terraform workspaces so you understand them, but don't make them the central environment-management strategy for your final project.

### Knowledge check

**Review:** When would you use separate state per environment?

**Review:** What problem are Terraform workspaces solving?

* * *

# DAY 10 — Import, Refactoring and Lifecycle

### Reading

Terraform's current workflow supports declarative `import` blocks, which allow existing infrastructure to be brought under Terraform management.

### Lab

Create an AWS resource manually.

Then import it into Terraform.

Practice:

```
import
state show
moved
lifecycle
prevent_destroy
ignore_changes
precondition
postcondition
```

Create a deliberate failure with:

```
prevent_destroy = true
```

Then explain what happened.

### Knowledge check

**Review:** How would you bring an existing manually created AWS resource under Terraform management?

**Review:** What is Terraform state supposed to represent?

* * *

# WEEK 3 — Production Terraform: State, CI/CD, Security and Testing

This is the week that turns the GitHub project from a tutorial into something that looks like engineering work.

* * *

# DAY 11 — Remote State

### Reading

Read:

* Terraform State
* S3 backend documentation
* HCP Terraform overview

Terraform's current S3 backend supports state locking through `use_lockfile`; DynamoDB-based locking is now deprecated. HashiCorp also recommends S3 bucket versioning for state recovery.

### Lab 6

First create a dedicated state bucket.

Then migrate your local state into:

```
S3
```

Configure:

```
terraform {
  backend "s3" {
    bucket       = "YOUR-STATE-BUCKET"
    key          = "ops-suite/dev/terraform.tfstate"
    region       = "us-east-1"
    use_lockfile = true
  }
}
```

Run:

```
terraform init
```

Terraform should ask whether you want to migrate the existing state.

Practice explaining:

```
Why remote state?
Why locking?
Why versioning?
Why shouldn't state be committed to Git?
Why shouldn't multiple environments share one state file?
```

### Optional extension

Create a free HCP Terraform workspace and compare:

```
local state
S3 remote state
HCP Terraform
```

The HCP Terraform Free plan currently allows up to 500 managed resources.

* * *

# DAY 12 — GitHub Actions Terraform CI

### Reading

Read about:

```
Terraform CLI automation
GitHub Actions
HashiCorp setup-terraform
```

The official `setup-terraform` action installs a chosen Terraform CLI version and lets subsequent GitHub Actions steps run Terraform commands normally.

### Lab 7

Create:

```
.github/workflows/terraform-pr.yml
```

Your pull-request pipeline should run:

```
checkout
terraform setup
terraform fmt -check
terraform init -backend=false
terraform validate
tflint
checkov
terraform test
```

A simplified beginning:

```
name: Terraform CI

on:
  pull_request:

permissions:
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: "1.16.5"

      - name: Format
        run: terraform fmt -check -recursive

      - name: Initialize
        run: terraform init -backend=false

      - name: Validate
        run: terraform validate
```

The important thing isn't memorizing the YAML.

It's understanding why each stage exists.

### Knowledge check

**Review:** Why run `terraform fmt` and `validate` in CI if developers already run them locally?

**Review:** Why might you use `terraform init -backend=false` in a configuration-validation job?

* * *

# DAY 13 — GitHub OIDC → AWS

This is one of the highest-value labs in the entire course.

### Reading

Read GitHub's AWS OIDC documentation.

GitHub recommends OIDC so Actions can obtain short-lived AWS credentials rather than storing long-lived AWS access keys as GitHub secrets. The AWS trust policy should restrict which workflows/repositories can obtain those credentials.

### Lab 8

Create:

```
GitHub Actions
       ↓
GitHub OIDC
       ↓
AWS IAM Role
       ↓
Terraform
       ↓
AWS
```

Configure the IAM trust relationship so only your repository's intended branch/workflow can assume the role.

Do NOT create:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

GitHub secrets for this.

Use:

```
permissions:
  id-token: write
  contents: read
```

and the AWS credentials action.

### Portfolio signal

Put an architecture diagram for this in:

```
docs/adr/004-oidc.md
```

Explain why OIDC is safer than long-lived credentials.

### Knowledge check

**Review:** What problem does OIDC solve?

**Review:** How would you prevent a different GitHub repository from assuming your AWS role?

* * *

# DAY 14 — Terraform Testing

### Reading

Terraform now has dedicated testing functionality through `terraform test`. Tests can validate module behavior independently of your existing state, and Terraform supports provider/resource mocking so tests can avoid provisioning infrastructure.

### Lab 9

Create:

```
modules/s3-bucket/tests/
modules/queue/tests/
```

Write `.tftest.hcl` tests.

Test:

```
input validation
resource naming
tags
configuration behavior
conditional resources
outputs
```

Then experiment with mocked providers.

Your goal is to be able to say:

> "I used Terraform's native test framework and provider mocking to test module behavior without needing to create AWS resources for every test."

That's a significantly stronger review answer than:

> "I ran terraform plan."

* * *

# DAY 15 — Infrastructure Security and Quality Gates

### Lab 10

Add:

```
TFLint
Checkov
terraform fmt
terraform validate
terraform test
```

to your GitHub Actions pipeline.

Checkov is designed to scan infrastructure-as-code for misconfigurations and security issues, and TFLint provides Terraform-specific linting.

Intentionally introduce:

```
unencrypted S3 bucket
overly broad IAM policy
missing tags
bad naming
invalid Terraform
```

Watch the CI pipeline fail.

Fix each problem.

### Week 3 checkpoint

At this point your public repository should demonstrate:

```
Terraform
AWS
Modules
Remote state
State locking
GitHub Actions
OIDC
Testing
TFLint
Checkov
Security practices
```

This is the point where the project starts becoming genuinely resume-worthy.

* * *

# WEEK 4 — CAPSTONE: AWS Operations Event Platform

> "Design a highly available event-processing system for operational events."

* * *

# DAY 16 — Architecture and Requirements

Before writing Terraform, create:

```
docs/architecture.md
docs/adr/003-serverless-architecture.md
```

Define an event:

```
{
  "event_type": "FLIGHT_DELAY",
  "flight_id": "SW1234",
  "airport": "DAL",
  "timestamp": "2026-10-04T18:30:00Z",
  "severity": "HIGH",
  "message": "Weather delay"
}
```

Requirements:

```
HTTP ingestion endpoint
asynchronous processing
retry behavior
dead-letter queue
persistent event state
least-privilege IAM
logging
basic monitoring
Infrastructure as Code
automated tests
CI/CD
```

Create the architecture diagram before implementation.

### Knowledge check

**Review:** Why put SQS between the API and processing Lambda?

Your answer should involve:

```
decoupling
asynchronous processing
retries
failure isolation
back-pressure
durability
```

* * *

# DAY 17 — Build the Serverless Infrastructure

Create modules:

```
modules/
    api/
    lambda/
    queue/
    dynamodb/
```

Provision:

```
API Gateway HTTP API
ingestion Lambda
SQS
SQS DLQ
processor Lambda
DynamoDB
IAM roles/policies
CloudWatch logs
```

API Gateway HTTP APIs are particularly appropriate here because they can directly integrate with Lambda and are designed as a lower-cost/minimal-feature API product.

Write the Lambda functions in **Python**.

### Acceptance test

Run:

```
curl ...
```

and verify:

```
API Gateway
    ↓
Lambda
    ↓
SQS
    ↓
Processor
    ↓
DynamoDB
```

* * *

# DAY 18 — Reliability Engineering

This should be one of the most impressive days.

Implement:

```
SQS visibility timeout
retry behavior
DLQ
CloudWatch log groups
log retention
basic CloudWatch alarms
Lambda error logging
```

Then deliberately break the processor.

For example:

```
processor rejects a particular event type
```

Verify the event eventually ends up in the DLQ.

Document what happened in:

```
docs/troubleshooting.md
```

Include:

```
Symptom
Investigation
Root cause
Fix
Preventive measure
```

### Reflection question

> An application suddenly starts failing 20% of its asynchronous events. How would you troubleshoot it?

Your answer should involve:

```
CloudWatch
Lambda logs
SQS metrics
DLQ
recent deployments
Terraform changes
configuration
permissions
dependency failures
```

* * *

# DAY 19 — Production CI/CD

Now combine everything.

Your GitHub pipeline should have approximately this lifecycle:

```
Developer
   │
   ▼
Pull Request
   │
   ├── fmt
   ├── validate
   ├── tflint
   ├── checkov
   └── terraform test
        │
        ▼
      Merge
        │
        ▼
     Terraform
       Plan
        │
        ▼
     Approval
        │
        ▼
       Apply
        │
        ▼
       AWS
```

Create:

```
terraform-pr.yml
terraform-plan.yml
terraform-apply.yml
```

The PR workflow should **not** have broad AWS write permissions.

Your deployment workflow should use:

```
GitHub OIDC
+
restricted AWS IAM role
```

not stored AWS access keys.

Upload the Terraform plan as a GitHub Actions artifact.

For extra credit, publish the plan result in the pull request.

* * *

# DAY 20 — Final Review + Simulation

This is not a "finish whatever code is left" day.

## Part 1 — Destroy and rebuild

From an empty AWS environment:

```
terraform init
terraform plan
terraform apply
```

Verify the entire system works.

Then:

```
terraform destroy
```

Rebuild it.

Your objective is to prove that your infrastructure is truly reproducible.

* * *

# Part 2 — Break it

Intentionally create three problems.

Examples:

```
wrong IAM permission
invalid variable
incorrect SQS setting
bad Terraform reference
removed resource
manual AWS change
```

Diagnose and fix them.

Commit at least one of the fixes.

* * *

# Part 3 — Explain your architecture

Record yourself answering these questions without notes.

### Terraform fundamentals

1. What is Terraform?
2. What is Infrastructure as Code?
3. What happens during `terraform init`?
4. What happens during `terraform plan`?
5. What happens during `terraform apply`?
6. What is Terraform state?
7. Why do teams need remote state?
8. Why is state locking necessary?
9. What is drift?
10. What is a provider?

### Terraform design

11. Difference between a resource and a data source?
12. `count` vs `for_each`?
13. What makes a good module?
14. When can modules make an architecture worse?
15. What are variables?
16. What are locals?
17. What are outputs?
18. What are preconditions/postconditions?
19. What is an `import` block?
20. What is a `moved` block?

### DevOps

21. Why run Terraform through CI/CD?
22. What should run on every pull request?
23. Why shouldn't AWS credentials be stored in GitHub?
24. What is OIDC?
25. How does Terraform authentication differ locally vs CI?
26. How would you prevent two deployment jobs from modifying the same state simultaneously?
27. How would you handle a failed deployment?
28. How would you detect infrastructure drift?
29. How would you secure Terraform state?
30. What would you do if Terraform wants to destroy a production resource unexpectedly?

### AWS

31. What is a VPC?
32. Public vs private subnet?
33. What does a route table do?
34. What does an Internet Gateway do?
35. What does NAT do?
36. What does an IAM role do?
37. Why use SQS?
38. Why use a DLQ?
39. Why use Lambda?
40. Why use DynamoDB here?

### Architecture

41. Why is the application asynchronous?
42. What happens if the processor goes down?
43. What happens if a message cannot be processed?
44. How would you make the system more observable?
45. How would you scale it?
46. What would you change for a production version?
47. How would you handle secrets?
48. How would you test infrastructure?
49. How would you safely make a Terraform change?
50. How would you roll back an infrastructure change?

You should be able to answer the important ones in **60–90 seconds without rambling**.

* * *

# 5. Your final README

The README is extremely important.

A recruiter should be able to understand the project in about two minutes.

Use this structure:

```
# Terraform AWS Ops Suite

## Overview

A production-style Infrastructure-as-Code project demonstrating
Terraform, AWS serverless architecture, CI/CD, testing, security,
remote state and operational troubleshooting.

## Architecture

[diagram]

## Technologies

- Terraform
- AWS
- Lambda
- API Gateway
- SQS
- DynamoDB
- IAM
- CloudWatch
- GitHub Actions
- Python
- Bash
- YAML

## Terraform Practices Demonstrated

- Modules
- Variables and outputs
- Data sources
- for_each
- Remote state
- State locking
- Import
- Lifecycle controls
- Preconditions/postconditions
- Terraform testing
- Provider locking

## CI/CD

[diagram]

## Security

- GitHub OIDC
- Least-privilege IAM
- No long-lived AWS credentials
- Checkov
- TFLint

## Reliability

- SQS retries
- Dead-letter queue
- CloudWatch monitoring
- Failure testing
- Troubleshooting documentation

## Testing

[GitHub Actions badge]

## Lessons Learned

...

## Future Improvements

...
```

HashiCorp's recommended module structure also calls for README documentation, variables, outputs, and a standard directory structure, which makes this organization defensible beyond simply "looking nice."

* * *

# 6. Things I specifically want you to demonstrate

There are several things I'd deliberately emphasize because they separate a beginner portfolio from an experienced-engineer portfolio.

### 1. Don't just show successful deployments

Show **failure**.

Your repository should have evidence of:

```
bad configuration
failed CI
Terraform error
AWS permission problem
debugging
fix
```

### 2. Explain decisions

Create lightweight ADRs:

```
Why Terraform?
Why SQS?
Why Lambda?
Why remote S3 state?
Why OIDC?
Why separate environments?
Why no NAT Gateway?
Why Python for Lambda?
```

### 3. Don't over-engineer it

A common Terraform beginner mistake is creating:

```
17 modules
83 variables
12 layers of abstraction
```

HashiCorp explicitly recommends moderation with modules and warns that unnecessary abstraction can make configurations harder to understand.

A small number of well-designed modules is better.

### 4. Treat Terraform like software

You already have a software-engineering background.

Apply it:

```
version control
code review
testing
linting
CI/CD
documentation
dependency management
reproducibility
failure testing
security
observability
```

That's the angle that will make this portfolio valuable.

* * *

# 7. What this should allow you to say in an explanation

At the beginning of the course, your answer would be:

> "I've been learning Terraform."

At the end, the answer becomes:

> "I built an AWS serverless operations-event platform entirely with Terraform. I organized the infrastructure into reusable modules, moved state to a remote S3 backend with locking and versioning, built GitHub Actions validation and deployment pipelines, used GitHub OIDC rather than long-lived AWS credentials, added Terraform tests and static analysis, and deliberately tested failure scenarios involving SQS retries and dead-letter processing."

That is a dramatically stronger story.

And importantly, it is **truthful**. You aren't claiming six months of professional Terraform experience. You're demonstrating that you can learn a new infrastructure technology and apply software-engineering discipline to it.

* * *

# 8. Final recruiter-readiness scorecard

Before applying, aim for:

| Area | Target |
| --- | --- |
| Terraform fundamentals | Can explain without notes |
| HCL | Comfortable writing from scratch |
| AWS | Can explain architecture |
| Networking | Can draw VPC/subnet/routing model |
| State | Can explain remote state + locking |
| Modules | Can design and justify modules |
| CI/CD | Working GitHub Actions pipeline |
| AWS authentication | OIDC working |
| Security | Checkov + least privilege |
| Testing | `terraform test` demonstrated |
| Operations | Failure scenario documented |
| GitHub | Clean public repo |
| Documentation | Architecture + ADRs |

* * *

# 9. The three most valuable portfolio artifacts

By the end of the four weeks, make sure the GitHub repository prominently displays these three things:

**Artifact #1 — Architecture**

A clean diagram showing:

```
API Gateway → Lambda → SQS → Lambda → DynamoDB
                       ↓
                      DLQ
```

**Artifact #2 — CI/CD**

A GitHub Actions workflow demonstrating:

```
fmt
validate
test
tflint
checkov
plan
OIDC
apply
```

**Artifact #3 — Production incident**

A short troubleshooting document showing:

```
Failure
↓
Investigation
↓
Root Cause
↓
Terraform/AWS Fix
↓
Test
↓
Prevention
```

* * *

# Official reading library

Keep these bookmarked throughout the course.

**Terraform AWS Getting Started** — installation, first AWS infrastructure, variables, modules, destroy, HCP Terraform. [HashiCorp Terraform — Get Started with AWS](https://developer.hashicorp.com/terraform/tutorials/aws-get-started?utm_source=chatgpt.com)

**Terraform State** — state purpose, mapping, remote state. [HashiCorp Terraform — State](https://developer.hashicorp.com/terraform/language/state?utm_source=chatgpt.com)

**Terraform Modules** — reusable module design and structure. [HashiCorp Terraform — Modules](https://developer.hashicorp.com/terraform/language/modules/develop?utm_source=chatgpt.com)

**Terraform Testing** — native tests and validation. [HashiCorp Terraform — Testing](https://developer.hashicorp.com/terraform/cli/test?utm_source=chatgpt.com)

**Terraform S3 Backend** — remote state and current S3 locking approach. [HashiCorp Terraform — S3 Backend](https://developer.hashicorp.com/terraform/language/backend/s3?utm_source=chatgpt.com)

**GitHub OIDC with AWS** — short-lived AWS credentials from Actions. [GitHub — Configuring OpenID Connect in AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws?utm_source=chatgpt.com)

**AWS VPC** — networking fundamentals. [AWS — What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html?utm_source=chatgpt.com)

**API Gateway HTTP APIs** — serverless API/Lambda integration. [AWS — HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api.html?utm_source=chatgpt.com)

**TFLint** — Terraform linting. [TFLint on GitHub](https://github.com/terraform-linters/tflint?utm_source=chatgpt.com)

**Checkov** — IaC security scanning. [Checkov GitHub Action](https://github.com/bridgecrewio/checkov-action?utm_source=chatgpt.com)

* * *
