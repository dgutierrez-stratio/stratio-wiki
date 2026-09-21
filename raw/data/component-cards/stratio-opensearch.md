---
id: stratio-opensearch
name: "Stratio OpenSearch Operator"
aliases: ["opensearch-operator", "opensearch", "OsCluster"]
category: operator
repo: keos-system-services
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio OpenSearch Operator

> A Kubernetes operator that deploys and manages OpenSearch clusters and their Dashboards UI as custom resources.

## 1. Overview

Stratio OpenSearch Operator manages the full lifecycle of OpenSearch clusters (and OpenSearch
Dashboards, its admin UI) on Kubernetes. It introduces an `OsCluster` custom resource made of
Master, Data, Coordinator, and (newer) ML/Search-role nodes, running in a chosen namespace for
high availability. Applications on the same cluster can only reach OpenSearch through services
this operator manages. Beyond deployment, it handles monitoring, elastic scaling, centralized
authorization via Stratio GoSec, and audit logging of all access. The cluster-wide operator (one
per cluster) reconciles the CRDs; tenants that need OpenSearch create their own `OsCluster`
instance, alongside optional add-on components for Dashboards and backup/restore.

## 2. Classification

| | |
|---|---|
| **Type** | Operator |
| **Deployment scope** | Cluster-wide operator (`opensearch-operator`, one instance per cluster) that reconciles tenant-scoped `OsCluster`, `OsDashboards`, and `OsBackup` custom resources (via separate components in keos-apps, one per tenant that needs OpenSearch) |
| **Home repo** | keos-system-services (operator); keos-apps (tenant OsCluster/Dashboards/backup instances) |

## 3. Dependencies

```
Stratio OpenSearch Operator (cluster-wide)
├─ Vault (via the platform Secrets Operator)   (required — KMS/secrets backend)
├─ Prometheus Operator                          (required — metrics)
└─ VPA (Vertical Pod Autoscaler)                (required — per-node-group autoscaling)

Stratio OpenSearch — OsCluster instance (per tenant)
├─ Stratio SIS / platform SSO (OIDC)             (required — authentication)
└─ Vault (via the platform Secrets Operator)      (required — per-instance secrets)
```

Optional S3 backend for backups, wired through its own secrets bundle.

**Used by:**
- Stratio GenAI (optional, togglable) — plus a separate connection to a Data-Governance-owned
  OpenSearch cluster for its governance-search integration
- Stratio Intelligence (optional, togglable)
- Stratio Data Governance's search-engine sub-component — required, with a dedicated HA-tuned
  OpenSearch profile
- GoSec agent (OpenSearch flavor) — registers an OpenSearch instance into GoSec's authorization
  model

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size | S \| M \| L | Instance count and resource sizing per data node. |
| Deployment profile | Generic default \| Governance-searcher (dedicated HA topology with role-split node groups) | Whether OpenSearch runs a general-purpose default profile or a purpose-built high-availability profile for Data Governance's search engine. |
| Backup target | Local PVC \| S3 | Where OsCluster backups are stored. |
| Node roles | Master, Data, Coordinator, plus newer ML and Search roles (for GenAI Document Search) | What role a given node group plays in the cluster. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| OpenSearch Dashboards (admin UI), under the platform's admin subdomain | SSO via Stratio SIS (OIDC) | Browsing and managing indices, dashboards, and cluster state as an authenticated user. |

The core OsCluster itself is not exposed externally by default — only Dashboards is.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Node-to-node/transport TLS for the core cluster; SSO via Stratio SIS (OIDC) for both the cluster's own security layer and Dashboards |
| **Credentials it holds** | Vault-issued secrets bundles per operator and per OsCluster instance; S3 backup credentials provisioned separately when the S3 backup flavor is used |
| **Notable access controls** | A dedicated GoSec agent runs alongside each OsCluster to push GoSec authorization policies into OpenSearch, and all access is logged; the operator runs with cluster-wide RBAC for reconciling its CRDs, while each instance gets its own dedicated ServiceAccount |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
