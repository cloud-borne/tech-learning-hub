---
title: AWS Multi-Account Strategy
linktitle: 🏢 Multi-Account
type: book
tags:
  - AWS
  - Organizations
  - Security
  - CI/CD
date: "2026-10-03T00:00:00Z"
weight: 160
toc: true
draft: false
---

One AWS account is a useful starting point. A well-designed **account portfolio** is how a growing team gives workloads room to move without giving risk room to spread.

<!--more-->

## 🧭 Multi-Account Strategies

An AWS account is more than a billing container: it is a strong boundary for identity, quotas, operational ownership, and many security controls. Separate accounts when workloads need different owners, environments, data classifications, or blast-radius limits.

{{< accordion "A practical account layout" >}}

- **Management**: owns the AWS Organization and billing; keep workloads out.
- **Security**: central security tooling, investigation, and delegated security services.
- **Log archive**: tightly controlled destination for organization-wide audit logs.
- **Shared services**: approved common services such as directory, artifact, or DNS services.
- **Workloads**: separate production, non-production, and sandbox environments where the risk and ownership justify it.

Start with a small number of purposeful accounts and automate account creation. Avoid both extremes: one account for everything, and an account for every tiny component before you can operate them.

{{< /accordion >}}

Good boundaries make incidents easier to contain, costs easier to attribute, and policy easier to reason about. They also add moving parts: cross-account roles, network paths, quotas, and deployment workflows all need deliberate design.

## 🌳 AWS Organizations

AWS Organizations groups accounts into an **organization**. The management account creates the organization, invites or creates accounts, arranges them into **organizational units (OUs)**, and applies organization-level policies.

| Building block | What it does |
| --- | --- |
| Root | Top of the organization hierarchy; every account belongs under it. |
| OU | Groups accounts with similar purpose or governance needs. OUs can nest. |
| Member account | A workload or shared-purpose account governed by the organization. |
| Management account | Owns organization-wide administration and consolidated billing. Keep workloads elsewhere. |

Use OUs to express **policy boundaries**, not just org charts. A common pattern is `Security`, `Infrastructure`, and `Workloads`, with workload OUs split further by environment or regulatory needs. Keep the hierarchy shallow enough that operators can predict inherited policy.

Consolidated billing provides a combined bill and may enable eligible pricing benefits across accounts; it does **not** merge account resources or replace chargeback and budget controls.

## 🛑 Service Control Policies

An **SCP** sets the maximum permissions available to IAM users and roles in member accounts. It is a guardrail around what account administrators can grant, not a permission grant itself. An identity still needs an IAM allow, and an SCP must not block the requested action.

- Attach policies to the organization root, an OU, or an account; the effective boundary follows the hierarchy.
- Explicit denies override allows. A restrictive allow-list SCP can also limit what is possible below it.
- SCPs do not affect principals in the management account and do not restrict service-linked roles.
- Roll out new guardrails in a test OU first; an overly broad deny can block account operations and deployments.
- Keep emergency access and policy-change procedures documented and tested.

Think of the decision as two gates: **identity/resource permissions allow the action**, and **organization guardrails do not deny it**. Both must be true.

## 👤 IAM Identity Center

IAM Identity Center gives people one sign-in experience for access to multiple AWS accounts. Connect it to an identity source, organize people into groups, create **permission sets**, then assign groups to accounts.

1. Put users in identity-provider groups that reflect job responsibilities.
2. Map groups to permission sets such as `ReadOnly`, `Developer`, or a carefully controlled `Administrator` set.
3. Assign those sets to the accounts where the group needs access.
4. Review access regularly and remove assignments when roles change.

Permission sets provision IAM roles in the target accounts and provide temporary credentials. This keeps routine workforce access out of long-lived IAM users and access keys. Use separate, tightly controlled break-glass access for emergencies and monitor its use.

## 🧰 AWS Control Tower

Control Tower helps establish and govern a multi-account landing zone on top of AWS Organizations. It sets up foundational accounts and logging, provides an account provisioning workflow through **Account Factory**, and helps apply **controls** across OUs.

