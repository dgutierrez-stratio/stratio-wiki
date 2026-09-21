---
id: stratio-postgres
name: "Stratio PostgreSQL Operator"
aliases: ["postgres-operator", "postgres", "PgCluster", "pgbouncer"]
category: operator
repo: keos-system-services
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio PostgreSQL Operator

> A Kubernetes operator that deploys and manages highly-available PostgreSQL clusters — the platform's default relational datastore, used by nearly every other component.

## 1. Overview

Stratio PostgreSQL Operator manages the full lifecycle of PostgreSQL clusters on Kubernetes. It
introduces a `PgCluster` custom resource (a Patroni-managed primary plus optional replicas) for
automatic failover and read-offload, alongside a separate `PgBouncer` custom resource that pools
connections in front of a given PgCluster. Beyond deployment, it provides continuous monitoring,
elastic scaling, and logical/physical backup and restore. Almost every stateful component in this
fleet — Discovery, Virtualizer, GenAI, BDL Agent, Data Governance, and more — depends on a
PostgreSQL instance pooled through PgBouncer as its relational datastore, and tenants commonly
run more than one isolated Postgres+PgBouncer pair for different purposes (e.g. a dedicated pair
for Data Governance).

## 2. Classification

| | |
|---|---|
| **Type** | Operator |
| **Deployment scope** | Cluster-wide operator (`postgres-operator`, one instance per cluster) that reconciles tenant-scoped `PgCluster` and `PgBouncer` custom resources (via separate components in keos-apps, one or more pairs per tenant) |
| **Home repo** | keos-system-services (operator); keos-apps (tenant PgCluster and PgBouncer instances) |

## 3. Dependencies

```
Stratio PostgreSQL Operator (cluster-wide)
├─ Vault (via the platform Secrets Operator)   (required — KMS/secrets backend)
├─ Prometheus Operator                          (required — metrics)
└─ VPA (Vertical Pod Autoscaler)                (required — per-instance autoscaling)

Stratio PostgreSQL — PgCluster instance (per tenant)
├─ Stratio PostgreSQL Operator                    (required — reconciles the cluster)
├─ Vault (via the platform Secrets Operator)       (required — per-instance secrets)
└─ Stratio PgBackupRepository (optional)            (required only when backups are enabled)

Stratio PostgreSQL — PgBouncer pooler (per PgCluster)
└─ The PgCluster it pools connections for            (required)
```

**Used by:**
- Nearly every stateful component in this fleet — confirmed direct dependents include Stratio
  Discovery, Stratio Virtualizer, Stratio GenAI, Stratio BDL Agent (`eureka-agent`), Stratio
  Data Governance (via a dedicated pair), Stratio DataREST, Stratio DLC, and Stratio Data
  Marketplace's `datamarket-agent`, among others. It's the platform's default relational
  datastore rather than something with a short, enumerable dependent list.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size | S \| M \| L | Instance count (1 in S, 3 in M/L), PostgreSQL tuning parameters, and storage (40Gi/100Gi/200Gi data + 20Gi/50Gi/100Gi WAL). |
| Backup mechanism | Velero (cluster-resource backup) \| pgBackRest (logical or physical, local or S3) | How and where the database itself is backed up, separate from Kubernetes-resource backup. |
| Backup encryption | on \| off (off by default) | Whether pgBackRest backups are encrypted. |
| Client authentication flavor (application-side, not a Postgres deployment option) | pginternal \| pgmd5 \| pgtls | How a consuming application authenticates to a given Postgres/PgBouncer instance — internal Stratio-managed secrets/certs, plain MD5 password, or TLS. |

## 5. Ingress / Exposed Endpoints

None — PostgreSQL is reached over its native wire protocol (5432), not HTTP, and there's no
admin UI (no pgAdmin) in this fleet. PgBouncer has an optional external-exposure toggle (via
LoadBalancer/external-DNS, not an HTTP Ingress) for cases where external clients need direct
protocol access through the pooler — off by default.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Certificate-based (mutual TLS) authentication is the deployed default for both PgCluster and PgBouncer |
| **Credentials it holds** | Vault-issued secrets bundles for the operator and each PgCluster/PgBouncer instance; S3-backed physical backups provision their own separate archiver credentials |
| **Notable access controls** | A dedicated GoSec agent enforces GoSec policies inside PostgreSQL for tenants that enable it; both PgCluster and PgBouncer run at an elevated priority class with pod disruption budgets enabled (except the smallest size tier) |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
