---
id: stratio-kafka
name: "Stratio Kafka Operator"
aliases: ["kafka-operator", "kafka", "KafkaCluster"]
category: operator
repo: keos-system-services
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Kafka Operator

> A Kubernetes operator that deploys and manages Apache Kafka clusters (brokers and controllers, RAFT mode) as a custom resource.

## 1. Overview

Stratio Kafka Operator manages the full lifecycle of Apache Kafka clusters on Kubernetes. It
introduces a `KafkaCluster` custom resource made of brokers and controllers running in RAFT mode
(no separate ZooKeeper required). Beyond initial deployment it handles monitoring, elastic
scaling (adding/removing nodes, rebalancing), centralized authorization via Stratio GoSec
(embedded directly in Kafka), and audit logging of all Kafka access. The cluster-wide operator
(one per cluster) reconciles the CRDs; tenants that need a Kafka cluster create their own
`KafkaCluster` instance, one or more per tenant. In this fleet's observed snapshot, no tenant
currently has an enabled Kafka instance.

## 2. Classification

| | |
|---|---|
| **Type** | Operator |
| **Deployment scope** | Cluster-wide operator (`kafka-operator`, one instance per cluster) that reconciles tenant-scoped `KafkaCluster` custom resources (one or more per tenant, via a separate `kafka` component in keos-apps) |
| **Home repo** | keos-system-services (operator); keos-apps (tenant `KafkaCluster` instance) |

## 3. Dependencies

```
Stratio Kafka Operator (cluster-wide)
├─ Vault (via the platform Secrets Operator)   (required — KMS/secrets backend)
├─ Prometheus Operator                          (required — metrics)
└─ VPA (Vertical Pod Autoscaler)                (required — per-node-group autoscaling)

Stratio Kafka — KafkaCluster instance (per tenant)
├─ Stratio GoSec (gosec-authz / gosec-services-daas)   (required — centralized authorization + auditing)
└─ Vault (via the platform Secrets Operator)      (required — per-instance secrets)
```

**Used by:**
- None found in this fleet's GitOps configs — no tenant component currently declares a
  dependency on a Kafka instance, and no tenant has one enabled in the observed snapshot.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size (controller + broker node groups) | S \| M \| L | Instance counts, replication factor (1 in S, 3 in M/L), CPU/memory, and storage (10Gi in S up to 40Gi in L). |
| External exposure | on \| off | Whether the Kafka protocol endpoints are reachable outside the cluster (native protocol, not HTTP). |
| Operator itself | fixed, no size tiers | The operator always runs as a single replica regardless of tenant sizing. |

## 5. Ingress / Exposed Endpoints

None in the HTTP sense — Kafka is reached over its native protocol (brokers on port 9092),
optionally exposed externally per tenant via a toggle (not through an Ingress/HTTPRoute). There
is no admin web UI; a small internal REST API handles topic management and health checks but
isn't exposed outside the cluster.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | TLS for client/broker connections |
| **Credentials it holds** | Vault-issued secrets bundles, provisioned separately for the operator and each KafkaCluster instance, via the platform's Secrets Operator |
| **Notable access controls** | Authorization is centralized through a GoSec plugin embedded directly in Kafka (all access is logged); the operator runs with cluster-wide RBAC for reconciling KafkaCluster CRDs, while each instance gets its own dedicated ServiceAccount |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