{{< tabs name="Control Types" >}}
{{% tab name="Preventive" %}}
Implemented with policy guardrails, commonly SCPs. They block selected actions before a change can happen.
{{% /tab %}}
{{% tab name="Detective" %}}
Implemented with monitoring and configuration checks. They report noncompliant state for investigation and remediation.
{{% /tab %}}
{{% tab name="Proactive" %}}
Validate supported resources against rules before provisioning through governed workflows. Coverage depends on the resource and control.
{{% /tab %}}
{{< /tabs >}}

Control Tower is a managed path to consistent governance, not a substitute for understanding Organizations, IAM, networking, or service-specific limits. Register and govern OUs deliberately, and test controls against real deployment needs before broad rollout.

## 🚦 CI/CD Deployment Models

Choose a deployment model that matches the number of accounts, required approval points, and the blast radius you can tolerate.

| Model | How it works | Good fit |
| --- | --- | --- |
| Per-account pipeline | Each workload account owns its build and deploy workflow. | Independent teams and strong operational autonomy. |
| Central tooling account | A shared pipeline assumes deployment roles in workload accounts. | Standardized controls and a shared platform team. |
| Hybrid | Shared build and security stages, with team-owned or delegated deployment stages. | Common governance with different release needs. |

Keep the **pipeline definition** separate from **environment configuration**. Promote the same versioned artifact through environments, with explicit approval or automated health checks at higher-risk boundaries. Avoid rebuilding different binaries for production after testing a different artifact in staging.

## 📦 CloudFormation StackSets

CloudFormation StackSets deploy one CloudFormation template to stacks across multiple accounts and Regions. They are useful for repeatable baseline resources such as organization-wide roles, configuration rules, or standard logging components.

- **Service-managed permissions** integrate with Organizations and can target OUs or the organization. This is usually the simpler model for organization-wide rollout.
- **Self-managed permissions** use administrator and execution roles that you manage in the target accounts.
- Configure deployment concurrency and failure tolerance to control rollout risk.
- Use stack instances and drift status to see where the template succeeded and where it did not.

StackSets are a fleet deployment mechanism, not a universal replacement for account vending or application pipelines. Test changes in a small target set, especially when an update can modify or delete shared resources.

## 🏗️ AWS Cloud Development Kit (CDK)

The AWS CDK lets you define infrastructure in a programming language, compose reusable **constructs**, and synthesize CloudFormation templates. It gives teams familiar programming tools, while CloudFormation remains the deployment engine.

```text
CDK source -> tests and policy checks -> cdk synth -> CloudFormation template
           -> reviewed artifact -> deploy to account/Region
```

- Pin dependencies and review synthesized templates as deployment artifacts.
- Use constructs to standardize secure defaults, not to hide important behavior.
- Bootstrap each target account and Region before deployment; bootstrap resources establish the roles and assets CDK uses.
- Separate application configuration from secrets, and use explicit environments for account and Region targeting.
- For organization-wide foundations, compare CDK pipelines, ordinary CI/CD, and StackSets against your ownership and rollout model.

## 🔐 Secrets Manager for IaC

Infrastructure code should describe **how a workload gets a secret**, not contain the secret value. Store credentials and other sensitive values in Secrets Manager, scope access to the consuming role, and enable rotation where the service and application support it.

- Never commit a secret into source, a CDK context file, a template parameter default, a build log, or a plain environment variable in a pipeline definition.
- Prefer runtime retrieval by the workload role when possible; the secret need not pass through the deployment system at all.
- CloudFormation dynamic references can resolve a Secrets Manager value during resource operations. The target service still needs a secure way to consume it, and not every resource or property supports dynamic references.
- For cross-account retrieval, configure the secret resource policy and KMS key policy as well as the caller's IAM permissions. Share only with named accounts or roles.
- Remember that rotation can invalidate a value cached by an application. Design refresh and reconnect behavior, not just the rotation schedule.

{{< accordion "A quick secret-handling test" >}}

Ask: **Could a person with repository, build-log, or template access read the secret?** If yes, move retrieval to the runtime identity or use a supported secret reference with narrowly scoped access.

{{< /accordion >}}

