# 06 — Rehabilitation Modules

Covers: Employment · Education · Digital Library · Goal Tracking · Mental Health

---

## 19. Employment Module Architecture

The economic engine of reintegration: verified employers, skill-matched vacancies, and a
dignified CV that presents the person's future, not their past.

```mermaid
flowchart LR
    subgraph SUPPLY["Employer Side"]
        EREG["Employer verification<br/>by district office"]
        VAC["Vacancy management<br/>publish · quotas · incentives info"]
        REV["Candidate review<br/>sees CV + certificates only<br/>(no case details)"]
    end

    subgraph ENGINE["employment-service"]
        MATCH["Matching engine<br/>profession codes · skills ·<br/>location · schedule constraints"]
        CVB["CV Builder<br/>guided steps · templates ·<br/>auto-import certificates"]
        APPT["Application tracker<br/>applied → interview → hired"]
        CONF["Employment confirmation<br/>employer attests start date"]
    end

    subgraph DEMAND["User Side"]
        SRCH["Job search & filters<br/>OpenSearch"]
        APLY["One-tap apply with CV"]
        PREP["Interview prep<br/>AI coaching + mentor review"]
    end

    OUT1["case-service<br/>employment = plan milestone"]
    OUT2["analytics<br/>employment rate KPI"]
    OFFJ["Officer visibility:<br/>application activity ·<br/>employment status"]

    SUPPLY --> ENGINE --> DEMAND
    CVB --> APLY
    CONF --> OUT1 & OUT2
    APPT --> OFFJ

    classDef sup fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef eng fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef dem fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef out fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class EREG,VAC,REV sup
    class MATCH,CVB,APPT,CONF eng
    class SRCH,APLY,PREP dem
    class OUT1,OUT2,OFFJ out
```

**Dignity safeguards:** employers never see probation case data — only what the user chooses to
share in the CV; the platform brands candidates as "Yangi Hayot program participants" only with
user consent.

---

## 20. Education Module Architecture

```mermaid
flowchart LR
    subgraph PROVIDE["Training Center Side"]
        ACC["Center accreditation<br/>district approval"]
        CAT["Course catalog<br/>welding · electrics · IT · tailoring ·<br/>driving · entrepreneurship"]
        SCHED["Cohort scheduling<br/>online / offline / hybrid"]
        CERT["Certificate issuance<br/>QR-verifiable"]
    end

    subgraph ENGINE["education-service"]
        RECO["Recommendation engine<br/>skills gap × local labor demand"]
        ENR["Enrollment workflow<br/>self-enroll or plan-assigned"]
        PROG["Progress tracking<br/>lessons · quizzes · attendance"]
        GAMI["Learning streaks & XP"]
    end

    subgraph LEARN["User Side"]
        BROWSE["Browse & preview courses"]
        STUDY["Lesson player<br/>video · text · quiz · offline packs"]
        PORTF["My certificates<br/>→ auto-attach to CV"]
    end

    LINK1["employment-service<br/>certificate → job matching boost"]
    LINK2["case-service<br/>course completion = plan milestone"]

    PROVIDE --> ENGINE --> LEARN
    CERT --> PORTF --> LINK1
    PROG --> LINK2

    classDef p fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef e fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef l fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef k fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class ACC,CAT,SCHED,CERT p
    class RECO,ENR,PROG,GAMI e
    class BROWSE,STUDY,PORTF l
    class LINK1,LINK2 k
```

---

## 21. Digital Library Architecture

```mermaid
flowchart LR
    subgraph CONTENT["Content Supply"]
        CUR["Curation board<br/>district office + psychologists"]
        ING["Ingestion pipeline<br/>EPUB · PDF · MP3 · MP4<br/>transcode · watermark"]
        META["Cataloging<br/>language · category · reading level"]
    end

    subgraph LIB["library-service"]
        CATL["Catalog & search<br/>OpenSearch facets"]
        LIC["Access control<br/>age/content policy"]
        PRG["Progress engine<br/>pages · playback position ·<br/>cross-device resume"]
        RECS["Recommendations<br/>editorial shelves + history"]
    end

    subgraph DELIVERY["Delivery"]
        CDN2["CDN edge<br/>HLS audio/video streaming"]
        OFFP["Offline packs<br/>DRM-light, device-bound"]
    end

    subgraph EXP["User Experience"]
        RDR["📖 Reader<br/>fonts · night mode · bookmarks"]
        AUD["🎧 Audiobook player<br/>speed · sleep timer · chapters"]
        VID["🎬 Motivational videos<br/>playlists · subtitles uz/ru"]
    end

    GAM["goal-service<br/>reading streaks · 'read 12 books' badge"]

    CONTENT --> LIB --> DELIVERY --> EXP
    PRG --> GAM
    EXP --> PRG

    classDef c fill:#fef7e0,stroke:#f9ab00,color:#a56500
    classDef s fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef d fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    classDef e fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    class CUR,ING,META c
    class CATL,LIC,PRG,RECS s
    class CDN2,OFFP d
    class RDR,AUD,VID e
```

