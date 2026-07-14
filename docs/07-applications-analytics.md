# 07 — Applications, Analytics & Reporting

Covers: Analytics Dashboard · Mobile Application Flow · Web Admin Panel Flow · Reporting System
Plus: full module specifications for the User App and the Admin Dashboard.

---

## 13. Mobile Application Flow (User App)

```mermaid
flowchart TB
    SPLASH["Splash / PIN · Biometric"] --> HOME

    subgraph HOME["🏠 HOME DASHBOARD"]
        MOTIV["🌅 Daily Motivation<br/>quote · verse · story"]
        TODAY["✅ Today: 3 focus actions"]
        RINGS["Progress rings<br/>habits · learning · reading"]
        QUICK["Quick mood check-in"]
    end

    subgraph GROW["🌱 GROWTH"]
        TASKS["Tasks"]
        GOALS["Goal Tracker"]
        HABITS["Habit Tracker"]
        ACH["Achievements & XP"]
        MOOD["Mood Tracker"]
    end

    subgraph LEARNH["📚 LEARN"]
        CRS["Courses"]
        LIBR["Digital Library"]
        AUDB["Audiobooks"]
        MVID["Motivational Videos"]
    end

    subgraph WORK["💼 WORK"]
        JOBC["Job Center"]
        CVB["CV Builder"]
        APPL["My Applications"]
    end

    subgraph CONNECT["💬 CONNECT"]
        CHT["Chat<br/>mentor · officer · psychologist"]
        AIA["AI Assistant 'Hamroh'"]
        CALU["Calendar"]
        NOTIF["Notifications"]
    end

    subgraph ME["👤 ME"]
        PROF["Profile & journey timeline"]
        SETT["Settings<br/>language · privacy · quiet hours"]
    end

    HOME --> GROW & LEARNH & WORK & CONNECT & ME
    QUICK --> MOOD
    TODAY --> TASKS
    CRS -->|"certificate earned"| CVB
    JOBC --> APPL
    CALU --> CHT

    classDef h fill:#1a73e8,stroke:#174ea6,color:#ffffff
    classDef g fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef l fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef w fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef c fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef m fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class MOTIV,TODAY,RINGS,QUICK h
    class TASKS,GOALS,HABITS,ACH,MOOD g
    class CRS,LIBR,AUDB,MVID l
    class JOBC,CVB,APPL w
    class CHT,AIA,CALU,NOTIF c
    class PROF,SETT m
```

### User application — module specification

| Module | Purpose | Key features |
|---|---|---|
| Home Dashboard | Daily anchor | Motivation card, 3 focus actions, progress rings, streak flame |
| Daily Motivation | Positive start | Curated quotes/stories, save favorites, share to mentor |
| Tasks | Structure | Official tasks + self tasks, due dates, evidence upload |
| Achievements | Momentum | XP, levels, badges (bronze/silver/gold), milestone cards |
| Goal Tracker | Direction | Plan goals + private goals, SMART milestones, progress % |
| Habit Tracker | Discipline | Daily/weekly habits, streaks with grace day, calendar heat view |
| Mood Tracker | Self-awareness | 2-tap check-in, tags, private notes, personal trend chart |
| Courses | Skills | Catalog, lesson player, quizzes, offline packs, certificates |
| Digital Library | Growth | Reader with night mode, bookmarks, reading goals |
| Audiobooks | Accessibility | Player with speed/sleep timer, chapter resume |
| Motivational Videos | Inspiration | Playlists, subtitles uz/ru, success stories of alumni |
| Job Center | Livelihood | Search/filter, one-tap apply, application status, saved jobs |
| CV Builder | Presentation | Guided wizard, templates, auto-import certificates, PDF export |
| Chat | Human support | Mentor/officer/psychologist channels, voice notes, read receipts |
| AI Assistant | 24/7 companion | Guided help, coaching, daily briefing, crisis-safe escalation |
| Calendar | Reliability | Unified agenda, reminders, reschedule requests |
| Notifications | Awareness | Inbox, priority levels, quiet hours |
| Profile | Identity | Journey timeline, documents, certificates wallet |
| Settings | Control | Language (uz/uz-Cyrl/ru), privacy consents, accessibility, PIN |

---

## 14. Web Admin Panel Flow (Professional Roles)

```mermaid
flowchart TB
    LOGIN["🔐 Login — OneID + MFA"] --> RBACX{"Role-based workspace"}

    subgraph PW["PROSECUTOR WORKSPACE (District / Regional)"]
        POVER["📊 Executive dashboard<br/>KPIs · trends · heat map"]
        PRISK["⚠️ Risk board<br/>flagged cases · early warnings"]
        PREP["📄 Reports<br/>generate · schedule · sign"]
        PAPPR["✔️ Approvals<br/>plans · partner verifications"]
    end

    subgraph OW["OFFICER WORKSPACE"]
        OCASE["📁 My caseload (≤25)<br/>case timeline · notes"]
        OTASK["Task management<br/>assign · verify evidence"]
        OMEET["Meeting calendar<br/>attendance flags"]
        OCHAT["Secure chat"]
    end

    subgraph SW["SPECIALIST WORKSPACES"]
        PSYW["🧠 Psychologist<br/>sessions · screenings · care plans<br/>(sealed clinical notes)"]
        MENW["🤝 Mentor<br/>assigned users · chat · goal support"]
    end

    subgraph XW["PARTNER PORTAL"]
        EMPW["🏭 Employer<br/>vacancies · candidates · confirm hires"]
        TCW["🎓 Training Center<br/>courses · cohorts · certificates"]
    end

    subgraph AW["ADMIN WORKSPACE (Super Admin)"]
        AUSR["User & role management"]
        ACFG["Content & configuration<br/>library · motivation · templates"]
        ALOG["System logs · audit UI"]
        AHEALTH["Platform health · usage"]
    end

    RBACX --> PW & OW & SW & XW & AW

    classDef p fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef o fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef s fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef x fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef a fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class POVER,PRISK,PREP,PAPPR p
    class OCASE,OTASK,OMEET,OCHAT o
    class PSYW,MENW s
    class EMPW,TCW x
    class AUSR,ACFG,ALOG,AHEALTH a
```