## 🔎 Drift Detection and Remediation

**Drift** is the difference between the state declared in IaC and the live resource configuration. A console change, emergency fix, or another automation system can create it.

1. Detect drift with CloudFormation drift detection, AWS Config rules, or an IaC-specific plan/check.
2. Triage whether the live change was intentional, unauthorized, or temporary.
3. Choose the source of truth: update code and redeploy, or restore the declared state.
4. Record the change and fix the process that allowed unreviewed configuration changes.

Detection coverage varies by resource and property; an `IN_SYNC` result is not proof that every aspect of a workload is correct. Remediation should be risk-aware: automatic correction is useful for safe, well-understood controls, but a blanket revert can interrupt a legitimate incident fix or production service.

## 🛡️ Pipeline Security Scanning

Treat a pipeline as production infrastructure: it can change every account it is trusted by. Add checks before credentials or deployment authority are available wherever possible.

| Stage | Example checks |
| --- | --- |
| Source / pull request | Secret scanning, peer review, branch protection, dependency and license checks. |
| Build | Unit tests, SAST, dependency and container-image scanning. |
| IaC validation | `cdk synth`, `cfn-lint`, policy-as-code checks such as `cfn-guard`, and CDK-specific checks such as `cdk-nag`. |
| Artifact | Immutable/versioned storage, encryption, provenance, and signing where supported. |
| Deployment | Least-privilege target roles, approvals for production, and post-deploy health checks. |

Use short-lived credentials (for example, OIDC federation from an external CI provider), pin third-party actions and build images, restrict who can change pipeline definitions, and prevent untrusted pull-request code from inheriting deployment credentials.

## 🔁 Cross-Account Pipeline Architecture

A central pipeline can deploy across account boundaries without copying human credentials. A typical flow is:

1. A source change starts the pipeline in a dedicated **tooling account**.
2. Build and test stages produce one versioned artifact.
3. The pipeline assumes a narrowly scoped deployment role in a non-production account.
4. After validation and any required approval, it assumes the corresponding production role.
5. The target role deploys the approved artifact and reports status to the pipeline.

The target role's trust policy should name the specific tooling account and, where practical, constrain which pipeline role can assume it. Its permissions should be limited to the deployment's resources and actions. For encrypted artifacts, grant the target role only the required S3 read and KMS decrypt access; the bucket policy, key policy, and IAM policy must agree.

Do not let a build job choose arbitrary target role ARNs or accounts. Make approved targets explicit, log role assumptions, and separate production approval from the identity that can modify the pipeline itself.

## 🚀 Deployment Strategies and Rollback

Use **waves** to increase confidence gradually: a test account, a small canary set, a broader non-production group, then production. Account waves limit the number of environments affected by a bad change.

| Strategy | Traffic or rollout behavior | Rollback consideration |
| --- | --- | --- |
| Rolling | Replace instances or resources in batches. | Capacity and compatibility must hold while old and new versions coexist. |
| Blue/green | Deploy a parallel environment, then shift traffic. | Fast traffic reversal is possible if the old environment remains healthy. |
| Canary | Send a small share of traffic to the new version, then increase it. | Automate metrics-based alarms and stop conditions. |
| All at once | Update all targets in one operation. | Fast, but exposes the full target set to a defect at once. |

Rollback is not always a simple redeploy. Database changes, one-way data migrations, and external side effects may not be reversible. Favor backward-compatible schema changes, keep artifacts immutable, define health metrics and alarms, and document when the correct recovery is **roll forward** instead.

## 🧩 Putting It Together - Finished Architecture

Here is one starting architecture: Organizations and Control Tower govern purpose-built OUs; Identity Center gives people temporary access; a tooling account builds and scans a single artifact, then deploys through constrained cross-account roles; security and log archive accounts centralize oversight.

![AWS multi-account organization and cross-account CI/CD deployment architecture](/images/uploads/aws-multi-account-architecture.svg)

Use the diagram as a conversation starter, not a universal blueprint. Add accounts, network boundaries, and approval steps where different ownership, data sensitivity, or recovery requirements justify them. The strongest design is the one your team can explain, operate, and test.