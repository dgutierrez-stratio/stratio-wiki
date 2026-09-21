---
id: stratio-intelligence
name: "Stratio Intelligence"
aliases: []
category: business-application
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Intelligence

> A web-based Jupyter data-science environment with direct access to platform data, for exploratory analysis and ML model development.

## 1. Overview

Stratio Intelligence is the data-science and ML development layer of the platform: a web-based
Jupyter Notebook environment, running on the cluster, with direct access to production data
across the platform's data stores. It supports Python, R, and Scala, integrates with Spark for
distributed processing, and provides unified SQL access across data sources. Data scientists use
it to explore data and train models (classification, regression, clustering, anomaly detection,
and more), then publish those models as microservices via Stratio Rocket — giving them the full
lifecycle from development through to production. It also integrates with GenAI's
natural-language-to-SQL "Talk to your data" chains.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Intelligence
├─ Stratio Connectors                                 (required)
├─ Stratio PostgreSQL (pooled through PgBouncer)       (required)
├─ Stratio GenAI                                        (required in every tenant observed)
├─ Stratio Virtualizer                                   (required in every tenant observed)
├─ Stratio OpenSearch                                     (optional — togglable, on in most tenants)
├─ Stratio GenAI LiteLLM                                   (optional — seen in one tenant)
└─ GoSec Postgres/OpenSearch agents                         (optional — conditionally wired per tenant)
```

Also uses an internal S3 bucket as its analytics filesystem — in at least one environment, the
absence of a real S3 bucket was the stated reason for disabling Intelligence entirely.

**Used by:**
- None found. No other component in this fleet's GitOps configs lists Stratio Intelligence as a
  dependency — it's a terminal/leaf application. Its integration with Stratio Rocket (publishing
  trained models as microservices) is a product-level relationship, not a formal GitOps
  dependency edge in either direction.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size | S \| M \| L | Selectable in GitOps, but sizing differences are defined inside the external Helm chart, not visible in this repo. |
| Kernel languages | Python, R, Scala (Scala and SparkR are being deprecated in a future release) | Which languages are available in the notebook environment. |
| Filesystem | Internal S3 bucket (on by default) | Where notebook/analytic-environment files are stored. |
| Enablement | on \| off, per tenant | Whether Intelligence is deployed for a given tenant. |

## 5. Ingress / Exposed Endpoints

Externally exposed as a Jupyter-based web UI, reachable via the platform's admin subdomain (the
exact ingress mechanism is templated inside an external Helm chart, not visible in this repo).
Currently supports multiple auth methods (headers/JWT in addition to SSO); a documented future
release will move authentication exclusively to the platform's SSO via OAuth2Proxy.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown in detail for the current release (auth method templated in an external chart); a documented future release will make SSO via the platform's OAuth2Proxy the only supported method |
| **Credentials it holds** | An auto-generated session cookie-signing secret and credentials for its internal S3 filesystem, synced via the platform's secret-sync mechanism; Vault-issued certificates for GoSec identity, used to provision a default Crossdata catalog/profile/user at install |
| **Notable access controls** | Pod-level Kyverno policies restrict which container images can run in its namespace, block mounting service-account-token secrets into pods, and limit which service accounts can be used at all; optional GoSec agent wiring routes its Postgres/OpenSearch access through GoSec-managed credentials when enabled |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
