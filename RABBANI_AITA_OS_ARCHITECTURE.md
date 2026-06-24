# RABBANI AITA OS — ENTERPRISE ARCHITECTURE (LOCKED)

| Field | Value |
|---|---|
| Project | RABBANI AITA OS |
| Document | Master Architecture Specification |
| Status | **LOCKED — v1.0** (source of truth for all future increments) |
| Author | Lead Enterprise Architect |
| Change rule | This document may be **extended** but its locked decisions may **not** be redesigned. Any change requires a formal `vNext` amendment appended to §11, never a rewrite. |

> **Domain assumption (explicit):** RABBANI AITA OS is treated as a **horizontal, multi-tenant AI-Agent Operating System** — a platform on which organizations run AI agents, automated workflows, and knowledge operations. Any vertical (education, support, HR, ops, etc.) attaches as a **Domain Module** that extends the kernel. The kernel defined below is fixed.

---

## 0. Locked Principles (read first)

1. **Kernel vs Modules.** The *Platform Kernel* (tenancy, identity, auth, agents, workflows, knowledge, data spine, billing, audit) is immutable. New capability = new **Module**, never a kernel redesign.
2. **Extension over modification.** Future increments add tables, endpoints, roles-via-permissions, workflow node types, and agent tools. They never rename/repurpose existing ones.
3. **Tenant isolation is non-negotiable.** Every tenant-scoped row carries `tenant_id`; Postgres Row-Level Security (RLS) enforces it at the database layer.
4. **Everything is auditable.** Every state-changing action writes to `audit_logs`.
5. **Provider-agnostic AI.** No business logic depends on a specific LLM vendor; all model calls pass through the Model Gateway.
6. **Async by default for AI/workflows.** Long-running work runs on the queue, never inline in a request thread.

---

## 1. Product Architecture (LOCKED)

### 1.1 Layered View

