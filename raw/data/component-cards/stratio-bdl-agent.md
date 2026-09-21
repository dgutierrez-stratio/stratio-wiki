---
id: stratio-bdl-agent
name: "Stratio BDL Agent"
aliases: ["eureka-agent", "BDL Agent"]
category: business-application
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio BDL Agent

> The synchronization engine that turns Stratio Data Governance's business semantic definitions into ready-to-query views for analytics and virtualization tooling.

## 1. Overview

Stratio BDL Agent (deployed in GitOps under the component name `eureka-agent`) is the central
synchronization and execution engine for the Business Data Layer (BDL) inside Stratio Generative
AI Data Fabric. It reads collection definitions and optimization metadata from Stratio Data
Governance and syncs them to execution environments — either publishing to a Hive catalog (backed
by PostgreSQL) for consumption by Stratio Virtualizer, or writing directly to PostgreSQL for
native JDBC access, depending on configuration. Once a collection is configured, syncing runs
automatically on a polling basis with no further manual intervention. It depends on a running
Stratio Data Governance instance and a Stratio Virtualizer instance, and is itself configured
indirectly through Stratio Command Center rather than through a UI of its own.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio BDL Agent
├─ Stratio Connectors                                    (required)
├─ Stratio PostgreSQL (pooled through PgBouncer)          (required)
└─ Stratio Data Governance agent                          (required in every observed tenant)
     — flavor varies by tenant: S3-backed or HDFS-backed
```

**Used by:**
- None found. No other component in this fleet's GitOps configs lists Stratio BDL Agent as a
  formal dependency. Stratio Virtualizer and Stratio Rocket are documented as needing matching
  Hive-catalog configuration to consume its output, but that relationship isn't wired as a GitOps
  dependency edge — it's a documented, not a configured, relationship.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Data Governance backend | S3-backed \| HDFS-backed | Which Data Governance agent flavor supplies the collections it syncs — varies per tenant in this fleet. |
| Output mode (documented, not distinguishable in this fleet's configs) | Hive catalog via PostgreSQL \| Direct PostgreSQL | Whether synced data lands in a Hive catalog for Stratio Virtualizer, or is written directly to PostgreSQL for native JDBC access. |
| Size | S \| M \| L | Selectable in GitOps, but all three currently render identically in this fleet — no confirmed behavioral difference. |

## 5. Ingress / Exposed Endpoints

_Internal only — reached directly by other components, nothing external._ No Ingress or
externally reachable UI/API was found. It's configured indirectly through Stratio Command
Center's "Bdl" services section, not through an endpoint of its own.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown — no SSO/OIDC wiring found; access appears to be internal-only credential-based |
| **Credentials it holds** | An S3 credential bundle (per-tenant secret, SOPS-encrypted), a generated database password, a generated keystore password, a TLS certificate, and a Kerberos principal — all issued through the platform's secrets-bundle machinery |
| **Notable access controls** | unknown — no explicit access-control policy found beyond credential issuance |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
