---
id: cloud-provisioner
name: "Stratio Cloud Provisioner"
aliases: []
category: unknown
repo: not-migrated
scope: unknown
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Cloud Provisioner

> A CLI tool that bootstraps a new Kubernetes cluster and its cloud infrastructure, before GitOps takes ownership of it.

**Not found in the GitOps repositories** — this is expected, not a migration gap. Stratio Cloud
Provisioner is a standalone CLI tool run once, off-cluster, to create a new Kubernetes cluster
(and its supporting cloud infrastructure) before Flux/GitOps takes ownership of it. It hands off
to the Stratio KEOS installer once the raw cluster exists, and is never deployed as an ongoing
workload in `keos-apps` or `keos-system-services`. Fields below marked `unknown` reflect that
architectural fact, not an unconfirmed detail about a real deployment.

## 1. Overview

Stratio Cloud Provisioner is a command-line tool that automates the deployment and management of
Kubernetes clusters across multiple cloud providers (AWS via EKS, Microsoft Azure, and Google
Cloud Platform via GKE). An operator runs it from a Linux machine with Docker installed to stand
up a new cluster from a descriptor file — provisioning nodes, CNI, storage classes,
cluster-autoscaling, and load-balancer controllers — then hands off to the Stratio KEOS installer
to complete the platform setup. It exists to give a consistent, repeatable cluster-bootstrap
experience across cloud providers, not to run as an ongoing service.

## 2. Classification

| | |
|---|---|
| **Type** | unknown — not a running Kubernetes workload; a CLI tool run by an operator |
| **Deployment scope** | Unknown — not confirmable without GitOps coordinates |
| **Home repo** | Not migrated to GitOps yet |

## 3. Dependencies

```
unknown — not deployed as an ongoing GitOps workload in this fleet (see callout above), so
there is no dependency wiring to confirm.
```

**Used by:**
- None found in this fleet's GitOps configs — it doesn't run as an ongoing workload that other
  components could depend on.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| unknown | unknown | Not confirmable without GitOps coordinates — consult the product docs directly for its cloud-provider and cluster-flavour options. |

## 5. Ingress / Exposed Endpoints

unknown — not applicable in the usual sense: it's a CLI binary run by an operator, not a deployed
service, so there is no Ingress/Service to confirm either way.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown |
| **Credentials it holds** | unknown |
| **Notable access controls** | unknown |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
