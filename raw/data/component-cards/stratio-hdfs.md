---
id: stratio-hdfs
name: "Stratio HDFS Operator"
aliases: ["hdfs-operator", "hdfs", "HDFSCluster"]
category: operator
repo: keos-system-services
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio HDFS Operator

> A Kubernetes operator that deploys and manages HDFS clusters (journalnodes, namenodes, datanodes, and an embedded Zookeeper) as a custom resource.

## 1. Overview

Stratio HDFS Operator manages the full lifecycle of HDFS (Hadoop Distributed File System)
clusters on Kubernetes. It introduces a custom resource, `HDFSCluster`, that describes a cluster
made of journalnodes, namenodes, and datanodes, plus an embedded Zookeeper cluster used solely
for HDFS's own internal coordination (not for general-purpose use). Beyond initial deployment, it
handles ongoing monitoring, elastic scaling (adding/removing nodes and rebalancing data),
centralized authorization and auditing through Stratio GoSec, and Kerberos-based authentication.
The cluster-wide operator (one per cluster) reconciles the CRDs; each tenant that needs HDFS
storage creates its own `HDFSCluster` instance, which the operator then turns into a running
cluster in that tenant's namespace.

## 2. Classification

| | |
|---|---|
| **Type** | Operator |
| **Deployment scope** | Cluster-wide operator (`hdfs-operator`, one instance per cluster) that reconciles tenant-scoped `HDFSCluster` custom resources (deployed via a separate `hdfs` component in keos-apps, one per tenant that needs HDFS) |
| **Home repo** | keos-system-services (operator); keos-apps (tenant `HDFSCluster` instance) |

## 3. Dependencies

```
Stratio HDFS Operator (cluster-wide)
├─ Vault (via the platform Secrets Operator)   (required — KMS/secrets backend)
├─ Prometheus Operator                          (required — metrics)
└─ VPA (Vertical Pod Autoscaler)                (required — per-node-group autoscaling)

Stratio HDFS — HDFSCluster instance (per tenant)
├─ Cluster-wide Kerberos configuration           (required — authentication)
├─ Stratio GoSec (gosec-authz / gosec-services-daas)   (required — centralized authorization + auditing)
└─ Vault (via the platform Secrets Operator)      (required — per-instance secrets)
```

**Used by:**
- Stratio Data Governance's dg-agent (HDFS-backed flavor, `dg-hdfs-agent`)
- Stratio Virtualizer (HDFS-backed flavor)
- Stratio Spark History (HDFS-backed flavor)
- Stratio Rocket (HDFS-backed storage flavor)
- Transitively, Stratio BDL Agent (`eureka-agent`) — via its `dg-hdfs-agent` dependency

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size (per node group: journalnode, namenode, datanode, and the embedded Zookeeper) | S \| M \| L | Instance counts, CPU/memory, and storage per node type — e.g. the datanode group goes from 1 instance / 2Gi memory in S to 5 instances / 8Gi memory + 500Gi storage in L. |
| Operator itself | fixed, no size tiers | The operator always runs as a single replica regardless of tenant sizing. |

## 5. Ingress / Exposed Endpoints

None — no admin UI or externally exposed API for either the operator or a tenant's HDFS cluster.
Node-level health/admin functionality (decommission/recommission, status) is served internally via
a REST API, not exposed outside the cluster.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Kerberos, wired to a cluster-provided Kerberos realm/config shared by other HDFS-dependent components; the embedded Zookeeper sub-cluster uses TLS instead, with its own separate (non-GoSec) authorization model |
| **Credentials it holds** | Vault-issued secrets bundles, provisioned separately for the operator, each HDFSCluster instance, and the embedded Zookeeper sub-cluster, via the platform's Secrets Operator |
| **Notable access controls** | Authorization and auditing are centralized through Stratio GoSec (all HDFS accesses are logged); the operator runs with cluster-wide RBAC for reconciling HDFSCluster CRDs, while each instance gets its own narrowly-scoped ServiceAccount |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
