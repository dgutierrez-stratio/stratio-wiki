---
id: stratio-rocket
name: "Stratio Rocket"
aliases: []
category: business-application
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Rocket

> The platform's top-layer data-science and ETL workbench — visual workflows, notebooks, and ML lifecycle management, executed on Spark.

## 1. Overview

Stratio Rocket is a web-based data processing and data-science development and production
environment — the top layer of the platform's data-fabric architecture. It offers a
project-based workspace with several asset types: Workflows (a visual editor for Spark
Streaming/Batch/Structured Streaming ETL pipelines), Notebooks (JupyterLab-based, Python/R/
Scala), AutoMLPipelines (no-code AutoML), and MLModel/MLTrainer/MLProject (an MLflow-based ML
lifecycle). It reads data through Stratio Virtualizer and executes distributed processing via
Stratio Spark, and can publish trained models as standalone microservices exposing a REST API for
production use. It's aimed at data scientists, business analysts, and other users who know the
data well enough to build pipelines and models without deep engineering support.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Rocket
├─ Stratio Connectors                               (required)
├─ Stratio PostgreSQL (pooled through PgBouncer)     (required)
├─ Stratio GenAI                                      (optional — togglable)
├─ Stratio Virtualizer                                 (optional — togglable, commonly enabled)
├─ Stratio HDFS or an internal S3 bucket                (required — one storage backend, chosen per tenant)
├─ Stratio Data Governance's dg-agent                     (commonly wired — described as a "mandatory
│                                                           dependency chain" in one tenant's config)
├─ GoSec Postgres agent                                     (commonly wired, cluster/profile-specific)
└─ Stratio GenAI LiteLLM                                     (seen in some tenant profiles)
```

**Used by:**
- Stratio DLC — executes its data-replication workflows in Rocket
- Stratio Data Marketplace's `datamarket-agent` — depends on Rocket, and Rocket is granted
  catalog-management access as part of that integration
- Models trained in Rocket can be deployed as standalone "mlmodel-server" microservices, which
  pull the trained model artifact from Rocket

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Storage backend | HDFS (Kerberos-secured) \| internal S3 | Where Rocket stores its workflow, notebook, and model artifacts. |
| Size | S \| M \| L | Resource sizing for the main Rocket container and its Debug/Catalog/Validator/External-Service sub-pools. |
| Enablement | on \| off, per tenant | Whether Rocket is deployed for a given tenant. |
| Asset types | Workflows, Notebooks, AutoMLPipelines, MLModel/MLTrainer/MLProject | Which kind of data-science asset a project uses (some, like AutoML via TransmogrifAI, R support, and MLeap, are documented as deprecated in a future release). |

## 5. Ingress / Exposed Endpoints

Presumed externally exposed as a web UI (consistent with docs describing it as a web-based
development environment), with admin-subdomain values passed into its chart — but the exact
ingress host/path and SSO-gating are templated inside an external Helm chart, not visible in this
repo. A separately deployed "mlmodel-server" microservice for a trained model can also be exposed
over HTTP; one release's docs note it being served without TLS.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown for the main Rocket UI/API specifically (not directly visible in this repo) |
| **Credentials it holds** | Credential bundles for both its main service identity and a separate "execution identity" used to run user workflow code, plus (for the S3 backend) internal and customer S3 secrets |
| **Notable access controls** | Two distinct GoSec identities are provisioned — a service identity with broad administration/project-management ACLs (owners-group only), and a separate execution identity; Rocket is also granted an "impersonator" ACL against Virtualizer, and its namespace enforces Kyverno policies restricting which container images and service accounts can run there |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
