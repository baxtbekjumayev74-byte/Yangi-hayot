# 03 — Data Architecture

Covers: Database ER Diagram (logical model, core entities).

---

## 6. Database ER Diagram

The model is organized around four clusters: **Identity & Jurisdiction**, **Supervision**,
**Rehabilitation**, and **Engagement & Audit**. Every operational table carries `district_id`
for tenancy and row-level security.

### 6.1 Identity, Jurisdiction & Supervision

```mermaid
erDiagram
    REGION ||--o{ DISTRICT : "contains"
    DISTRICT ||--o{ USER_ACCOUNT : "registers"
    DISTRICT ||--o{ ORGANIZATION : "hosts"
    USER_ACCOUNT ||--o{ USER_ROLE : "is granted"
    ROLE ||--o{ USER_ROLE : "assigned via"
    USER_ACCOUNT ||--o| PROBATION_CASE : "subject of"
    USER_ACCOUNT ||--o{ FAMILY_LINK : "consents to"
    PROBATION_CASE ||--o{ CASE_ASSIGNMENT : "staffed by"
    USER_ACCOUNT ||--o{ CASE_ASSIGNMENT : "serves as officer or specialist"
    PROBATION_CASE ||--|| REINTEGRATION_PLAN : "guided by"
    PROBATION_CASE ||--o{ RISK_ASSESSMENT : "evaluated by"
    PROBATION_CASE ||--o{ TASK : "includes"
    REINTEGRATION_PLAN ||--o{ PLAN_GOAL : "defines"

    REGION {
        uuid id PK
        string name
        string code
    }
    DISTRICT {
        uuid id PK
        uuid region_id FK
        string name
        string status "pilot | active"
    }
    USER_ACCOUNT {
        uuid id PK
        uuid district_id FK
        string oneid_pinfl "encrypted"
        string full_name "encrypted"
        string phone "encrypted"
        string locale "uz | uz-Cyrl | ru"
        string status
        timestamptz created_at
    }
    ROLE {
        uuid id PK
        string code "10 role codes"
        string scope_level "national | region | district | caseload | self"
    }
    USER_ROLE {
        uuid id PK
        uuid user_id FK
        uuid role_id FK
        uuid scope_district_id FK
        timestamptz granted_at
    }
    ORGANIZATION {
        uuid id PK
        uuid district_id FK
        string type "employer | training_center"
        string name
        string verification_status
    }
    FAMILY_LINK {
        uuid id PK
        uuid user_id FK
        uuid family_member_id FK
        string consent_scope "progress_only"
        boolean active
    }
    PROBATION_CASE {
        uuid id PK
        uuid user_id FK
        uuid district_id FK
        date supervision_start
        date supervision_end
        string status "active | completed | suspended"
    }
    CASE_ASSIGNMENT {
        uuid id PK
        uuid case_id FK
        uuid staff_user_id FK
        string role_in_case "officer | psychologist | mentor"
    }
    REINTEGRATION_PLAN {
        uuid id PK
        uuid case_id FK
        int version
        string approved_by
        timestamptz approved_at
    }
    PLAN_GOAL {
        uuid id PK
        uuid plan_id FK
        string category "employment | education | wellbeing | social"
        string title
        date target_date
        string status
    }
    RISK_ASSESSMENT {
        uuid id PK
        uuid case_id FK
        uuid assessed_by FK
        int score "1-100"
        string level "low | medium | high"
        timestamptz assessed_at
    }
    TASK {
        uuid id PK
        uuid case_id FK
        uuid created_by FK
        string title
        string type "meeting | document | activity"
        date due_date
        string status
    }
```

### 6.2 Rehabilitation: Employment, Education, Library

```mermaid
erDiagram
    ORGANIZATION ||--o{ JOB_VACANCY : "publishes"
    JOB_VACANCY ||--o{ JOB_APPLICATION : "receives"
    USER_ACCOUNT ||--o{ JOB_APPLICATION : "submits"
    USER_ACCOUNT ||--o{ CV_DOCUMENT : "builds"
    JOB_APPLICATION ||--o| EMPLOYMENT_RECORD : "results in"
    ORGANIZATION ||--o{ COURSE : "offers"
    COURSE ||--o{ COURSE_MODULE : "structured as"
    COURSE_MODULE ||--o{ LESSON : "contains"
    USER_ACCOUNT ||--o{ ENROLLMENT : "enrolls via"
    COURSE ||--o{ ENROLLMENT : "accepts"
    ENROLLMENT ||--o{ LESSON_PROGRESS : "tracked by"
    ENROLLMENT ||--o| CERTIFICATE : "earns"
    MEDIA_ITEM ||--o{ MEDIA_PROGRESS : "consumed via"
    USER_ACCOUNT ||--o{ MEDIA_PROGRESS : "reads or listens"

    JOB_VACANCY {
        uuid id PK
        uuid organization_id FK
        uuid district_id FK
        string title
        string profession_code
        numeric salary_from
        string status "draft | published | filled"
    }
    JOB_APPLICATION {
        uuid id PK
        uuid vacancy_id FK
        uuid user_id FK
        uuid cv_id FK
        string status "applied | interview | hired | rejected"
        timestamptz applied_at
    }
    CV_DOCUMENT {
        uuid id PK
        uuid user_id FK
        jsonb content "skills, experience, education"
        string visibility "private | job_center"
        int version
    }
    EMPLOYMENT_RECORD {
        uuid id PK
        uuid user_id FK
        uuid organization_id FK
        date start_date
        string confirmation_status "employer_confirmed"
    }
    COURSE {
        uuid id PK
        uuid organization_id FK
        string title
        string profession_code
        string format "online | offline | hybrid"
        int duration_hours
    }
    COURSE_MODULE {
        uuid id PK
        uuid course_id FK
        string title
        int order_index
    }
    LESSON {
        uuid id PK
        uuid module_id FK
        string title
        string content_type "video | text | quiz"
        uuid media_id FK
    }
    ENROLLMENT {
        uuid id PK
        uuid course_id FK
        uuid user_id FK
        string status "active | completed | dropped"
        numeric progress_pct
    }
    LESSON_PROGRESS {
        uuid id PK
        uuid enrollment_id FK
        uuid lesson_id FK
        string status
        timestamptz completed_at
    }
    CERTIFICATE {
        uuid id PK
        uuid enrollment_id FK
        string certificate_no
        string verify_hash
        timestamptz issued_at
    }
    MEDIA_ITEM {
        uuid id PK
        string type "book | audiobook | video | podcast"
        string title
        string language
        string category "motivation | profession | psychology"
        string storage_key "S3 object"
    }
    MEDIA_PROGRESS {
        uuid id PK
        uuid media_id FK
        uuid user_id FK
        numeric position "page or seconds"
        numeric completion_pct
        timestamptz updated_at
    }
```

