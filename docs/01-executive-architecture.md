# 01 — Executive Architecture

Audience: District Prosecutor, Regional Prosecutor, Prosecutor General's Office, project sponsors.

---

## 1. Executive Architecture Overview

One page, one story: **citizens engage through human-centered applications; professionals
supervise through governed workspaces; the state receives evidence, not surveillance.**

```mermaid
flowchart TB
    subgraph CH["EXPERIENCE CHANNELS"]
        direction LR
        MOB["📱 User Mobile App<br/>iOS · Android · Offline-first"]
        WEB["🖥️ Professional Web Panel<br/>Prosecutors · Officers · Specialists"]
        FAM["👪 Family Portal<br/>Limited, consent-based"]
        PRT["🤝 Partner Portal<br/>Employers · Training Centers"]
    end

    subgraph CAP["PLATFORM CAPABILITIES"]
        direction LR
        subgraph REHAB["Rehabilitation Services"]
            EMP["Employment<br/>& CV Builder"]
            EDU["Education<br/>& Courses"]
            LIB["Digital Library<br/>& Audiobooks"]
            MH["Mental Health<br/>& Mood"]
            GOAL["Goals · Habits<br/>· Achievements"]
        end
        subgraph ENG["Engagement Services"]
            CHAT["Secure Chat<br/>& Mentoring"]
            AI["AI Assistant<br/>(guarded)"]
            CAL["Calendar<br/>& Meetings"]
            NOTIF["Notifications"]
        end
        subgraph SUP["Supervision Services"]
            CASE["Case & Task<br/>Management"]
            RISK["Risk & Progress<br/>Indicators"]
            RPT["Reporting<br/>& Analytics"]
        end
    end

    subgraph FND["TRUSTED FOUNDATION"]
        direction LR
        IDP["Identity & Access<br/>OneID · Keycloak · MFA"]
        SEC["Security & Audit<br/>Immutable logs · Encryption"]
        DATA["Sovereign Data Platform<br/>PostgreSQL · Kafka · S3 (in-country)"]
        INFRA["Cloud Infrastructure<br/>Kubernetes · Tashkent + DR"]
    end

    subgraph GOV["GOVERNMENT INTEGRATION"]
        ONEID["OneID<br/>National e-ID"]
        SMSGW["SMS Gateway<br/>Eskiz / Playmobile"]
        EGOV["E-Gov Registries<br/>(roadmap)"]
    end

    CH --> CAP --> FND
    FND <--> GOV

    classDef blue fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef green fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef gray fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    classDef navy fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    class MOB,WEB,FAM,PRT blue
    class EMP,EDU,LIB,MH,GOAL,CHAT,AI,CAL,NOTIF green
    class CASE,RISK,RPT navy
    class IDP,SEC,DATA,INFRA,ONEID,SMSGW,EGOV gray
```

**Key executive messages**

1. Every capability serves one KPI: **successful reintegration rate**.
2. Supervision consumes *outcomes* (progress, risk, attendance) — not private content.
3. The foundation is sovereign, auditable, and reusable nationally without redesign.

---

## 2. Government Organization Flow

Jurisdictional hierarchy and how oversight information flows upward while mandates flow downward.
The pilot node (Sherobod) is highlighted.

