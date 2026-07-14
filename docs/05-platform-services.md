# 05 — Platform Services

Covers: AI Assistant · Notification Flow · Communication Flow · Chat System · Calendar

---

## 10. AI Assistant Architecture

The assistant ("Hamroh" — *companion*) motivates, explains, and guides. It operates behind strict
guardrails: **no legal advice, no clinical diagnosis, no case decisions** — and it escalates to
humans when risk signals appear.

```mermaid
flowchart TB
    U["📱 User asks in Uzbek / Russian"]

    subgraph GUARD_IN["INPUT GUARDRAILS"]
        LANG["Language & intent detection"]
        SAFE["Safety classifier<br/>self-harm · crisis · abuse signals"]
        SCOPE["Scope filter<br/>legal / clinical questions → redirect"]
    end

    subgraph CORE["ASSISTANT CORE — ai-service"]
        ORCH["Orchestrator<br/>conversation state · persona policy"]
        RAG["RAG Retrieval<br/>pgvector knowledge base"]
        KB[("Knowledge Base<br/>platform guides · job/course catalog<br/>motivation content · FAQ<br/>approved by district office")]
        CTX["Consented user context<br/>goals · courses · streaks<br/>(never mood notes / chat / case files)"]
        LLM["LLM Gateway<br/>in-country proxy · no PII to provider<br/>prompt shielding"]
    end

    subgraph GUARD_OUT["OUTPUT GUARDRAILS"]
        MOD["Response moderation"]
        CITE["Grounding check<br/>answers cite KB sources"]
        TONE["Tone: respectful, hopeful, plain language"]
    end

    subgraph ESC["HUMAN ESCALATION"]
        CRISIS["🚨 Crisis protocol<br/>psychologist alerted ≤ 5 min<br/>+ hotline 1050 shown"]
        HAND["Handoff to mentor / officer<br/>with user consent"]
    end

    U --> GUARD_IN --> ORCH
    ORCH --> RAG --> KB
    ORCH --> CTX
    ORCH --> LLM --> GUARD_OUT --> U
    SAFE -->|"risk detected"| CRISIS
    ORCH -->|"needs human"| HAND

    classDef in fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef core fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef out fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef esc fill:#fce8e6,stroke:#d93025,color:#a50e0e
    class U,LANG,SAFE,SCOPE in
    class ORCH,RAG,KB,CTX,LLM core
    class MOD,CITE,TONE out
    class CRISIS,HAND esc
```

**Assistant capabilities (pilot):** platform navigation help · interview & CV coaching ·
study-plan suggestions · motivational conversations · habit encouragement · meeting reminders Q&A ·
"what should I do today?" daily briefing.

---

## 11. Notification Flow

```mermaid
flowchart LR
    subgraph TRIG["Event Triggers"]
        E1["Task assigned / due"]
        E2["Meeting in 24h / 1h"]
        E3["New message"]
        E4["Course / job updates"]
        E5["Streak & achievement"]
        E6["Official notice (prosecutor)"]
    end

    KAF[("Kafka<br/>notification.requests")]

    subgraph NS["notification-service"]
        PREF["Preference & quiet hours<br/>(22:00–07:00, non-critical)"]
        LOC["Localization<br/>uz · uz-Cyrl · ru templates"]
        ROUTE{"Channel router"}
        RETRY["Retry & fallback ladder<br/>push → in-app → SMS"]
    end

    subgraph CHN["Channels"]
        PUSH["FCM / APNs push"]
        INAPP["In-app inbox"]
        SMS["SMS gateway<br/>critical + no-smartphone users"]
        EMAIL["Email<br/>officials & partners"]
    end

    DLV["Delivery receipts →<br/>analytics + audit"]

    TRIG --> KAF --> PREF --> LOC --> ROUTE
    ROUTE --> PUSH & INAPP & SMS & EMAIL
    PUSH & SMS --> RETRY
    CHN --> DLV

    classDef t fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef n fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef c fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef d fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class E1,E2,E3,E4,E5,E6 t
    class PREF,LOC,ROUTE,RETRY n
    class PUSH,INAPP,SMS,EMAIL c
    class KAF,DLV d
```

**Priority classes:** P1 official/legal (SMS + push, no quiet hours) · P2 schedule (push, respects
quiet hours) · P3 engagement (in-app digest, max 3/day to prevent fatigue).

---

## 17. Communication Flow (Who Talks to Whom)

