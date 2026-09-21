---
id: stratio-genai
name: "Stratio GenAI"
aliases: []
category: business-application
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio GenAI

> The platform's generative-AI application — lets users query governed data in natural language and build/deploy no-code AI agents.

## 1. Overview

Stratio GenAI is the generative-AI application within Stratio Generative AI Data Fabric. It
offers two main capabilities: "Talk to your data," which translates natural-language questions
into SQL, runs them through Stratio Virtualizer under the asking user's own identity, and returns
text, tables, or charts (which can feed dashboards); and Stratio Cowork, a no-code platform for
building and deploying AI agents, skills, and MCP servers, each running in its own isolated
sandbox. It also exposes over 30 MCP tools that any MCP-compatible agent — internal or external —
can use to interact with governed data. All of GenAI's calls to language models and external
tool/API credentials are routed through a sibling component, Stratio GenAI LiteLLM, which is the
only part of the platform that holds LLM-provider credentials directly.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio GenAI
├─ Stratio GenAI LiteLLM                          (required — the sole holder of LLM-provider
│                                                    credentials; routes all model/MCP calls)
├─ Stratio PostgreSQL (pooled through PgBouncer)   (required — stores conversation history/results,
│                                                    isolated per user)
├─ Stratio Virtualizer                             (required — governed-data queries run through it
│                                                    under the asking user's own identity)
├─ Stratio OpenSearch                              (optional — togglable)
└─ Stratio Discovery                               (optional — togglable)
```

**Used by:**
- Stratio Rocket — depends on GenAI (togglable) and on LiteLLM
- Stratio Intelligence — depends on GenAI and LiteLLM

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| LLM provider (via LiteLLM) | OpenAI, Azure OpenAI, Amazon Bedrock, Google Vertex AI, on-premise vLLM, other certified providers, or a fully offline/air-gapped setup | Which model backend GenAI's chat, governance, and translation features actually call. |
| Model per function | Separate model choices for chat, governance, and translation roles | Lets different GenAI features use different underlying models. |
| Size | S \| M \| L | Resource sizing tier. |
| Legacy GenAI Gateway | Still required for older Rocket/Intelligence deployments with GenAI enabled; new deployments should use LiteLLM instead | Which LLM-routing component a given tenant's Rocket/Intelligence integration uses. |

## 5. Ingress / Exposed Endpoints

Externally exposed as a browser UI (grouping "Talk to your data" and Stratio Cowork). For each
running Cowork agent project, GenAI's own API dynamically provisions its own Service and Ingress
at runtime so the GenAI UI can reach the agent's sandbox from the user's browser — these
per-agent endpoints aren't static GitOps manifests. The exact ingress mechanism for the main
GenAI UI/API is templated inside an external Helm chart, so specific auth annotations weren't
directly visible in this repo, though the platform's identity/impersonation model (see Security)
implies SSO-gated access consistent with the rest of the platform.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | unknown for the main UI/API ingress specifically (not directly visible in this repo); LiteLLM performs mutual TLS and SSO integration with GoSec for the calls it makes on GenAI's behalf |
| **Credentials it holds** | A Vault-issued TLS certificate and keystore password per release; LLM-provider API keys are deliberately NOT held here — they live only in the sibling LiteLLM component |
| **Notable access controls** | GenAI's API service account is granted an "impersonator" GoSec ACL against Virtualizer, so governed-data queries run under the asking user's own identity rather than a shared service identity; per-agent API keys with scoped MCP permissions are issued for agentic workflows. Docs explicitly warn that GenAI's custom-chain/Cowork developer framework is not supervised by Stratio tools — anyone building a custom chain or agent is responsible for what data they choose to share with it |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
