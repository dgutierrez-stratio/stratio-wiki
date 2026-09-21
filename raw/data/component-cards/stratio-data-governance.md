---
id: stratio-data-governance
name: "Stratio Data Governance"
aliases: ["governance", "dg-agent", "dg-s3-agent", "dg-hdfs-agent"]
category: business-application
repo: keos-apps
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Data Governance

> The platform's data catalog, business glossary, and governance engine — discovers and catalogs data automatically, and hosts Data Marketplace as a plugin.

## 1. Overview

Stratio Data Governance implements data governance for the platform: it catalogs data, assigns
ownership, and manages data through its lifecycle across four areas — a Data Catalog (technical
and business-layer metadata, feeding the Business Data Layer), Data Market (a plugin exposing a
marketplace for data producers and consumers — see Stratio Data Marketplace), an
Ontology/Knowledge Graph layer, and a shared Business Glossary. A cluster-wide governance core
service (one instance per cluster) hosts the catalog, glossary, and marketplace UI/API; separate
tenant-scoped "dg-agent" instances (backed by S3 or HDFS, one per tenant) run automated discovery
agents that scan a tenant's data stores every 30 seconds by default and feed results back into
the shared catalog.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Cluster-wide core service (`governance`, one instance per cluster) with a tenant-scoped, per-backend discovery agent (`dg-agent`, deployed as `dg-s3-agent` or `dg-hdfs-agent` per tenant) |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Data Governance — governance (core)
├─ Stratio PostgreSQL (pooled through PgBouncer)              (required)
└─ Stratio Search Engine (OpenSearch)                          (optional — togglable)

Stratio Data Governance — dg-agent (per tenant)
├─ Stratio Connectors                                          (required)
├─ Stratio PostgreSQL (governance's own database, via PgBouncer)  (required)
└─ Stratio HDFS                                                 (required — HDFS-backed flavor only)
```

**Used by:**
- Stratio BDL Agent (`eureka-agent`) — depends on the tenant's dg-agent instance
- Stratio Rocket — depends on the tenant's dg-agent instance
- Stratio Virtualizer — requires the governance core service to be installed first
- Stratio Data Marketplace's `datamarket-agent` — calls governance's API post-install to assign
  producer/consumer roles
- Stratio GenAI — integrates functionally with governance (routing/config variables), not wired
  as a formal chart dependency

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Discovery agent backend | S3-backed (`dg-s3-agent`) \| HDFS-backed (`dg-hdfs-agent`, Kerberos-authenticated) | Which storage system the tenant's discovery agent scans and authenticates against. |
| Data Marketplace plugin | on \| off | Whether the marketplace producer/consumer UI and API ship alongside the core catalog. |
| Ontology / Knowledge Graph API | on \| off | Whether the business-concept ontology layer is enabled. |
| Size | S \| M \| L | Resource sizing tier, applied per sub-service (catalog, discovery-manager, ontology API, marketplace UI/API). |
| Discovery interval | default 30 seconds, configurable at install | How often discovery agents rescan connected data stores. |

## 5. Ingress / Exposed Endpoints

No externally-exposed Ingress/Route was confirmed in this fleet's GitOps configs for the
governance core service — only internal NetworkPolicy rules allowing other namespaces' pods to
reach its catalog/discovery/marketplace services. It does have its own web UI (a "homepage" with
a Data Catalog section) reachable by authenticated users, but the exact ingress mechanism is
templated inside an external Helm chart not visible in this repo.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown — no direct SSO/ingress wiring found in this repo for the core service, though it requires "a valid user with permissions to log in"; GoSec-issued user certificates are used to authorize API access (a recent release note describes certificate-based API auth replacing a cookie) |
| **Credentials it holds** | A Vault-issued TLS certificate provisioned per post-install job via the platform's secrets-identity mechanism; the S3-backed dg-agent additionally holds an S3 credentials bundle, while the HDFS-backed dg-agent uses Kerberos authentication instead of a stored credential |
| **Notable access controls** | GoSec groups/roles are provisioned per tenant during install (e.g. mapping a tenant's users/semantic-users groups to built-in governance roles); a post-install job also grants a GenAI-agents group access via GoSec |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
