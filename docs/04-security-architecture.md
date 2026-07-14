# 04 — Security, Identity & Resilience Architecture

Covers: Authentication · Permission Matrix · Security Architecture · Audit Log · Backup Strategy · Disaster Recovery

---

## 7. Authentication Architecture

Two entry classes: **citizens** (mobile, OneID-verified, then lightweight daily PIN/biometric)
and **officials/partners** (web, OneID or corporate credentials + mandatory MFA).

```mermaid
flowchart TB
    subgraph ACTORS["Entry Points"]
        direction LR
        MU["📱 User / Family<br/>Mobile App"] ~~~ AU["🖥️ Officials & Partners<br/>Web Panel"]
    end

    subgraph IAM["IDENTITY PLATFORM"]
        KC["Keycloak<br/>OIDC / OAuth 2.1 Authorization Server"]
        ONE["OneID Broker<br/>national e-ID federation"]
        MFA["MFA Service<br/>TOTP · SMS OTP"]
        POL["Session Policy Engine<br/>role-based token lifetimes"]
    end

    subgraph TOKENS["Token Model"]
        direction LR
        AT["Access Token — JWT, 15 min<br/>claims: roles, district_id, caseload"] ~~~ RT["Refresh Token — rotating<br/>mobile 30d · web 8h"] ~~~ DK["Device Binding<br/>attested device key (mobile)"]
    end

    MU -->|"first login: OneID"| KC
    MU -->|"daily: PIN / biometric + device key"| KC
    AU -->|"OneID / password"| KC
    KC <--> ONE
    AU --> MFA
    KC --> POL --> AT & RT
    MU --> DK

    classDef a fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef i fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef t fill:#e6f4ea,stroke:#188038,color:#0d652d
    class MU,AU a
    class KC,ONE,MFA,POL i
    class AT,RT,DK t
```

### Login sequence (user first-time)

```mermaid
sequenceDiagram
    autonumber
    actor U as Probation User
    participant APP as Mobile App
    participant KC as Keycloak
    participant ONE as OneID (gov e-ID)
    participant IDS as identity-service

    U->>APP: Open app (activation code from officer)
    APP->>KC: Authorization Code + PKCE
    KC->>ONE: Federated authentication
    ONE-->>U: e-ID verification (PINFL)
    ONE-->>KC: Verified identity claims
    KC->>IDS: Match PINFL to pre-registered case
    IDS-->>KC: Roles + district scope
    KC-->>APP: ID / Access / Refresh tokens
    APP->>APP: Enroll device key + PIN/biometric
    Note over APP,KC: Subsequent logins: PIN/biometric<br/>+ device-bound refresh, OneID re-verify every 90 days
```

---

## 8. Permission Matrix

Legend: **F** full · **M** manage (create/update within scope) · **R** read · **O** own data only · **–** none.
Scope column defines the jurisdiction boundary enforced by RLS.

| Module | Super Admin | Regional Prosecutor | District Prosecutor | Probation Officer | Psychologist | Mentor | Employer | Training Center | User | Family |
|---|---|---|---|---|---|---|---|---|---|---|
| **Scope** | National | Region | District | Caseload | Assigned users | Assigned users | Own org | Own org | Self | Linked user |
| User management | F | R | M | M | – | – | – | – | O | – |
| Case & plans | R | R | M | M | R | R | – | – | O | – |
| Tasks | R | R | M | M | M | R | – | – | O | – |
| Risk indicators | R | R | R | M | M | – | – | – | – | – |
| Goals & habits | – | – | R | R | R | R | – | – | F | R* |
| Mood tracker | – | – | – | R° | F | – | – | – | F | – |
| Psych. sessions | – | – | R° | R° | F | – | – | – | O | – |
| Courses | M | R | R | R | – | R | – | M | O | – |
| Digital library | M | R | R | R | R | R | – | – | F | – |
| Job center | M | R | R | M | – | R | M | – | F | – |
| CV builder | – | – | – | R | – | R | R† | – | F | – |
| Chat | M‡ | – | – | M | M | M | – | – | F | – |
| AI assistant | M | – | – | – | – | – | – | – | F | – |
| Calendar | R | R | M | M | M | M | – | R | O | – |
| Notifications | M | M | M | M | M | M | – | – | O | O |
| Analytics | F | R (region) | R (district) | R (caseload) | R (assigned) | – | – | R (own) | O | R* |
| Reports | F | M (region) | M (district) | M (caseload) | M (assigned) | – | – | – | – | – |
| Audit logs | F | R (region) | R (district) | – | – | – | – | – | – | – |
| System settings | F | – | – | – | – | – | – | – | – | – |

