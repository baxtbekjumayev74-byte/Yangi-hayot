# 02 — System & Infrastructure Architecture

Covers: System Architecture · Backend Microservices · Cloud Infrastructure · Data Flow · API Architecture · Folder Structure

---

## 4. System Architecture (Layered View)

```mermaid
flowchart TB
    subgraph L1["CLIENT LAYER"]
        direction LR
        MA["📱 Flutter Mobile App<br/>User + Family"] ~~~ WA["🖥️ Next.js Admin Panel<br/>All professional roles"] ~~~ PP["🤝 Partner Web Portal<br/>Employer · Training Center"]
    end

    subgraph L2["EDGE & DELIVERY LAYER"]
        direction LR
        CDN["CDN + Media Edge<br/>books · audio · video"] ~~~ WAF["WAF + DDoS Protection"] ~~~ LB["Load Balancer<br/>TLS 1.3 termination"]
    end

    subgraph L3["API LAYER"]
        direction LR
        GW["API Gateway (Kong)<br/>AuthN · rate limits · versioning"] ~~~ WS["WebSocket Gateway<br/>chat · presence · live updates"] ~~~ BFF["BFF Aggregators<br/>mobile-bff · admin-bff"]
    end

    subgraph L4["SERVICE LAYER — Domain Microservices"]
        direction LR
        S1["Supervision Domain<br/>identity · case · risk"] ~~~ S2["Rehabilitation Domain<br/>employment · education · library<br/>mental health · goals"] ~~~ S3["Engagement Domain<br/>chat · AI · calendar · notifications"] ~~~ S4["Insight Domain<br/>analytics · reporting · audit"]
    end

    subgraph L5["DATA LAYER"]
        direction LR
        PG[("PostgreSQL 16<br/>OLTP + RLS")] ~~~ RD[("Redis")] ~~~ OS[("OpenSearch")] ~~~ S3O[("MinIO S3")] ~~~ KF[("Kafka")] ~~~ VDB[("pgvector")]
    end

    subgraph L6["EXTERNAL INTEGRATIONS"]
        direction LR
        ONEID["OneID e-ID"] ~~~ FCM["FCM / APNs Push"] ~~~ SMS["SMS Gateway (Eskiz)"] ~~~ LLM["LLM Provider<br/>(in-country proxy)"]
    end

    L1 --> L2 --> L3 --> L4 --> L5
    L4 <--> L6

    classDef c1 fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef c2 fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    classDef c3 fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef c4 fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef c5 fill:#fef7e0,stroke:#f9ab00,color:#a56500
    class MA,WA,PP c1
    class CDN,WAF,LB c2
    class GW,WS,BFF c3
    class S1,S2,S3,S4 c4
    class PG,RD,OS,S3O,KF,VDB c2
    class ONEID,FCM,SMS,LLM c5
```

**Pilot-to-national scaling note.** At 50 users, services run as modest replicas on a small
Kubernetes cluster; the identical topology scales horizontally per layer — no re-architecture
between pilot and national rollout. District ID is a first-class tenancy key in every service.

---

## 5. Backend Microservice Architecture

```mermaid
flowchart TB
    GW["API Gateway"]

    subgraph DOM1["IDENTITY & SUPERVISION DOMAIN"]
        IDN["identity-service<br/>users · roles · OneID · sessions"]
        CASE["case-service<br/>probation cases · plans · tasks"]
        RISKS["risk-service<br/>risk scoring · flags · reviews"]
    end

    subgraph DOM2["REHABILITATION DOMAIN"]
        EMPS["employment-service<br/>jobs · applications · CV builder"]
        EDUS["education-service<br/>courses · enrollments · certificates"]
        LIBS["library-service<br/>books · audiobooks · videos · progress"]
        MHS["mentalhealth-service<br/>moods · assessments · sessions"]
        GOALS["goal-service<br/>goals · habits · streaks · achievements"]
    end

    subgraph DOM3["ENGAGEMENT DOMAIN"]
        CHATS["chat-service<br/>conversations · moderation"]
        AIS["ai-service<br/>assistant · RAG · guardrails"]
        CALS["calendar-service<br/>meetings · schedules · reminders"]
        NOTS["notification-service<br/>push · SMS · in-app routing"]
    end

    subgraph DOM4["INSIGHT DOMAIN"]
        ANLS["analytics-service<br/>KPIs · dashboards · heat maps"]
        RPTS["reporting-service<br/>scheduled PDF/Excel reports"]
        AUDS["audit-service<br/>immutable event trail"]
    end

    KAFKA[("Kafka Event Backbone<br/>user.events · case.events · progress.events<br/>chat.events · audit.events")]

    GW --> DOM1 & DOM2 & DOM3
    DOM1 & DOM2 & DOM3 -->|"publish domain events"| KAFKA
    KAFKA -->|"consume"| DOM4
    KAFKA -->|"notification triggers"| NOTS

    classDef dom1 fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef dom2 fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef dom3 fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef dom4 fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    classDef bus fill:#fef7e0,stroke:#f9ab00,color:#a56500
    class IDN,CASE,RISKS dom1
    class EMPS,EDUS,LIBS,MHS,GOALS dom2
    class CHATS,AIS,CALS,NOTS dom3
    class ANLS,RPTS,AUDS dom4
    class KAFKA bus
```

