# Christ Everywhere Control Plane

**Identity, tenancy, federation, and provisioning across a multi-product application ecosystem.**

> **Architecture case study · private implementation**
>
> The production source repository is intentionally private. This public case study describes the platform architecture and engineering principles without exposing credentials, internal endpoints, database identifiers, security-sensitive implementation details, or unresolved attack surfaces.

## Why This Exists

Christ Everywhere is not one application. It is a family of independently deployable products with different users, workflows, data models, and authorization needs.

The control plane connects that family without turning it into one tightly coupled monolith.

```text
                         CE CONTROL PLANE
              Identity · Tenancy · App Entitlements
                 Organization / Application Graph
                              │
                     least-privilege federation
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
   Christ Everywhere       Koinonia            Ezra
     Community domain    Church operations   Pastor tools
            │                 │                 │
            └──────── each app owns its data ───┘
```

## The Core Design Principle

**The control plane owns relationships between products. Each product owns its domain.**

The platform keeps canonical identity, organization relationships, suite-level application access, cross-product linkage, and provisioning state. It does **not** become a shared application database.

That means:

- Christ Everywhere remains authoritative for community experiences.
- Koinonia remains authoritative for church operations and its own role model.
- Ezra remains authoritative for pastor workflows, content, and local capabilities.
- The control plane answers suite-level questions without absorbing product-level permissions.

## What The Architecture Demonstrates

### Canonical identity without centralized product permissions

A person can be recognized across applications while each application retains authority over what that person may do inside its own domain.

This deliberately separates two questions:

```text
CONTROL PLANE                 APPLICATION
─────────────────────         ─────────────────────
Who is this person?           What can they do here?
Which org is this?            What data can they see?
Which app may they enter?     Which domain actions are allowed?
```

This avoids a central "god service" that must understand every permission in every product.

### Least-privilege application federation

Applications federate through narrow application-specific trust boundaries rather than receiving broad administrative credentials to the control plane.

Design goals include:

- one application cannot impersonate another;
- one compromised application does not automatically imply total control-plane compromise;
- application identity is established by the trust boundary, not by a caller-provided label;
- sensitive platform credentials remain inside the platform boundary;
- individual applications consume only the cross-product facts they actually need.

### Durable, idempotent provisioning

Cross-product provisioning is modeled as orchestration, not a synchronous chain of fragile API calls.

```text
New organization
       │
       ▼
Canonical platform record
       │
       ▼
Durable provisioning work
   ┌───┼───────────────┐
   ▼   ▼               ▼
  CE  Koinonia        Ezra
       │
       └── retries + reconciliation ──► self-healing state
```

The intent is that partial failure is normal and recoverable:

- provisioning operations are safe to retry;
- application outages do not require rolling back the entire ecosystem;
- reconciliation can discover and repair missing projections;
- product read paths do not block on provisioning completing everywhere.

### Explicit ownership boundaries

The architecture uses contracts between independently deployable products rather than cross-database reads.

This matters because a shared database can make integration look easy while silently destroying ownership boundaries, release independence, and failure isolation.

### Human-verifiable integration contracts

Cross-application behavior is treated as a product contract: ownership, source of truth, accepted data shapes, failure behavior, and definitions of done are recorded and verified against running systems rather than inferred from a successful HTTP response.

## Platform Engineering Principles

- **Join on durable IDs, not editable display properties.**
- **Authenticate the caller; do not trust an asserted actor.**
- **Keep product authorization inside the product.**
- **Make retries idempotent.**
- **Prefer reconciliation to distributed rollback.**
- **Keep public/read experiences available during partial provisioning.**
- **Minimize cross-product data movement.**
- **Design integration failures to degrade locally instead of globally.**
- **Audit consequential cross-product actions.**

## Product Ecosystem

| Product | Domain responsibility |
|---|---|
| **Christ Everywhere** | Community, Bible, prayer, media, groups, events, and public community experiences |
| **Koinonia** | Church operations, people, giving, service planning, volunteers, attendance, and worship presentation |
| **Ezra** | Pastor research, sermon preparation, evidence workflows, review, and pastor-owned content |
| **Control Plane** | Canonical identity, tenancy relationships, application entitlements, federation, and provisioning |

## Technology Surface

The implementation uses a lightweight web administration layer, managed relational data services, and server-side/serverless federation and orchestration components. The public case study intentionally stays above production network, credential, and database details.

## What I Owned

Product and platform architecture across:

- domain boundaries and sources of truth;
- identity and federation model;
- cross-application contracts;
- provisioning and reconciliation strategy;
- application-vs-platform authorization boundaries;
- failure isolation and idempotency requirements;
- security and privacy tradeoffs;
- acceptance criteria and live verification strategy.

## Why The Source Is Private

This repository sits at the trust boundary between multiple production applications. Publishing the implementation would expose substantially more security-relevant detail than is necessary to demonstrate the architecture.

For technical diligence, the right progression is:

**architecture case study → live product behavior → controlled design review → selected code walkthrough when appropriate.**