`R*` progress summary only, with user consent · `R°` attendance & risk level only, never clinical content ·
`R†` only CVs shared to their vacancy · `M‡` moderation policy, not message content by default.

```mermaid
flowchart LR
    subgraph SCOPES["Jurisdiction Scopes (RLS)"]
        N["National"] --> RG["Region"] --> D["District"] --> C["Caseload"] --> S["Self"]
    end
    SA["Super Admin"] --- N
    RP["Regional Prosecutor"] --- RG
    DP["District Prosecutor"] --- D
    PO["Officer · Psychologist · Mentor"] --- C
    US["User · Family"] --- S

    classDef s fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef r fill:#e6f4ea,stroke:#188038,color:#0d652d
    class N,RG,D,C,S s
    class SA,RP,DP,PO,US r
```

---

## 9. Security Architecture (Defense in Depth)

```mermaid
flowchart TB
    subgraph L1["1 · PERIMETER"]
        direction LR
        WAF["WAF · Anti-DDoS · Geo-fencing (UZ)"] ~~~ TLS["TLS 1.3 everywhere · HSTS · cert pinning (mobile)"]
    end
    subgraph L2["2 · IDENTITY"]
        direction LR
        OIDC["OIDC + MFA + device binding"] ~~~ PAM["Privileged access management for admins"]
    end
    subgraph L3["3 · APPLICATION"]
        direction LR
        RBAC["RBAC + ABAC guards on every endpoint"] ~~~ VAL["Input validation · OWASP ASVS L2"] ~~~ SAST["SAST / DAST / dependency scanning in CI"]
    end
    subgraph L4["4 · SERVICE MESH"]
        direction LR
        MTLS["mTLS between services"] ~~~ NP["Network policies · zero-trust east-west"]
    end
    subgraph L5["5 · DATA"]
        direction LR
        ENC["AES-256 at rest · column crypto for PII"] ~~~ RLS["Row-level security by district"] ~~~ DLP["Pseudonymization for analytics"] ~~~ VLT["Vault-managed keys · rotation 90d"]
    end
    subgraph L6["6 · DETECTION & RESPONSE"]
        direction LR
        SIEM["SIEM correlation · anomaly alerts"] ~~~ AUD["Immutable audit trail"] ~~~ IR["Incident response runbooks · 24/7 on-call"]
    end

    L1 --> L2 --> L3 --> L4 --> L5 --> L6

    classDef l fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    class WAF,TLS,OIDC,PAM,RBAC,VAL,SAST,MTLS,NP,ENC,RLS,DLP,VLT,SIEM,AUD,IR l
```

**Compliance anchors:** Law of RUz "On Personal Data" (ZRU-547) — in-country storage;
O'zDSt cryptography alignment where mandated; OWASP ASVS L2; ISO 27001-aligned ISMS for operations.

---

## 26. Audit Log Architecture

Every security-relevant action — including **reads of personal data by officials** — produces an
immutable audit event. Officials know their access is logged; users can request their access history.

```mermaid
flowchart LR
    subgraph SRC["Event Producers"]
        SVC["All microservices<br/>(audit interceptor)"]
        GW["API Gateway<br/>access logs"]
        KCA["Keycloak<br/>auth events"]
        DBA["Database<br/>privileged query log"]
    end

    TOPIC[("Kafka: audit.events<br/>schema-validated")]

    subgraph PIPE["Audit Pipeline"]
        ENR["Enricher<br/>actor · role · district · request-id"]
        SIGN["Hash-chain signer<br/>tamper evidence (Merkle)"]
    end

    subgraph STORES["Storage & Analysis"]
        WORM[("WORM Store<br/>append-only · 7 years")]
        SIEM["SIEM<br/>rules & anomaly detection"]
        AUI["Audit UI<br/>Super Admin · Prosecutors (scoped)"]
    end

    ALERT["⚠️ Alerts<br/>unusual data access · off-hours export<br/>cross-district attempts"]

    SRC --> TOPIC --> ENR --> SIGN --> WORM
    SIGN --> SIEM --> ALERT
    WORM --> AUI

    classDef s fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef p fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef st fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef al fill:#fce8e6,stroke:#d93025,color:#a50e0e
    class SVC,GW,KCA,DBA s
    class TOPIC,ENR,SIGN p
    class WORM,SIEM,AUI st
    class ALERT al
```