**Service design rules**

| Rule | Detail |
|---|---|
| Database-per-service | Each service owns its PostgreSQL schema; cross-service reads via API or events only |
| Sync + async | REST/gRPC for commands & queries; Kafka events for facts (\"user completed course\") |
| Tenancy | Every table carries `district_id`; RLS enforces jurisdiction at the database layer |
| Idempotency | All event consumers idempotent; outbox pattern for reliable publishing |
| Sensitive isolation | `mentalhealth-service` data is encrypted with a separate key and never flows to analytics in raw form |

---

## 15. Cloud Infrastructure

```mermaid
flowchart TB
    subgraph DC1["PRIMARY — Government Cloud, Tashkent (Zone A + Zone B)"]
        subgraph K8S["Kubernetes Cluster (HA control plane)"]
            direction LR
            NPA["Node Pool: apps<br/>microservices · BFFs<br/>3–12 nodes, autoscale"] ~~~ NPD["Node Pool: data<br/>Kafka · OpenSearch<br/>dedicated, local SSD"] ~~~ NPAI["Node Pool: ai<br/>inference / RAG workers"]
        end
        PGHA[("PostgreSQL HA<br/>Patroni: 1 primary + 2 replicas<br/>across zones")]
        REDIS[("Redis Sentinel")]
        MINIO[("MinIO — 4-node<br/>erasure coding")]
        MON["Observability Stack<br/>Prometheus · Grafana · Loki · Alertmanager"]
        VAULT["HashiCorp Vault<br/>secrets · encryption keys"]
    end

    subgraph DC2["DISASTER RECOVERY — Second Region Site"]
        K8S2["Standby K8s Cluster<br/>(pilot: warm minimal / national: hot)"]
        PGR[("PostgreSQL<br/>streaming replica")]
        MINIO2[("MinIO replica<br/>async bucket replication")]
        BK[("Backup Vault<br/>WAL archive + snapshots<br/>immutable, 7-year audit tier")]
    end

    EDGE["National Edge<br/>WAF · CDN · Anti-DDoS"]
    UZNET["UZ-IX / Government Network"]

    UZNET --> EDGE --> K8S
    K8S --> PGHA & REDIS & MINIO
    K8S --> VAULT
    MON -.-> K8S
    PGHA -->|"async streaming replication"| PGR
    MINIO -->|"bucket replication"| MINIO2
    PGHA & MINIO -->|"scheduled backups"| BK
    K8S2 -.->|"activated on failover"| PGR

    classDef prim fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef data fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef dr fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef net fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class NPA,NPD,NPAI,MON,VAULT,K8S2 prim
    class PGHA,REDIS,MINIO data
    class PGR,MINIO2,BK dr
    class EDGE,UZNET net
```

**Sizing**

| Phase | Users | App nodes | DB | Notes |
|---|---|---|---|---|
| Pilot (Sherobod) | 50 | 3 × 8 vCPU | 1×primary + 1×replica | Single cluster, warm DR |
| Region | ~1,500 | 6–8 nodes | +1 replica, PgBouncer | Read replicas for analytics |
| National | ~50,000+ | 12+ nodes, multi-cluster | Citus/partitioning by region | Hot DR, regional edge caches |

---

## 16. Data Flow Diagram

```mermaid
flowchart LR
    subgraph SRC["DATA SOURCES"]
        U1["User actions<br/>moods · habits · reading · lessons"]
        P1["Professional inputs<br/>assessments · case notes · meetings"]
        X1["Partner inputs<br/>vacancies · courses · hiring decisions"]
        S1["System events<br/>logins · notifications · AI usage"]
    end

    subgraph ING["INGESTION & VALIDATION"]
        API["API Layer<br/>schema validation · authZ"]
        OUT["Transactional Outbox"]
    end

    subgraph PROC["PROCESSING"]
        KAF[("Kafka Topics")]
        STR["Stream Processors<br/>progress scoring · streaks<br/>risk indicators · anonymization"]
    end

    subgraph STORE["STORAGE"]
        OLTP[("Operational DBs<br/>PostgreSQL per service")]
        DWH[("Analytics Store<br/>star schema, pseudonymized")]
        AUD[("Audit Store<br/>append-only, WORM")]
        OBJ[("Object Store<br/>media · reports")]
    end

    subgraph CONS["CONSUMPTION"]
        APPD["User app<br/>own data only"]
        DASH["Role dashboards<br/>jurisdiction-scoped"]
        REP["Official reports<br/>PDF / Excel"]
        AIH["AI assistant<br/>consented context"]
    end

    SRC --> API --> OUT --> KAF --> STR
    API --> OLTP
    STR --> DWH & AUD
    OLTP --> APPD
    DWH --> DASH & REP
    OLTP --> AIH
    OBJ --> APPD & REP

    classDef src fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef ing fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef proc fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef sto fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef con fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class U1,P1,X1,S1 src
    class API,OUT ing
    class KAF,STR proc
    class OLTP,DWH,AUD,OBJ sto
    class APPD,DASH,REP,AIH con
```

**Data classification**

| Class | Examples | Handling |
|---|---|---|
| C1 Public | Course catalog, library titles | Standard controls |
| C2 Internal | Tasks, goals, achievements | Encrypted at rest, role-scoped |
| C3 Personal | Identity, employment, case data | RLS + field encryption, audit on read |
| C4 Sensitive | Mental-health records, chat content | Separate keys, need-to-know, never in analytics raw |

---

## 29. API Architecture

```mermaid
flowchart TB
    subgraph CLIENTS["Consumers"]
        direction LR
        M["Mobile App"] ~~~ W["Admin Panel"] ~~~ P["Partner Portal"]
    end

    subgraph GATE["API GATEWAY — api.yangihayot.uz"]
        direction LR
        AUTH["OIDC token validation<br/>JWT + JWKS"] ~~~ RATE["Rate limiting<br/>per-role quotas"] ~~~ VER["Versioning<br/>/api/v1 · /api/v2"] ~~~ AUD2["Access logging → audit"]
    end

    subgraph STYLES["API Styles"]
        direction LR
        REST["REST + OpenAPI 3.1<br/>domain CRUD & queries"] ~~~ GRPC["gRPC<br/>service-to-service"] ~~~ WSS["WebSocket<br/>chat · presence · live dashboards"] ~~~ WHK["Webhooks<br/>partner callbacks (signed)"]
    end

    subgraph CATALOG["API Catalog (v1)"]
        direction LR
        A1["Identity<br/>/auth · /users · /roles"] ~~~ A2["Supervision<br/>/cases · /tasks · /risk"] ~~~ A3["Rehabilitation<br/>/jobs · /cv · /courses · /library<br/>/moods · /goals · /habits"] ~~~ A4["Engagement & Insight<br/>/chats · /calendar · /notifications<br/>/ai/assistant · /analytics · /reports"]
    end

    CLIENTS --> GATE --> STYLES --> CATALOG

    classDef c fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef g fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef s fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef a fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class M,W,P c
    class AUTH,RATE,VER,AUD2 g
    class REST,GRPC,WSS,WHK s
    class A1,A2,A3,A4 a
```

**API standards**

- OpenAPI 3.1 contract-first; generated typed clients for Flutter and TypeScript
- Cursor pagination, RFC 9457 problem-details errors, `Accept-Language: uz | uz-Cyrl | ru`
- Response envelope carries `district_id` scoping context; server-side enforcement always wins
- Deprecation policy: N-1 version supported 12 months after N release

---

## 30. Folder Structure (Monorepo)

```text
yangi-hayot/
├── apps/
│   ├── mobile/                     # Flutter — user & family app
│   │   ├── lib/
│   │   │   ├── core/               # theme, i18n (uz, uz-Cyrl, ru), router, DI
│   │   │   ├── features/
│   │   │   │   ├── auth/           ├── dashboard/       ├── motivation/
│   │   │   │   ├── tasks/          ├── goals/           ├── habits/
│   │   │   │   ├── mood/           ├── courses/         ├── library/
│   │   │   │   ├── audiobooks/     ├── videos/          ├── jobs/
│   │   │   │   ├── cv_builder/     ├── chat/            ├── ai_assistant/
│   │   │   │   ├── calendar/       ├── achievements/    ├── notifications/
│   │   │   │   └── profile/
│   │   │   └── shared/             # widgets, offline sync, analytics client
│   │   └── test/
│   ├── admin-web/                  # Next.js — professional roles
│   │   └── src/
│   │       ├── app/(dashboard)/    # analytics, users, cases, risk, reports
│   │       ├── app/(management)/   # meetings, tasks, notifications, logs
│   │       └── modules/            # charts, heatmaps, permission-aware UI kit
│   └── partner-web/                # Next.js — employers & training centers
├── services/
│   ├── identity-service/           # each service: src/{api,domain,infra,events}
│   ├── case-service/
│   ├── risk-service/
│   ├── employment-service/
│   ├── education-service/
│   ├── library-service/
│   ├── mentalhealth-service/
│   ├── goal-service/
│   ├── chat-service/
│   ├── ai-service/
│   ├── calendar-service/
│   ├── notification-service/
│   ├── analytics-service/
│   ├── reporting-service/
│   └── audit-service/
├── packages/                       # shared libraries
│   ├── contracts/                  # OpenAPI + protobuf + event schemas
│   ├── auth-lib/                   # JWT validation, RBAC guards, RLS helpers
│   ├── kafka-lib/                  # outbox, idempotent consumers
│   └── ui-kit/                     # design system (web)
├── infra/
│   ├── terraform/                  # cloud resources per environment
│   ├── k8s/                        # Helm charts, Kustomize overlays (dev/stage/prod)
│   ├── observability/              # dashboards, alerts, SLOs
│   └── security/                   # policies, vault config, network rules
├── docs/                           # this architecture set + ADRs
│   └── adr/
└── tools/                          # codegen, seeding, load tests
```