```mermaid
flowchart TB
    PGO["🏛️ Prosecutor General's Office<br/>Republic of Uzbekistan<br/><i>National policy · Phase 4</i>"]
    RPO["🏛️ Surxondaryo Regional<br/>Prosecutor's Office<br/><i>Regional oversight · Phase 2</i>"]
    DPO["⭐ Sherobod District<br/>Prosecutor's Office<br/><b>PILOT AUTHORITY · Phase 1</b>"]
    ODPO["Other District Offices<br/>Surxondaryo (13)<br/><i>Phase 2 rollout</i>"]

    subgraph FIELD["District Operations — Sherobod"]
        PO["👮 Probation Officers<br/>caseload ≤ 25 users each"]
        PSY["🧠 Psychologists"]
        MEN["🤝 Mentors<br/>vetted community volunteers"]
    end

    subgraph ECO["Local Ecosystem Partners"]
        EMPL["🏭 Employers"]
        TC["🎓 Training Centers"]
    end

    USERS["👤 ~50 Probation Users<br/>+ 👪 Family Members (limited)"]

    PGO -->|"national mandate · methodology"| RPO
    RPO -->|"supervision · resource allocation"| DPO
    RPO -.->|"Phase 2"| ODPO
    DPO -->|"case assignment · approvals"| FIELD
    DPO -->|"partnership agreements"| ECO
    FIELD -->|"guidance · support"| USERS
    ECO -->|"jobs · skills"| USERS

    USERS -->|"progress · engagement data"| FIELD
    FIELD -->|"case reports · risk flags"| DPO
    DPO -->|"district KPI reports"| RPO
    RPO -->|"aggregated regional analytics"| PGO

    classDef gov fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef pilot fill:#1a73e8,stroke:#174ea6,color:#ffffff
    classDef field fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef eco fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef usr fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class PGO,RPO,ODPO gov
    class DPO pilot
    class PO,PSY,MEN field
    class EMPL,TC eco
    class USERS usr
```

**Reporting cadence**

| Flow | Frequency | Artifact |
|---|---|---|
| Officer → District Prosecutor | Weekly | Caseload status, risk flags |
| District → Regional | Monthly | District KPI report (auto-generated) |
| Regional → Prosecutor General | Quarterly | Regional reintegration analytics |

---

## 3. User Journey — From Supervision to Independence

### 3.1 Rehabilitation lifecycle

```mermaid
flowchart LR
    A["1 · INTAKE<br/>Registration by officer<br/>OneID verification<br/>device handover / app install"]
    B["2 · ASSESSMENT<br/>Psychological baseline<br/>skills & needs profile<br/>risk assessment"]
    C["3 · PERSONAL PLAN<br/>Individual reintegration plan<br/>goals · courses · job track<br/>mentor & psychologist assigned"]
    D["4 · DAILY LIFE<br/>Tasks · habits · mood check-in<br/>learning · reading · applying to jobs<br/>chat with mentor · AI assistant"]
    E["5 · MILESTONES<br/>Certificates earned<br/>employment confirmed<br/>risk level reduced"]
    F["6 · GRADUATION<br/>Probation completed<br/>alumni status<br/>optional continued access"]

    A --> B --> C --> D --> E --> F
    D -->|"monthly review"| C
    E -->|"setback detected"| C

    classDef stage fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef final fill:#e6f4ea,stroke:#188038,color:#0d652d
    class A,B,C,D,E stage
    class F final
```

### 3.2 Emotional journey (first 90 days)

```mermaid
journey
    title First 90 Days of a Probation User
    section Week 1 — Intake
        Registered by officer: 3: User, Officer
        First login via OneID: 4: User
        Meets mentor in app: 5: User, Mentor
    section Weeks 2-4 — Orientation
        Completes baseline assessment: 3: User, Psychologist
        Personal plan agreed: 5: User, Officer
        First habit streak (7 days): 6: User
    section Months 2-3 — Momentum
        Enrolls in welding course: 6: User, Training Center
        First mood dip - psychologist session: 4: User, Psychologist
        CV published to Job Center: 6: User
        First job interview: 5: User, Employer
    section Day 90 — Review
        Progress review meeting: 6: User, Officer, Prosecutor
        Risk level lowered: 7: User
```

### 3.3 A day in the app

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant APP as Mobile App
    participant AI as AI Assistant
    participant M as Mentor

    U->>APP: Morning login (PIN / biometric)
    APP-->>U: Daily motivation + today's tasks
    U->>APP: Mood check-in (2 taps)
    APP-->>U: Habit reminders (reading, exercise)
    U->>APP: 20 min course lesson + 10 pages of book
    U->>AI: "Qanday qilib intervyuga tayyorlanaman?"
    AI-->>U: Interview preparation tips + practice plan
    U->>M: Message about upcoming interview
    M-->>U: Encouragement + advice
    APP-->>U: Evening summary: streak +1, 40 XP, badge progress
```
