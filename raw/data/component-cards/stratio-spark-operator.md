---
id: stratio-spark-operator
name: "Stratio Spark Operator"
aliases: ["spark-operator"]
category: operator
repo: not-migrated
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Spark Operator

> Converts Spark applications into native Kubernetes custom resources — installed automatically by the KEOS installer, not tracked as a Flux-managed component.

**Not tracked in this fleet's GitOps repos — by design, not a migration gap.** Since version
5.0.0, the Spark Operator is automatically installed cluster-wide by the Stratio KEOS installer
(the same installer that Cloud Provisioner hands off to), rather than being deployed as a
HelmRelease in `keos-system-services`. It still runs, in its own `keos-spark` namespace,
alongside the platform's other privileged system services — kyverno's webhook-exclusion lists and
monitoring-dashboard toggles both reference it — but the facts below come from product docs and
indirect fleet signals, not a directly-inspected GitOps manifest.

## 1. Overview

Stratio Spark Operator converts Spark applications into native Kubernetes objects
(`SparkApplication` and `ScheduledSparkApplication` custom resources), so a Spark job is
submitted, scheduled, and monitored the same way as any other Kubernetes resource. It's based on
the Kubeflow/GCP Spark Operator with Stratio-specific hardening: it can run jobs against
different Stratio Spark versions instead of bundling its own, avoids duplicate job launches when
multiple operators watch the same namespace, and can inject HDFS-connection configuration into a
job automatically. Internally it's built from a driver (watches `SparkApplication` events), a
submission runner (executes `spark-submit`), a pod monitor (reports job status back), and a
mutating admission webhook (configures driver/executor pods).

## 2. Classification

| | |
|---|---|
| **Type** | Operator |
| **Deployment scope** | Cluster-wide — one instance per cluster, installed automatically by the KEOS installer (namespace `keos-spark`) |
| **Home repo** | Not tracked as a Flux/GitOps component in this fleet — installed and upgraded by the KEOS installer instead |

## 3. Dependencies

```
Stratio Spark Operator
├─ Per-tenant Kubernetes RBAC (ServiceAccount + Role + RoleBinding)   (required — tenants must
│    create this themselves before submitting jobs)
└─ Stratio Secrets Operator (SecretsBundle)                            (used to provision certs,
     Kerberos principals, and passwords needed by submitted jobs)
```

**Used by:**
- Circumstantial evidence only — Stratio Rocket's namespace policy allowlists Spark
  driver/executor-style container images alongside its own, consistent with Rocket launching
  Spark jobs through this operator, though no formal GitOps dependency declaration exists
  (expected, since job submission happens at runtime, not via a static manifest).

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Namespace scope | All namespaces (default) \| a specific list | Which namespaces the operator will act on. |
| SparkUI exposure | on \| off (default off) | Whether a submitted job's own Spark UI is exposed through the ingress. |
| Resource quota enforcement | on \| off (default off) | Whether the operator enforces resource quotas on submitted jobs. |

## 5. Ingress / Exposed Endpoints

None for the operator itself (it's a controller, not a UI). An optional toggle can expose an
individual submitted job's own Spark UI through the ingress — off by default.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown at the operator level — access is governed by the per-tenant RBAC a tenant must set up before submitting jobs, not by the operator itself |
| **Credentials it holds** | Integrates with the platform's SecretsBundle mechanism to provision certificates, Kerberos principals, and passwords needed by submitted jobs |
| **Notable access controls** | Tenants must create their own ServiceAccount, Role, and RoleBinding scoped to their namespace before the operator will run jobs on their behalf; its namespace (`keos-spark`) is treated as a privileged system namespace, excluded from some cluster-wide admission policies alongside other core platform namespaces |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