**Pilot catalog target:** 200 books (uz/ru), 60 audiobooks, 100 motivational videos, 12
profession explainer series — reviewed by the curation board before publication.

---

## 23. Goal Tracking Architecture (Goals · Habits · Achievements)

```mermaid
flowchart LR
    subgraph SET["Goal Setting"]
        OFFICIAL["Plan goals<br/>from reintegration plan<br/>(officer-agreed)"]
        PERSONAL["Personal goals<br/>user-defined, private by default"]
        SMART["SMART decomposition<br/>goal → milestones → weekly actions"]
    end

    subgraph TRACK["goal-service Engines"]
        HABIT["Habit engine<br/>daily/weekly schedule ·<br/>streak calculation · grace days"]
        PROGE["Progress engine<br/>auto-progress from events:<br/>lesson done · pages read ·<br/>application sent · session attended"]
        XP["Gamification engine<br/>XP · levels · badges<br/>bronze → silver → gold"]
    end

    subgraph FEEDBACK["Feedback Loops"]
        DAILYV["Daily view<br/>today's 3 focus actions"]
        CELE["Celebrations<br/>confetti · certificates · milestone cards"]
        NUDGE["Smart nudges<br/>streak-at-risk · comeback support<br/>(never shaming)"]
    end

    KAFG[("Kafka progress.events")]
    RISKG["risk-service<br/>engagement drop = early-warning signal"]
    REPG["Officer/prosecutor dashboards<br/>plan-goal progress only"]

    SET --> TRACK --> FEEDBACK
    TRACK --> KAFG --> RISKG & REPG

    classDef s fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef t fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef f fill:#d2e3fc,stroke:#174ea6,color:#174ea6
    classDef k fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class OFFICIAL,PERSONAL,SMART s
    class HABIT,PROGE,XP t
    class DAILYV,CELE,NUDGE f
    class KAFG,RISKG,REPG k
```

**Behavioral design:** losing a streak triggers *comeback* framing ("start a new streak today"),
not failure messaging. Personal goals stay private; only plan goals surface to supervisors.

---

## 24. Mental Health Module

The clinically sensitive core. Confidentiality is architectural, not procedural.

```mermaid
flowchart LR
    subgraph SELF["Self-Care Layer (User)"]
        MOODT["Mood tracker<br/>2-tap daily check-in + tags"]
        JOUR["Private journal<br/>user-only, encrypted"]
        EXER["Guided exercises<br/>breathing · CBT-lite · sleep hygiene"]
        PSYEDU["Psychoeducation library<br/>stress · anger · addiction · family"]
    end

    subgraph CARE["Professional Care Layer"]
        SCREEN["Standardized screenings<br/>PHQ-9 · GAD-7 (localized)<br/>scheduled by psychologist"]
        SESS["Session management<br/>in-person · video · notes (sealed)"]
        CAREPL["Care plan<br/>psychologist-managed"]
    end

    subgraph SAFETY["Safety Net"]
        SIGNAL["Signal detection<br/>mood decline pattern ·<br/>AI-flagged crisis language ·<br/>screening thresholds"]
        TRIAGE2{"Severity triage"}
        ROUTINE["Routine: next session"]
        PRIOR["Priority: psychologist<br/>contact within 24h"]
        CRIT["🚨 Crisis: immediate call protocol<br/>+ hotline 1050"]
    end

    subgraph VISIBILITY["What Supervisors See"]
        AGG["Attendance · engagement ·<br/>clinician-set risk level ONLY<br/>— never content, scores or notes"]
    end

    SELF --> SIGNAL
    SCREEN --> SIGNAL
    SIGNAL --> TRIAGE2 --> ROUTINE & PRIOR & CRIT
    CARE --> AGG
    MOODT -.->|"trend only, if user consents"| SESS

    classDef s fill:#e8f0fe,stroke:#1a73e8,color:#174ea6
    classDef c fill:#e6f4ea,stroke:#188038,color:#0d652d
    classDef f fill:#fce8e6,stroke:#d93025,color:#a50e0e
    classDef v fill:#f1f3f4,stroke:#5f6368,color:#3c4043
    class MOODT,JOUR,EXER,PSYEDU s
    class SCREEN,SESS,CAREPL c
    class SIGNAL,TRIAGE2,ROUTINE,PRIOR,CRIT f
    class AGG v
```

**Clinical governance:** screenings and protocols approved by a licensed supervising psychologist;
crisis protocol co-designed with the regional mental-health service; users are told at onboarding
exactly what is and is not shared.
