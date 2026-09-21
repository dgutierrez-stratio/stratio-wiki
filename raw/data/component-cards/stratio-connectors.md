---
id: stratio-connectors
name: "Stratio Connectors"
aliases: []
category: system-service
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Connectors

> Hosts the driver/JAR artifacts that let other tenant components reach external and internal data sources.

## 1. Overview

Stratio Connectors packages and serves the driver artifacts ("connectors") that let other
components of the Stratio Generative AI Data Fabric platform reach external data sources —
relational databases, object storage, distributed file systems, and more — in a way that's
transparent to the rest of the platform. Other tenant components (Stratio BDL Agent, Stratio Data
Governance's dg-agent, Stratio Virtualizer, Stratio Rocket, Stratio Intelligence, Stratio Spark
History) pull the connector JARs they need from this component rather than bundling their own
drivers. Data-source credentials used by a given connector are managed separately, through the
platform's Stratio KMS, not by Connectors itself. As of this release, its REST API is not exposed
outside the cluster by default, closing a previously mitigated broken-access-control
vulnerability.

## 2. Classification

| | |
|---|---|
| **Type** | System service (tenant-scoped internal glue) |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Connectors
└─ (none) — self-contained; bundles its driver JARs directly with no declared
   dependency on another platform component
```

**Used by:**
- Stratio BDL Agent (`eureka-agent`)
- Stratio Data Governance's data-discovery agent (`dg-agent`, both S3- and HDFS-backed flavors)
- Stratio Virtualizer
- Stratio Rocket (most tenants observed)
- Stratio Intelligence (most tenants observed)
- Stratio Spark History (some tenants observed)

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Connector / driver type | S3, ADLS2, GCS, JDBC/CData, Hive, Impala, MSSQL, PostgreSQL, BigQuery, Elasticsearch, MongoDB, Oracle, Db2, plus several types slated for removal in a future release (ArangoDB, Cassandra, ClickHouse, SAP HANA, SAS, Teradata) | Which external or internal data sources other components can reach through this instance. |
| Size | S \| M \| L | Currently identical in this fleet — no resource-tier differentiation observed. |
| API exposure | Internal-only (default) \| exposed via Ingress (opt-in) | Whether the Connectors REST API is reachable from outside the cluster — off by default as a security hardening measure; not seen enabled in this fleet. |

## 5. Ingress / Exposed Endpoints

_Internal only — reached directly by other components, nothing external in this fleet's observed
deployments._ Exposing the API externally is a documented opt-in option, not enabled anywhere
observed here.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown — no auth wiring found in Connectors' own deployment config |
| **Credentials it holds** | none found in its own deployment — actual data-source credentials used by a given connector are managed externally via Stratio KMS, not stored by Connectors itself |
| **Notable access controls** | A recent release mitigated a Broken Access Control (A01:2025) vulnerability in its REST API by closing default external exposure |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
