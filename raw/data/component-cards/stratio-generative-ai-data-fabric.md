---
id: stratio-generative-ai-data-fabric
name: "Stratio Generative AI Data Fabric"
aliases: ["Data Fabric", "the platform"]
category: unknown
repo: not-migrated
scope: unknown
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Generative AI Data Fabric

> The umbrella product name for the whole platform and release train — not a component deployed on its own.

**Not a discrete GitOps deployable — this is the umbrella product/release name.** "Stratio
Generative AI Data Fabric" is the name of the whole platform (and its coordinated release train,
e.g. release 14.10), not a component with its own HelmRelease or Kustomization. The product docs
themselves describe it as three groups of already-separately-cataloged components: **Core**
(Cloud Provisioner, KEOS, GoSec, Command Center, Kafka/HDFS/OpenSearch/PostgreSQL Operators),
**Data management** (Data Governance, Data Marketplace, Virtualizer, Spark, Connectors, DLC), and
**Data exploitation** (Intelligence, Rocket, Discovery, GenAI). See each of those components' own
card for real GitOps-confirmed facts — the Dependencies, Used by, Ingress, and Security sections
below genuinely don't apply to this umbrella entry.

## 1. Overview

Stratio Generative AI Data Fabric is Stratio's platform for data governance and management
through generative AI, built on Stratio KEOS (a Kubernetes-based ecosystem deployable in cloud or
on-premise environments). It's a coordinated release train — release 14.10, for example, bundles
specific versions of GenAI, Data Governance, Data Marketplace, Virtualizer, DLC, the KEOS
installer, and Spark, among others. The product docs organize its components into three groups:
Core (cluster/security/operator infrastructure), Data management (governance, cataloging,
virtualization, connectivity), and Data exploitation (the user-facing analytics and AI
applications). Each of those components is documented and deployed independently; this entry
exists only to record that relationship, not as a thing with its own configuration.

## 2. Classification

| | |
|---|---|
| **Type** | unknown — not a Kubernetes workload; the umbrella product/release name |
| **Deployment scope** | Not applicable — it has no discrete GitOps identity of its own |
| **Home repo** | Not applicable |

## 3. Dependencies

Not applicable — see each individually-cataloged component (Stratio BDL Agent, Cloud Provisioner,
Command Center, Connectors, Data Governance, Data Marketplace, DataREST, Discovery, DLC, GenAI,
GoSec, and the rest of the catalog) for real, GitOps-confirmed dependency graphs.

**Used by:**
- Not applicable.

## 4. Flavors & Deployment Options

Not applicable as a deployable. The product docs describe deployment-topology guidance
(namespace/tenant layout patterns, e.g. standard vs. billing-oriented tenant structures) for its
constituent components, not flavors of the umbrella product itself.

## 5. Ingress / Exposed Endpoints

Not applicable — it has no ingress of its own; each constituent component exposes (or doesn't
expose) its own endpoints, documented on its own card.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Not applicable |
| **Credentials it holds** | Not applicable |
| **Notable access controls** | Not applicable — each constituent component manages its own |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
