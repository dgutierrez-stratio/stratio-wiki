---
id: stratio-virtualizer
name: "Stratio Virtualizer"
aliases: ["Crossdata"]
category: system-service
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Virtualizer

> A Spark-based data virtualization/federation engine (formerly branded Crossdata) that unifies SQL access across heterogeneous data sources for nearly every other platform component.

## 1. Overview

Stratio Virtualizer is a data federation tool, based on Apache Spark, that aggregates data from
different sources — HDFS, PostgreSQL, Oracle, and other JDBC sources — behind a single unified
SQL interface, with JDBC/ODBC drivers for BI tools and row/column-level security. It pushes
supported queries down directly to the source data store rather than always routing through
Spark, and its catalog integrates with Stratio Data Governance so definitions can be shared
across the platform. It's the evolution of a product formerly called Crossdata — that identity
still shows up internally, e.g. as the "engine" name other components register it under, and as
the resource type used in its own GoSec access-control policies. It has no UI or external
exposure by default; nearly every other application-layer component in this fleet queries it
directly as an internal service, with each caller granted its own scoped "impersonator"
permission so queries run under the identity of the end user who triggered them.

## 2. Classification

| | |
|---|---|
| **Type** | System service (a data-virtualization layer other applications build on, not exposed externally by default) |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Virtualizer
├─ Stratio PostgreSQL (pooled through PgBouncer)   (required — catalog/metadata backend)
├─ Stratio Connectors                                (required — for the S3/GCS storage flavors)
└─ Stratio HDFS                                        (required only for the HDFS storage flavor)
```

A documented "Data Governance must be installed before Virtualizer" claim (found in a training
doc elsewhere) is not corroborated by any GitOps dependency wiring — Virtualizer's catalog
integration with Data Governance appears to be a soft, optional relationship, not a hard
install-order dependency.

**Used by:**
- Stratio GenAI, Stratio Intelligence, Stratio Rocket — direct Helm-level dependency
- Stratio Discovery — wired via tenant config and registered as a data source at install time
- Stratio Data Marketplace's `datamarket-agent` — direct dependency
- All of the above are additionally granted their own "impersonator" GoSec permission against
  Virtualizer, so their queries run under the requesting end user's identity rather than a
  shared service identity

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Storage source | HDFS (Kerberos-secured) \| internal S3 \| internal GCS | Which underlying storage Virtualizer's event logs / default file system use. |
| Size | S \| M \| L | Number of Spark executors and their CPU/memory allocation. |
| Monitor / UI | on \| off (off by default) | Whether an optional monitoring dashboard and admin UI are deployed alongside the core query service. |
| Default install | Single Spark executor, no HA, native queries capped at 10,000 rows, mutual TLS on, no external exposure | The out-of-the-box configuration before any sizing/exposure customization. |

## 5. Ingress / Exposed Endpoints

None by default — the default install documents "no external exposure: the service is not
exposed outside the cluster." Other tenant applications reach it internally as a cluster service
on a fixed query port. An optional UI/monitoring add-on exists, but its exposure and SSO-gating
aren't confirmed in this repo.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Mutual TLS for both client connections and server-to-server (Akka/Hazelcast) traffic; Kerberos for the HDFS storage flavor |
| **Credentials it holds** | Vault-backed certificates, a Kerberos keytab (HDFS flavor), LDAP credentials, and storage credential bundles for the S3/GCS flavors |
| **Notable access controls** | GoSec manages Virtualizer-specific resource types (Datastore, Database, Table, UDF, and Impersonator); each consuming application is granted its own scoped "impersonator" ACL, identified by its client TLS certificate, so it can act on behalf of the end user rather than a shared service identity — this is Virtualizer's core access-control model, not an incidental grant |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