### Admin dashboard — feature specification

| Feature | Description | Primary roles |
|---|---|---|
| Charts | Trend lines: engagement, employment, course completion, mood index (aggregated) | Prosecutors, Admin |
| User statistics | Active users, DAU/WAU, module adoption, cohort comparisons | Prosecutors, Admin |
| Progress reports | Per-case and aggregate plan-goal progress, milestone timeline | Officer, Prosecutors |
| Heat maps | Mahalla-level engagement & employment map of district; calendar heat of activity | Prosecutors |
| Risk indicators | Composite early-warning score: engagement drop, missed meetings, mood trend (aggregated), officer flags | Officer, Prosecutors |
| Task completion | Completion rates by user, type and officer; overdue queue | Officer, Prosecutors |
| Meeting calendar | District meeting board, attendance analytics, no-show alerts | Officer, Prosecutors |
| Analytics | Drill-down explorer with export (jurisdiction-scoped) | Prosecutors, Admin |
| Notifications | Broadcast official notices, template management, delivery stats | Prosecutors, Admin |
| System logs | Audit trail viewer, security alerts, session history | Admin, Prosecutors (scoped) |

---

## 12. Analytics Dashboard Architecture

```mermaid
flowchart LR
    subgraph IN["Inputs"]
        KAFA[("Kafka<br/>progress · case · usage events")]
        OLTPA[("Operational DBs<br/>nightly snapshots")]
    end

    subgraph PIPE["Analytics Pipeline — analytics-service"]
        ANON["Pseudonymizer<br/>drops direct identifiers"]
        ETL["Stream + batch ETL<br/>dbt models"]
        DWHA[("Warehouse<br/>star schema:<br/>fact_engagement · fact_employment ·<br/>fact_learning · fact_wellbeing_agg ·<br/>dim_user_pseudo · dim_district · dim_time")]
        KPI["KPI engine<br/>reintegration score ·<br/>risk composite · benchmarks"]
    end

    subgraph OUT["Consumption"]
        EXEC["Executive dashboards<br/>district → region → national roll-up"]
        HEAT["Heat maps<br/>geo + calendar"]
        RISKB["Risk early-warning board"]
        EXPORT["Governed exports<br/>watermarked · audited"]
    end

    IN --> ANON --> ETL --> DWHA --> KPI --> OUT

    classDef i fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    classDef p fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef o fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    class KAFA,OLTPA i
    class ANON,ETL,DWHA,KPI p
    class EXEC,HEAT,RISKB,EXPORT o
```

**North-star KPIs (pilot):** ≥70% weekly active engagement · ≥40% employed or in training by
month 6 · ≥60% course completion · measurable mood-index improvement · zero probation
violations among top-engagement quartile (hypothesis to validate).

**Small-cohort privacy rule:** with ~50 users, any aggregate slice smaller than 5 people is
suppressed to prevent indirect identification.

---

## 25. Reporting System

```mermaid
flowchart TB
    subgraph DEF["Report Definitions"]
        T1["Weekly caseload report<br/>(officer)"]
        T2["Monthly district KPI report<br/>(district prosecutor)"]
        T3["Quarterly regional analytics<br/>(regional prosecutor)"]
        T4["Pilot evaluation report<br/>(steering committee)"]
        T5["Ad-hoc case summary<br/>(court / official request)"]
    end

    subgraph GEN["reporting-service"]
        SCHEDR["Scheduler<br/>cron + on-demand"]
        BUILD["Report builder<br/>templates (uz-Cyrl official format)<br/>charts · tables · narrative blocks"]
        RENDER["Renderer<br/>PDF (signed) · Excel"]
        SIGNR["E-imzo digital signature<br/>(official reports)"]
    end

    subgraph GOVR["Governance"]
        SCOPEC["Jurisdiction check<br/>data ≤ requester scope"]
        WATERM["Watermark + recipient stamp"]
        AUDR["Every generation & download<br/>→ audit trail"]
    end

    STORE2[("Report archive<br/>MinIO, retention policy")]
    DELIV2["Delivery<br/>in-panel · secure link · print"]

    DEF --> SCHEDR --> BUILD --> RENDER --> SIGNR --> STORE2 --> DELIV2
    BUILD --> SCOPEC
    RENDER --> WATERM
    DELIV2 --> AUDR

    classDef d fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef g fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef v fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef s fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class T1,T2,T3,T4,T5 d
    class SCHEDR,BUILD,RENDER,SIGNR g
    class SCOPEC,WATERM,AUDR v
    class STORE2,DELIV2 s
```

**Official format note:** prosecutorial reports render in the state document format (Uzbek
Cyrillic headers, official seals block, E-imzo signature line) so platform output slots directly
into existing government workflows.