```mermaid
flowchart TB
    DP["District Prosecutor"]
    PO["Probation Officer"]
    PSY["Psychologist"]
    MEN["Mentor"]
    USR["User"]
    FAM["Family Member"]
    EMP["Employer"]
    TC["Training Center"]
    AI["AI Assistant"]

    DP <-->|"directives · case reviews"| PO
    DP -.->|"official notices (one-way)"| USR
    PO <-->|"secure chat · meetings"| USR
    PO <-->|"case coordination"| PSY
    PO <-->|"guidance sync"| MEN
    PSY <-->|"confidential sessions"| USR
    MEN <-->|"motivational chat"| USR
    USR <-->|"24/7 guided support"| AI
    USR -->|"applications · interview chat"| EMP
    USR -->|"course Q&A"| TC
    FAM -.->|"encouragement prompts<br/>(templated, consent-based)"| USR
    EMP -.->|"hiring status"| PO
    TC -.->|"attendance & certificates"| PO

    classDef gov fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef pro fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef cit fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef ext fill:#fef7e0,stroke:#f9ab00,color:#a56500
    class DP gov
    class PO,PSY,MEN pro
    class USR,FAM,AI cit
    class EMP,TC ext
```

**Communication policy**

| Rule | Rationale |
|---|---|
| No user↔user chat in pilot | Prevents negative peer pressure; group programs are moderated, roadmap Phase 2 |
| Psychologist channel is confidential | Officers/prosecutors see session attendance only |
| Family gets templated prompts, not free chat | Protects user privacy while enabling encouragement |
| All partner communication is job/course-scoped | No unsolicited contact with users |

---

## 18. Chat System Architecture

```mermaid
flowchart TB
    subgraph CLIENTS["Clients"]
        MC["Mobile App"]
        WC["Web Panel"]
    end

    WSGW["WebSocket Gateway<br/>sticky sessions · JWT auth<br/>heartbeat · reconnect"]

    subgraph CS["chat-service"]
        ROOM["Conversation manager<br/>policy-driven membership"]
        DELIV["Delivery engine<br/>sent → delivered → read"]
        MODQ["Moderation pipeline<br/>toxicity · threat lexicon (uz/ru)<br/>→ flags to officer, never auto-punish"]
        MEDIA["Attachment handler<br/>images · voice notes · AV-scanned"]
    end

    RD[("Redis<br/>presence · unread counters<br/>fan-out pub/sub")]
    PGC[("PostgreSQL<br/>messages, encrypted at rest")]
    S3C[("MinIO<br/>attachments")]
    KAFC[("Kafka<br/>chat.events → notifications · analytics counts")]
    OFF["Offline path:<br/>push notification via FCM"]

    CLIENTS <--> WSGW <--> CS
    CS <--> RD
    CS --> PGC
    MEDIA --> S3C
    CS --> KAFC --> OFF

    classDef c fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef s fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef d fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class MC,WC,WSGW c
    class ROOM,DELIV,MODQ,MEDIA s
    class RD,PGC,S3C,KAFC,OFF d
```

**Trust design:** users are told exactly which conversations are visible to whom. Mentor/officer
chats are subject to safeguarding review on flags; psychologist chats are confidential;
analytics receives message *counts*, never content.

---

## 22. Calendar Architecture

```mermaid
flowchart TB
    subgraph SRC["Event Sources"]
        SUP["Supervision meetings<br/>(officer / prosecutor)"]
        PSYE["Psychology sessions"]
        MENE["Mentor sessions"]
        CRSE["Course schedule<br/>(training center)"]
        JOBE["Job interviews<br/>(employer)"]
        PERS["Personal reminders<br/>(user)"]
    end

    subgraph CAL["calendar-service"]
        ENG["Scheduling engine<br/>availability · conflict detection"]
        RRULE["Recurrence (RRULE)<br/>weekly check-ins etc."]
        ATT["Attendance tracking<br/>held · missed · rescheduled"]
        REM["Reminder scheduler<br/>T-24h · T-1h · T-15m"]
    end

    subgraph VIEWS["Role Views"]
        UV["User: unified personal agenda"]
        OV["Officer: caseload calendar<br/>+ missed-meeting flags"]
        DV["Prosecutor: district meeting board"]
    end

    NOTI["notification-service"]
    CASEV["case-service<br/>attendance → progress & risk"]

    SRC --> ENG --> RRULE --> ATT
    ENG --> REM --> NOTI
    ATT --> CASEV
    CAL --> VIEWS

    classDef s fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef c fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef v fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef x fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class SUP,PSYE,MENE,CRSE,JOBE,PERS s
    class ENG,RRULE,ATT,REM c
    class UV,OV,DV v
    class NOTI,CASEV x
```

Missed **supervision** meetings raise a case flag automatically; missed **personal** items only
nudge the user — the calendar reinforces structure without becoming punitive.
