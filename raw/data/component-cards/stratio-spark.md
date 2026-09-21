---
id: stratio-spark
name: "Stratio Spark"
aliases: ["spark-history"]
category: system-service
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Spark

> The data processing engine (Apache Spark) used across the platform; its one concretely-provisioned piece in this fleet is the Spark History Server, a monitoring UI for completed Spark jobs.

## 1. Overview

Stratio Spark is the platform's data processing engine, based on Apache Spark, with
Stratio-specific hardening: Vault-based identity and secrets access, Kerberos-secured HDFS
access, mutual-TLS Postgres access, an SSO-gated job-history UI, and Prometheus/Grafana metrics.
It's used to launch jobs by Stratio Virtualizer, Stratio Intelligence, and Stratio Rocket — each
embeds and runs Spark directly (visible as an allowed container image in their pod policies, or
as embedded executor-sizing config), without a separate GitOps component of their own for that
usage. The one piece of Stratio Spark deployed as its own tenant-scoped application is Spark
History Server: a monitoring UI that shows completed and running Spark application details.
(Spark job submission and lifecycle management on Kubernetes is handled by the separate Stratio
Spark Operator, documented on its own card.)

## 2. Classification

| | |
|---|---|
| **Type** | System service (tenant-scoped internal glue — the job-history/monitoring UI for a runtime consumed directly by other applications) |
| **Deployment scope** | Tenant — one instance per tenant that enables it (Spark History Server); the broader Spark runtime itself has no separate GitOps identity — it's embedded directly into Virtualizer, Intelligence, and Rocket |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Spark — Spark History Server (per tenant)
├─ Stratio Connectors                     (required, across all storage flavors)
└─ Stratio HDFS (Kerberos-secured)          (required — only for the HDFS storage flavor)
```

S3 and GCS storage flavors use internally-managed cloud credentials instead of an HDFS
dependency.

**Used by:**
- Per product docs, the underlying Spark runtime is used to launch jobs by Stratio Virtualizer,
  Stratio Intelligence, and Stratio Rocket — confirmed in GitOps by Rocket's and Intelligence's
  namespace policies explicitly allowlisting the Spark runtime's container image, and by
  Spark-executor sizing configuration embedded directly in Virtualizer's own deployment.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Storage backend (Spark History Server) | HDFS (Kerberos-secured) \| internal S3 \| internal GCS | Where Spark event logs are stored and read from. |
| Size | S \| M \| L | Resource sizing tier. |
| Enablement | on \| off, per tenant | Whether Spark History Server is deployed for a given tenant. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| Spark History Server UI, exposed via the platform's Admin Router over TLS | SSO (only registered users can view job history) | Viewing completed and running Spark job details. |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | SSO, via OAuth client credentials issued for GoSec SSO integration |
| **Credentials it holds** | A client-server TLS certificate; a Kerberos keytab for HDFS access (HDFS flavor); a JWT signing/encryption secret; storage credentials (internal S3/GCS secret bundles for the respective flavors) |
| **Notable access controls** | The broader Spark runtime (as consumed by Virtualizer/Intelligence/Rocket) uses Vault-based Kubernetes-identity role auth to access secrets and mutual-TLS for Postgres access, per product docs |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