### 6.3 Well-being, Engagement & Audit

```mermaid
erDiagram
    USER_ACCOUNT ||--o{ MOOD_ENTRY : "checks in"
    USER_ACCOUNT ||--o{ PSY_SESSION : "attends"
    USER_ACCOUNT ||--o{ HABIT : "commits to"
    HABIT ||--o{ HABIT_LOG : "logged daily"
    USER_ACCOUNT ||--o{ USER_GOAL : "sets"
    USER_ACCOUNT ||--o{ USER_ACHIEVEMENT : "unlocks"
    ACHIEVEMENT ||--o{ USER_ACHIEVEMENT : "instantiated as"
    CONVERSATION ||--o{ MESSAGE : "contains"
    USER_ACCOUNT ||--o{ CONVERSATION_MEMBER : "participates via"
    CONVERSATION ||--o{ CONVERSATION_MEMBER : "has"
    USER_ACCOUNT ||--o{ MEETING : "attends"
    USER_ACCOUNT ||--o{ NOTIFICATION : "receives"
    USER_ACCOUNT ||--o{ AI_CONVERSATION : "talks to assistant"
    USER_ACCOUNT ||--o{ AUDIT_EVENT : "actions recorded as"

    MOOD_ENTRY {
        uuid id PK
        uuid user_id FK
        int mood_score "1-5"
        string tags "sleep, family, work"
        text note "encrypted, user-private"
        timestamptz created_at
    }
    PSY_SESSION {
        uuid id PK
        uuid user_id FK
        uuid psychologist_id FK
        string type "scheduled | crisis"
        text clinical_note "encrypted, psychologist-only"
        timestamptz held_at
    }
    HABIT {
        uuid id PK
        uuid user_id FK
        string title
        string schedule "daily | weekly"
        int current_streak
        int best_streak
    }
    HABIT_LOG {
        uuid id PK
        uuid habit_id FK
        date log_date
        boolean done
    }
    USER_GOAL {
        uuid id PK
        uuid user_id FK
        uuid plan_goal_id FK "optional link to official plan"
        string title
        numeric progress_pct
        string status
    }
    ACHIEVEMENT {
        uuid id PK
        string code
        string title
        string tier "bronze | silver | gold"
        int xp_reward
    }
    USER_ACHIEVEMENT {
        uuid id PK
        uuid user_id FK
        uuid achievement_id FK
        timestamptz unlocked_at
    }
    CONVERSATION {
        uuid id PK
        string type "user_mentor | user_officer | group"
        string moderation_level
        timestamptz created_at
    }
    CONVERSATION_MEMBER {
        uuid id PK
        uuid conversation_id FK
        uuid user_id FK
        string member_role
    }
    MESSAGE {
        uuid id PK
        uuid conversation_id FK
        uuid sender_id FK
        text body "encrypted at rest"
        string status "sent | delivered | read"
        timestamptz sent_at
    }
    MEETING {
        uuid id PK
        uuid case_id FK
        uuid organizer_id FK
        string type "supervision | psychology | mentoring"
        timestamptz starts_at
        string status "planned | held | missed"
    }
    NOTIFICATION {
        uuid id PK
        uuid user_id FK
        string channel "push | sms | inapp"
        string template_code
        string status
        timestamptz sent_at
    }
    AI_CONVERSATION {
        uuid id PK
        uuid user_id FK
        string topic
        boolean escalated_to_human
        timestamptz created_at
    }
    AUDIT_EVENT {
        uuid id PK
        uuid actor_id FK
        string action
        string resource_type
        uuid resource_id
        uuid district_id
        jsonb context
        timestamptz occurred_at "append-only"
    }
```

### Data protection notes

| Concern | Mechanism |
|---|---|
| Jurisdiction isolation | PostgreSQL RLS on `district_id` / `region_id`, enforced for every professional role |
| Identity data | PINFL, name, phone encrypted at column level (AES-256, keys in Vault) |
| Clinical confidentiality | `PSY_SESSION.clinical_note` readable **only** by the authoring psychologist; supervisors see attendance and risk level, never content |
| Analytics | Warehouse receives pseudonymized IDs; re-identification only inside the operational domain |
| Retention | Case data: statutory period; chat: 12 months rolling; audit: 7 years, WORM |