```
┌─────────────────────────────────────────────────────────────┐
│  CLIENTS:  Web App (Next.js) │ Admin Console │ Public REST API │ Webhooks │
└───────────────┬─────────────────────────────────────────────┘
                │ HTTPS / SSE
┌───────────────▼─────────────────────────────────────────────┐
│  API GATEWAY  — auth, tenant resolution, rate-limit, routing  │
├──────────────────────────────────────────────────────────────┤
│  APPLICATION SERVICES (modular monolith, NestJS)              │
│  Identity │ Tenancy │ Agents │ Workflows │ Knowledge │        │
│  Conversations │ Integrations │ Billing │ Admin/Observability │
├──────────────────────────────────────────────────────────────┤
│  AI ORCHESTRATION LAYER                                       │
│  Model Gateway │ Agent Runtime │ Tool Registry │ RAG Pipeline │ Memory │
├──────────────────────────────────────────────────────────────┤
│  ASYNC BACKBONE   Redis + BullMQ (queues, schedulers, workers)│
├──────────────────────────────────────────────────────────────┤
│  DATA LAYER   PostgreSQL 15+ (+pgvector) │ Redis cache │ S3 obj │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 Locked Technology Stack

| Concern | Choice (LOCKED) |
|---|---|
| Backend runtime | Node.js (LTS), **TypeScript** |
| Backend framework | **NestJS** (modular monolith; module = bounded context) |
| ORM / migrations | **Prisma** |
| Primary database | **PostgreSQL 15+** |
| Vector storage | **pgvector** (same Postgres instance) |
| Cache / queue / rate-limit | **Redis** + **BullMQ** |
| Object storage | **S3-compatible** (AWS S3 / MinIO) |
| Frontend | **Next.js (App Router)** + TypeScript + Tailwind |
| Auth | **JWT access + rotating refresh tokens**; API keys for M2M |
| AI access | **Provider-agnostic Model Gateway** (Anthropic / OpenAI / others) |
| Tooling protocol | **MCP** for external tool/connector integration |
| Realtime AI output | **Server-Sent Events (SSE)** |
| Observability | Structured logs + OpenTelemetry traces + metrics |

> Deployment shape is a **modular monolith**, not microservices. Modules are extractable later, but the boundary is the NestJS module — locked.

### 1.3 Locked Folder Structure (monorepo)

```
rabbani-aita-os/
├── apps/
│   ├── api/                      # NestJS backend (the kernel + modules live here)
│   │   └── src/
│   │       ├── core/             # cross-cutting: config, logging, errors, guards, RLS
│   │       ├── kernel/           # LOCKED bounded contexts
│   │       │   ├── identity/     # users, sessions, auth
│   │       │   ├── tenancy/      # tenants, membership, tenant context
│   │       │   ├── authz/        # roles, permissions, policy engine
│   │       │   ├── agents/       # agent defs, versions, runtime adapters
│   │       │   ├── workflows/    # definitions, engine, runs
│   │       │   ├── knowledge/    # KBs, documents, chunks, RAG
│   │       │   ├── conversations/# conversations, messages, streaming
│   │       │   ├── integrations/ # connectors, webhooks, API keys
│   │       │   ├── billing/      # plans, subscriptions, usage metering
│   │       │   └── observability/# audit, events, usage records
│   │       ├── modules/          # DOMAIN/VERTICAL modules extend kernel here
│   │       ├── ai/               # model gateway, tool registry, memory, guardrails
│   │       └── jobs/             # BullMQ processors/workers
│   ├── web/                      # Next.js end-user app
│   └── admin/                    # Next.js platform admin console
├── packages/
│   ├── shared-types/             # DTOs/contracts shared FE/BE
│   ├── sdk/                      # generated API client
│   └── config/                   # shared lint/ts/tailwind config
├── prisma/
│   ├── schema.prisma             # kernel schema (extend, never rewrite)
│   └── migrations/
└── infra/                        # IaC, docker, ci
```

> **Rule:** verticals are added only under `apps/api/src/modules/<name>/` and `apps/web/...`. The `kernel/` directory is frozen.

---

## 2. Database Architecture (LOCKED)

### 2.1 Locked Decisions

- **Engine:** PostgreSQL 15+, UUID (v7-style, time-ordered) primary keys.
- **Isolation model:** **Shared database, shared schema, `tenant_id` discriminator + Row-Level Security.** (Chosen for SaaS density and operational simplicity; schema-per-tenant explicitly rejected to keep migrations single-path.)
- **Every tenant-scoped table** has `tenant_id UUID NOT NULL` and an RLS policy `tenant_id = current_setting('app.tenant_id')`.
- **Standard columns** on every table: `id`, `created_at`, `updated_at`, `deleted_at` (soft delete via nullable timestamp).
- **Naming:** snake_case, plural tables, FK as `<entity>_id`.
- **High-volume tables** (`messages`, `audit_logs`, `usage_records`, `workflow_run_steps`) are **partitioned by month** (range on `created_at`).
- **Vectors** live in `document_chunks.embedding vector(N)` with an HNSW/IVFFlat index.

### 2.2 Kernel Schema (frozen entity set)

**Tenancy & Identity**

| Table | Key fields | Notes / relationships |
|---|---|---|
| `tenants` | id, name, slug, status, plan_id, settings(jsonb), data_region | root isolation entity |
| `users` | id, email(unique), password_hash, name, status, is_platform_staff | global identity (one user → many tenants) |
| `tenant_memberships` | id, tenant_id, user_id, status, invited_by | join: user↔tenant; **carries the tenant scope** |
| `sessions` | id, user_id, tenant_id, refresh_token_hash, expires_at, ip, ua | rotating refresh tokens |

**Authorization**

| Table | Key fields | Notes |
|---|---|---|
| `roles` | id, tenant_id(nullable=system), key, name, is_system | system roles have null tenant_id |
| `permissions` | id, key, description | global catalog (e.g. `agents.create`) |
| `role_permissions` | role_id, permission_id | M:N |
| `membership_roles` | membership_id, role_id | roles assigned per tenant membership |

**Agents & AI**

| Table | Key fields | Notes |
|---|---|---|
| `agents` | id, tenant_id, key, name, description, status, current_version_id | logical agent |
| `agent_versions` | id, agent_id, version, system_prompt, model_config(jsonb), tool_config(jsonb), guardrail_config(jsonb), knowledge_base_ids | immutable, versioned |
| `tools` | id, tenant_id(nullable), key, type, schema(jsonb), config(jsonb) | function/MCP/http tools |

**Knowledge / RAG**

| Table | Key fields | Notes |
|---|---|---|
| `knowledge_bases` | id, tenant_id, name, embedding_model | |
| `documents` | id, tenant_id, knowledge_base_id, source_uri, status, metadata(jsonb) | |
| `document_chunks` | id, tenant_id, document_id, content, embedding vector(N), metadata(jsonb) | partition candidate; vector index |

**Conversations**

| Table | Key fields | Notes |
|---|---|---|
| `conversations` | id, tenant_id, agent_id, user_id(nullable), channel, status, metadata(jsonb) | |
| `messages` | id, tenant_id, conversation_id, role, content(jsonb), token_usage(jsonb), parent_id | **partitioned monthly** |

**Workflows**

| Table | Key fields | Notes |
|---|---|---|
| `workflows` | id, tenant_id, key, name, status, current_version_id | |
| `workflow_versions` | id, workflow_id, version, definition(jsonb DAG), triggers(jsonb) | immutable |
| `workflow_runs` | id, tenant_id, workflow_id, version_id, status, trigger_type, input(jsonb), output(jsonb), started_at, finished_at | |
| `workflow_run_steps` | id, tenant_id, run_id, node_key, status, attempt, input/output(jsonb), error | **partitioned monthly** |

**Integrations & Platform**

| Table | Key fields | Notes |
|---|---|---|
| `integrations` | id, tenant_id, provider, status, credentials_ref | secrets stored in vault, not DB |
| `api_keys` | id, tenant_id, name, key_hash, scopes(jsonb), last_used_at | M2M auth |
| `webhooks` | id, tenant_id, url, event_types(jsonb), secret, status | outbound |
| `files` | id, tenant_id, storage_key, content_type, size, owner_id | S3 pointer |
| `notifications` | id, tenant_id, user_id, type, payload(jsonb), read_at | |

**Billing & Observability**

| Table | Key fields | Notes |
|---|---|---|
| `plans` | id, key, name, limits(jsonb), price(jsonb) | catalog |
| `subscriptions` | id, tenant_id, plan_id, status, period_start, period_end | |
| `usage_records` | id, tenant_id, metric, quantity, unit, occurred_at, ref(jsonb) | **partitioned monthly**; meters tokens/runs/storage |
| `audit_logs` | id, tenant_id, actor_id, actor_type, action, target_type, target_id, before(jsonb), after(jsonb), ip, created_at | **partitioned monthly**; append-only |
| `events` | id, tenant_id, type, payload(jsonb), processed_at | internal event bus / outbox |

> **Extension rule:** Domain modules add **new tables** (each with `tenant_id` + RLS + standard columns) under their own prefix (e.g. `edu_courses`). They never alter kernel tables; they reference them by FK only.

---

## 3. API Architecture (LOCKED)

### 3.1 Locked Conventions

- **Style:** RESTful, resource-oriented, JSON over HTTPS.
- **Versioning:** URL-prefixed — `/api/v1/...`. New versions are additive; v1 is frozen once shipped.
- **Tenant scope:** resolved from JWT claim `tenant_id` (also overridable by `X-Tenant-Id` for platform staff). All tenant routes are nested logically under the resolved tenant — never expose `tenant_id` in the path.
- **Auth header:** `Authorization: Bearer <jwt>` (users) or `Authorization: Bearer <api_key>` (M2M).
- **Pagination:** cursor-based — `?limit=&cursor=`; responses return `{ data, page: { next_cursor, has_more } }`.
- **Filtering/sorting:** `?filter[field]=&sort=-created_at`.
- **Errors:** RFC 7807-style — `{ type, title, status, detail, code, trace_id }`.
- **Idempotency:** mutations accept `Idempotency-Key` header.
- **Rate limiting:** per-tenant + per-key token buckets in Redis; returns `429` + `Retry-After`.
- **Streaming:** AI responses stream via **SSE** at `.../stream` endpoints.
- **Contracts:** OpenAPI 3 generated from NestJS; SDK generated from it.

### 3.2 Locked Resource Map (v1)

| Group | Base path | Core resources |
|---|---|---|
| Auth | `/api/v1/auth` | `login`, `refresh`, `logout`, `register`, `password` |
| Tenants | `/api/v1/tenants` | tenant CRUD, `settings`, `members`, `invitations` |
| AuthZ | `/api/v1/roles`, `/api/v1/permissions` | role CRUD, assignment |
| Agents | `/api/v1/agents` | agents, `:id/versions`, `:id/publish` |
| Conversations | `/api/v1/conversations` | conversations, `:id/messages`, `:id/messages/stream` |
| Workflows | `/api/v1/workflows` | workflows, `:id/versions`, `:id/runs`, `runs/:id` |
| Knowledge | `/api/v1/knowledge-bases` | KBs, `:id/documents`, `:id/search` |
| Tools | `/api/v1/tools` | tool registry |
| Integrations | `/api/v1/integrations`, `/api/v1/webhooks`, `/api/v1/api-keys` | connectors |
| Files | `/api/v1/files` | upload (presigned), metadata |
| Billing | `/api/v1/billing` | `subscription`, `usage`, `plans` |
| Admin | `/api/v1/admin/*` | platform-staff only |

> **Extension rule:** modules register **new route groups** under `/api/v1/<module>`. Existing groups/paths are never repurposed.

---

## 4. Multi-Tenant Architecture (LOCKED)

| Aspect | Locked design |
|---|---|
| Isolation tier | **Pooled** (shared DB/compute) with hard data isolation via RLS + quotas |
| Tenant resolution | JWT `tenant_id` claim → set Postgres session var `app.tenant_id` at request start (middleware) |
| Data isolation | **Postgres RLS** on every tenant table; app code also filters by `tenant_id` (defense in depth) |
| Compute isolation | Shared workers with **per-tenant queue concurrency limits + rate limits** (noisy-neighbor control) |
| Config isolation | `tenants.settings` jsonb + per-tenant feature flags |
| Secrets | Per-tenant integration credentials referenced from a secrets vault, never stored plaintext in DB |
| Tenant lifecycle | `provisioning → active → suspended → deleting → deleted`; suspension blocks at gateway; deletion is soft then purged on schedule |
| Data residency | `tenants.data_region` reserved; region-pinned storage supported without schema change |
| Cross-tenant access | Only `is_platform_staff` users, only via `/admin/*`, always audited |

> One human (`users` row) can belong to many tenants via `tenant_memberships`. The **active tenant** is bound per session/token — switching tenants issues a new scoped token.

---

## 5. User Roles (LOCKED)

RBAC = **roles → permissions**, assigned **per tenant membership**. Roles are sugar over the permission catalog; **new capabilities are added as permissions, not new hardcoded roles.**

### 5.1 Role Tiers

**Platform-level (system roles, `tenant_id = null`)**

| Role | Purpose |
|---|---|
| `SUPER_ADMIN` | Full platform control, all tenants |
| `PLATFORM_SUPPORT` | Read + scoped support actions across tenants (audited) |
| `PLATFORM_BILLING` | Plans, subscriptions, invoicing |

**Tenant-level (default seeded roles)**

| Role | Scope |
|---|---|
| `TENANT_OWNER` | Full control of one tenant incl. billing & deletion |
| `TENANT_ADMIN` | Manage members, roles, settings, all resources |
| `BUILDER` | Create/edit/publish agents, workflows, knowledge bases, tools |
| `OPERATOR` | Run workflows, manage conversations, no design rights |
| `MEMBER` | Use published agents, create conversations |
| `VIEWER` | Read-only |
| `SERVICE_ACCOUNT` | Non-human; scoped via API key permissions |

### 5.2 Permission Matrix (representative — full catalog lives in `permissions`)

| Permission key | OWNER | ADMIN | BUILDER | OPERATOR | MEMBER | VIEWER |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| `tenant.manage` | ✓ | ✓ | | | | |
| `members.manage` | ✓ | ✓ | | | | |
| `billing.manage` | ✓ | | | | | |
| `agents.create` | ✓ | ✓ | ✓ | | | |
| `agents.publish` | ✓ | ✓ | ✓ | | | |
| `workflows.create` | ✓ | ✓ | ✓ | | | |
| `workflows.run` | ✓ | ✓ | ✓ | ✓ | | |
| `knowledge.manage` | ✓ | ✓ | ✓ | | | |
| `conversations.use` | ✓ | ✓ | ✓ | ✓ | ✓ | |
| `*.read` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

> **Extension rule:** a new module ships its own permission keys (e.g. `edu.courses.manage`) and may **add** them to existing roles via seed migration. Role *names* are frozen.

---

## 6. Workflow Architecture (LOCKED)

### 6.1 Model

- A **Workflow** has immutable **versions**; each version holds a **DAG definition** (`definition` jsonb) and **triggers**.
- Execution = a **`workflow_run`** advancing through a **state machine**, with one **`workflow_run_step`** per node attempt.
- Execution is **async** on BullMQ; the engine is an orchestrator, workers execute nodes.

### 6.2 Locked Trigger Types

`manual` · `scheduled (cron)` · `event` (internal `events`) · `webhook` (inbound) · `api` (programmatic).

### 6.3 Locked Node Types (extensible registry)

| Node type | Function |
|---|---|
| `agent` | Invoke an agent version |
| `tool` | Call a registered tool / MCP action |
| `http` | Call external HTTP endpoint |
| `condition` | Branch on expression |
| `loop` | Iterate over collection |
| `human_approval` | Pause for human-in-the-loop decision |
| `transform` | Map/shape data |
| `subworkflow` | Invoke another workflow |
| `delay` | Wait until time/condition |

### 6.4 Run State Machine

```
queued → running → ( waiting_approval ⇄ running ) → completed
                                         ├→ failed (retries exhausted)
                                         └→ cancelled
```

- **Retries:** per-node policy (max attempts, backoff); each attempt = new `workflow_run_step` row.
- **Idempotency:** node executions keyed by `(run_id, node_key, attempt)`.
- **Durability:** state persisted after every node; engine resumes safely after restart.
- **Observability:** every step writes timing, IO, and errors; full run is replayable/inspectable.

> **Extension rule:** new node types register in the node registry; the engine, schema, and run model do not change.

---

## 7. AI Agent Architecture (LOCKED)

### 7.1 Agent Definition (an `agent_version`)

An agent is fully described by its version record:
`system_prompt` · `model_config` (provider, model, temperature, max tokens) · `tool_config` (allowed tools) · `knowledge_base_ids` (RAG sources) · `guardrail_config` (input/output policies) · `memory_config`.

### 7.2 Locked Sub-systems

| Sub-system | Responsibility |
|---|---|
| **Model Gateway** | Single provider-agnostic interface for all LLM calls; handles provider routing, retries, fallback, token accounting → `usage_records`. **No business code calls a vendor SDK directly.** |
| **Agent Runtime** | Executes the agent loop: assemble context → call model → handle tool calls → stream output. |
| **Tool Registry** | Declarative tools (function / HTTP / **MCP**) with JSON schemas; per-tenant + system scope; permission-gated. |
| **RAG Pipeline** | Ingest → chunk → embed → store (`document_chunks`) → retrieve (vector + metadata filter, tenant-scoped) → inject. |
| **Memory** | Short-term = conversation `messages`; long-term = vector recall scoped by tenant/agent. |
| **Guardrails** | Pre/post filters: PII, prompt-injection, content policy, max-cost ceilings. |
| **Orchestration** | Single-agent by default; **supervisor/multi-agent** pattern supported via the `agent` workflow node (an agent can call sub-agents as tools). |
| **Observability/Evals** | Every model call logged with tokens, latency, cost, and trace id; eval hooks for quality scoring. |

### 7.3 Agent Execution Flow (conceptual)

```
request → resolve tenant + agent_version
        → guardrails (input)
        → RAG retrieve (tenant-scoped)
        → Model Gateway call ──┬─ tool_call? → Tool Registry → loop
                               └─ final → guardrails (output) → stream (SSE)
        → persist message + usage_records + audit_log
```

> **Extension rule:** new model providers plug into the Model Gateway; new capabilities ship as **tools** or **workflow nodes**. The agent definition shape and runtime contract are frozen.

---

## 8. Cross-Cutting Concerns (LOCKED)

| Concern | Decision |
|---|---|
| Security | TLS everywhere; secrets in vault; passwords Argon2; JWT short-lived + refresh rotation; RLS; least-privilege permissions |
| Audit | Append-only `audit_logs` on every mutation; immutable |
| Observability | Structured JSON logs, OpenTelemetry traces, Prometheus-style metrics; `trace_id` propagated to clients |
| Async backbone | Redis + BullMQ; dead-letter queues; scheduled jobs |
| Config | 12-factor env config; per-tenant feature flags in `tenants.settings` |
| Data lifecycle | Soft delete (`deleted_at`) → scheduled hard purge; tenant export/delete honored |
| Migrations | Single forward-only Prisma migration path; additive only |

---

## 9. What "LOCKED" Forbids (guardrails for future prompts)

Future increments **must not**:
- ❌ change the isolation model, `tenant_id` discriminator, or RLS approach
- ❌ rename, drop, or repurpose any kernel table or column
- ❌ change PK strategy, soft-delete, or partitioning conventions
- ❌ change the API versioning scheme, auth scheme, or error format
- ❌ restructure the monorepo or `kernel/` directory
- ❌ add hardcoded roles in place of permissions
- ❌ call an LLM vendor SDK outside the Model Gateway
- ❌ run AI/workflow work synchronously inside a request thread

Future increments **may** (this is how we grow):
- ✅ add Domain Modules under `modules/` with their own tables/endpoints/permissions
- ✅ add new workflow node types, tools, and model providers via their registries
- ✅ add new permissions and attach them to existing roles via seed migrations
- ✅ add new `/api/v1/<module>` route groups

---

## 10. Open Inputs (refine the kernel without redesigning it)

To tighten v1 you can later confirm — none of these alter locked decisions, they only parameterize them:
1. The **actual vertical/domain** (defines the first Domain Module).
2. Primary **LLM provider(s)** + embedding dimension `N` for `vector(N)`.
3. Expected **scale** (tenants, msgs/day) — tunes partitioning/indexes only.
4. **Compliance/residency** needs (sets `data_region` policy).

---

## 11. Amendments Log

| Version | Date | Change | Author |
|---|---|---|---|
| v1.0 | — | Initial locked architecture | Lead Enterprise Architect |

> *(All future architecture changes are appended here as v1.1, v1.2 … — never by editing sections above.)*

---

**STATUS: AWAITING APPROVAL TO LOCK.**
On your approval, this becomes the binding source of truth and all future prompts extend — never redesign — it. No code has been generated.
