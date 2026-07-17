# Yangi Hayot — Otaliqqa olingan shaxslar monitoring platformasi

> O'zbekiston Respublikasi Prokuratura organlari uchun **ichki (internal) enterprise monitoring tizimi**.

**MUHIM:** Platformadan **faqat prokuratura xodimlari** foydalanadi. Fuqarolar uchun hech qanday kabinet, login yoki tashqi kirish **mavjud emas**.

---

## 1. Loyiha maqsadi

| # | Maqsad |
|---|--------|
| 1 | Otaliqqa olingan shaxslarni yagona bazada monitoring qilish |
| 2 | Shaxslar muammolarini (bandlik, uy-joy, kredit, davolanish...) nazorat qilish |
| 3 | Elektron topshiriqlar orqali vazifalarni boshqarish |
| 4 | Rahbarlar uchun real vaqt statistikasi va KPI |
| 5 | Barcha harakatlarni to'liq audit qilish |
| 6 | Elektron ish yuritish (hujjatlar, versiyalash, QR tekshiruv) |

## 2. Hujjatlar indeksi

| Hujjat | Qamrab olingan mavzular |
|---|---|
| [01 — System Architecture](docs/01-system-architecture.md) | To'liq tizim arxitekturasi · komponentlar · Deployment (Docker/Nginx/CI-CD) |
| [02 — Database Architecture](docs/02-database-architecture.md) | ER diagramma · to'liq jadvallar ro'yxati · Prisma schema · index/soft-delete strategiyasi |
| [03 — Backend Architecture](docs/03-backend-architecture.md) | NestJS modul tuzilishi · folder structure · REST API strukturasi · Swagger · BullMQ |
| [04 — Frontend Architecture](docs/04-frontend-architecture.md) | Next.js folder structure · UI/UX sahifalar · Sidebar menyu · State management |
| [05 — Roles & Security](docs/05-roles-security.md) | 7 ta rol · to'liq Permission Matrix · JWT/2FA/RBAC · Security arxitekturasi |
| [06 — Workflow & User Flows](docs/06-workflow-userflows.md) | Ish jarayoni (workflow) · barcha foydalanuvchi oqimlari |
| [07 — Notifications & AI](docs/07-notifications-ai.md) | Bildirishnoma arxitekturasi (Telegram/SMS/Email/Push) · Smart AI moduli · Smart Reminder |
| [08 — Standards & Roadmap](docs/08-standards-roadmap.md) | Coding standards · Enterprise best practices · bosqichma-bosqich roadmap |

## 3. Asosiy modullar (24)

| # | Modul | # | Modul |
|---|-------|---|-------|
| 1 | Dashboard (real-time statistika) | 13 | Workflow (ish jarayoni) |
| 2 | Otaliqqa olingan shaxslar reestri | 14 | Audit (to'liq jurnal) |
| 3 | Timeline (shaxs tarixi) | 15 | Arxiv |
| 4 | Muammolar nazorati | 16 | Foydalanuvchilar va rollar |
| 5 | Elektron topshiriqlar | 17 | KPI (xodim samaradorligi) |
| 6 | Tashriflar (GPS, foto, audio, video) | 18 | Smart AI (risk, tavsiya, hisobot) |
| 7 | Elektron hujjatlar (versioning, QR) | 19 | Smart Reminder |
| 8 | Excel import/export | 20 | Monitoring Center (katta ekran) |
| 9 | Analitika (ECharts) | 21 | Security (JWT, 2FA, RBAC...) |
| 10 | GIS xarita (Yandex Maps) | 22 | System Settings |
| 11 | Aqlli qidiruv | 23 | REST API + Swagger |
| 12 | Bildirishnomalar | 24 | PostgreSQL Database |

## 4. Texnologiyalar

| Qatlam | Texnologiya |
|---|---|
| Frontend | Next.js 15 · React 19 · TypeScript · TailwindCSS · shadcn/ui · TanStack Query · React Hook Form · Zod · ECharts · Yandex Maps |
| Backend | NestJS 11 · TypeScript · Prisma ORM · PostgreSQL 16 · Redis 7 · BullMQ |
| Storage | MinIO (S3-compatible, mamlakat ichida) |
| Auth | JWT (access + refresh) · RBAC · 2FA (TOTP) |
| Deployment | Docker · Docker Compose · Nginx · GitHub Actions |

## 5. Rollar (7)

| # | Rol | Ko'lam |
|---|-----|--------|
| 1 | Super Admin | Butun tizim, texnik boshqaruv |
| 2 | Respublika | Respublika miqyosidagi rahbariyat |
| 3 | Viloyat | O'z viloyati doirasida |
| 4 | Tuman | O'z tumani doirasida |
| 5 | Prokuror | O'ziga biriktirilgan shaxslar |
| 6 | Operator | Ma'lumot kiritish (tuman doirasida) |
| 7 | Kuzatuvchi | Faqat ko'rish (read-only) |

## 6. Dizayn tamoyillari

1. **Internal-only** — tizim faqat ichki tarmoq/VPN orqali, faqat xodimlar uchun.
2. **Least privilege** — har bir rol faqat o'z hududi va vakolati doirasidagi ma'lumotni ko'radi (data scoping).
3. **Audit-first** — har bir CRUD, login, export harakati o'zgarishlar (old/new value) bilan jurnalga yoziladi.
4. **Soft delete** — hech narsa jismonan o'chirilmaydi; arxiv va tiklash imkoniyati.
5. **Sovereign data** — barcha ma'lumot va fayllar O'zbekiston hududidagi serverlarda saqlanadi.
6. **AI — yordamchi, qaror emas** — AI tavsiya beradi va hisobot yozadi; yuridik qarorni faqat inson qabul qiladi.