**Audited event classes:** authentication, role grants, case reads/writes, report exports,
chat moderation actions, AI escalations, configuration changes, backup/restore operations.

---

## 27. Backup Strategy

**3-2-1 rule:** 3 copies · 2 storage types · 1 off-site (DR region), with immutable tier for audit data.

```mermaid
flowchart LR
    subgraph PROD["Production Data"]
        PG[("PostgreSQL")]
        MIN[("MinIO media")]
        CFG[("Cluster config<br/>+ Vault")]
    end

    subgraph LOCAL["Tier 1 — Local (Primary DC)"]
        WAL["WAL archive<br/>continuous, RPO ≈ 0-5 min"]
        SNAP["Nightly snapshots<br/>02:00 Tashkent"]
    end

    subgraph OFFSITE["Tier 2 — Off-site (DR Region)"]
        REPL["Streaming replica<br/>+ bucket replication"]
        VAULTB[("Backup Vault<br/>immutable · versioned")]
    end

    TEST["🔁 Monthly restore drill<br/>automated verify + timing report"]

    PG --> WAL --> VAULTB
    PG --> SNAP --> VAULTB
    MIN --> REPL --> VAULTB
    CFG --> SNAP
    PG --> REPL
    VAULTB --> TEST

    classDef p fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef l fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef o fill:#fef7e0,stroke:#f9ab00,color:#a56500
    class PG,MIN,CFG p
    class WAL,SNAP l
    class REPL,VAULTB,TEST o
```

| Dataset | Method | Frequency | Retention |
|---|---|---|---|
| PostgreSQL | WAL archiving + base backup | Continuous / nightly | 35 days PITR |
| PostgreSQL | Logical dump (per district) | Weekly | 12 months |
| MinIO media | Bucket replication + versioning | Continuous | 6 months versions |
| Audit WORM | Immutable object lock | Continuous | 7 years |
| K8s config / Vault | GitOps repo + sealed snapshots | On change / daily | 12 months |

---

## 28. Disaster Recovery Plan

```mermaid
flowchart TB
    subgraph NORMAL["Normal Operation"]
        P["Primary DC — Tashkent<br/>active"]
        D["DR Site — second region<br/>warm standby, data current"]
        P -->|"replication (async, lag < 60s)"| D
    end

    INC["🔥 Incident detected<br/>health checks + SIEM + operator"]
    TRIAGE{"Severity?"}
    Z["Zone failure<br/>→ K8s reschedules, PG auto-failover<br/>RTO ≈ 5 min · no data loss"]
    RSTR["Data corruption / ransomware<br/>→ PITR restore from immutable vault<br/>RTO ≤ 8h · RPO ≤ 24h"]
    FAIL["Full DC loss<br/>→ DR activation"]

    subgraph DRPROC["DR Activation Procedure"]
        S1["1 · Declare DR — District Prosecutor + Ops lead"]
        S2["2 · Promote DR PostgreSQL replica"]
        S3["3 · Scale standby cluster · restore secrets"]
        S4["4 · Repoint DNS / national edge"]
        S5["5 · Verify: auth, case data, chat, media"]
        S6["6 · Notify roles via SMS status channel"]
        S1 --> S2 --> S3 --> S4 --> S5 --> S6
    end

    INC --> TRIAGE
    TRIAGE --> Z
    TRIAGE --> RSTR
    TRIAGE --> FAIL --> DRPROC

    classDef n fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef i fill:#fce8e6,stroke:#d93025,color:#a50e0e
    classDef s fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    class P,D n
    class INC,FAIL,RSTR i
    class TRIAGE,Z,S1,S2,S3,S4,S5,S6 s
```

| Objective | Pilot target | National target |
|---|---|---|
| RTO (full DC loss) | ≤ 4 hours | ≤ 30 minutes (hot standby) |
| RPO (full DC loss) | ≤ 15 minutes | ≤ 1 minute (sync options) |
| DR drill cadence | Quarterly tabletop + semi-annual failover | Quarterly full failover |
| Degraded mode | Mobile app offline-first: tasks, library, habits keep working; sync on recovery | Same + regional edge cache |
