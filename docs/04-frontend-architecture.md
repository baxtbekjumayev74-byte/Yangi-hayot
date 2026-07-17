# 04 — Frontend Architecture

Next.js 15 (App Router) · React 19 · TypeScript · TailwindCSS · shadcn/ui · TanStack Query · React Hook Form + Zod · ECharts · Yandex Maps.

---

## 1. Folder Structure

```
apps/web/
├── src/
│   ├── app/                              # App Router
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx            # login + 2FA bosqichi
│   │   │   └── layout.tsx                # minimal layout (sidebar yo'q)
│   │   │
│   │   ├── (main)/                       # asosiy ish maydoni
│   │   │   ├── layout.tsx                # Sidebar + Header + AuthGuard
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── persons/
│   │   │   │   ├── page.tsx              # reestr (jadval + filterlar)
│   │   │   │   ├── new/page.tsx          # bosqichli (wizard) forma
│   │   │   │   └── [id]/
│   │   │   │       ├── page.tsx          # profil (umumiy ma'lumot)
│   │   │   │       ├── timeline/page.tsx
│   │   │   │       ├── problems/page.tsx
│   │   │   │       ├── tasks/page.tsx
│   │   │   │       ├── visits/page.tsx
│   │   │   │       ├── documents/page.tsx
│   │   │   │       └── edit/page.tsx
│   │   │   ├── problems/page.tsx         # umumiy muammolar boshqaruvi
│   │   │   ├── tasks/
│   │   │   │   ├── page.tsx              # barcha topshiriqlar (rahbar ko'rinishi)
│   │   │   │   └── my/page.tsx           # mening topshiriqlarim
│   │   │   ├── visits/page.tsx           # tashriflar jadvali + kalendar
│   │   │   ├── documents/page.tsx
│   │   │   ├── map/page.tsx              # GIS xarita
│   │   │   ├── analytics/page.tsx
│   │   │   ├── kpi/page.tsx
│   │   │   ├── search/page.tsx           # advanced qidiruv
│   │   │   ├── notifications/page.tsx
│   │   │   ├── archive/page.tsx
│   │   │   ├── audit/page.tsx
│   │   │   ├── ai/page.tsx               # AI yordamchi ish maydoni
│   │   │   ├── admin/
│   │   │   │   ├── users/page.tsx
│   │   │   │   ├── roles/page.tsx
│   │   │   │   ├── regions/page.tsx      # hudud/mahalla lug'atlari
│   │   │   │   ├── dictionaries/page.tsx # muammo turlari, statuslar
│   │   │   │   ├── security/page.tsx     # IP whitelist, sessiyalar
│   │   │   │   └── settings/page.tsx     # logotip, tashkilot
│   │   │   └── profile/page.tsx          # shaxsiy sozlamalar, 2FA, qurilmalar
│   │   │
│   │   ├── monitoring/page.tsx           # Monitoring Center (fullscreen, alohida layout)
│   │   ├── verify/[qr]/page.tsx          # QR hujjat tekshiruvi (ichki)
│   │   └── layout.tsx                    # root: providers, fonts, theme
│   │
│   ├── components/
│   │   ├── ui/                           # shadcn/ui primitivlar
│   │   ├── layout/                       # Sidebar, Header, Breadcrumbs, UserMenu
│   │   ├── data-table/                   # universal jadval (sort/filter/pagination/export)
│   │   ├── charts/                       # ECharts wrapperlar (Pie/Bar/Line/Heatmap/KPI)
│   │   ├── map/                          # Yandex Maps wrapperlar (markers, heatmap, radius)
│   │   ├── forms/                        # RHF + Zod form primitivlar, FileUpload, GpsPicker
│   │   └── widgets/                      # dashboard widgetlari
│   │
│   ├── features/                         # domenga bog'liq UI logika
│   │   ├── persons/ {api,hooks,components,schemas}/
│   │   ├── problems/ ...
│   │   ├── tasks/ ...
│   │   ├── visits/ ...
│   │   ├── documents/ ...
│   │   ├── dashboard/ ...
│   │   ├── ai/ ...
│   │   └── auth/ ...
│   │
│   ├── lib/
│   │   ├── api-client.ts                 # fetch wrapper: token refresh, envelope unwrap
│   │   ├── query-client.ts               # TanStack Query konfiguratsiya
│   │   ├── ws.ts                         # socket.io client
│   │   ├── permissions.ts                # can(user, "persons.create")
│   │   └── utils.ts
│   │
│   ├── stores/                           # Zustand (faqat UI/session state)
│   │   ├── auth.store.ts
│   │   ├── ui.store.ts                   # sidebar collapse, theme
│   │   └── filters.store.ts              # saqlanadigan jadval filterlari
│   │
│   ├── i18n/                             # uz-latin (asosiy), uz-cyrillic, ru
│   └── types/                            # @yangi-hayot/shared dan re-export
├── middleware.ts                          # auth cookie tekshiruv, redirect
└── tailwind.config.ts
```

## 2. Sidebar Menu (rolga qarab filtrlash)

```
📊 Dashboard
👥 Otaliqqa olingan shaxslar
   ├─ Reestr
   ├─ Yangi qo'shish            [persons.create]
   └─ Xarita (GIS)
⚠️ Muammolar
📋 Topshiriqlar
   ├─ Barcha topshiriqlar       [tasks.view_all]
   └─ Mening topshiriqlarim
🚗 Tashriflar
📄 Hujjatlar
🔍 Qidiruv
📈 Analitika
🏆 KPI                          [kpi.view]
🤖 AI Yordamchi                 [ai.use]
🔔 Bildirishnomalar
🗄️ Arxiv                        [archive.view]
🧾 Audit                        [audit.view]
📺 Monitoring Center            [monitoring.view] (yangi oynada ochiladi)
⚙️ Boshqaruv                    [admin bo'limi]
   ├─ Foydalanuvchilar          [users.manage]
   ├─ Rollar va ruxsatlar       [roles.manage]
   ├─ Hududlar va mahallalar    [geo.manage]
   ├─ Lug'atlar                 [dictionaries.manage]
   ├─ Xavfsizlik                [security.manage]
   └─ Tizim sozlamalari         [settings.manage]
👤 Profil
```

Menyu elementi foydalanuvchi permissioniga mos kelmasa umuman render qilinmaydi (himoya UI emas — server ham tekshiradi).

## 3. UI/UX sahifa strukturasi (asosiy ekranlar)

| Sahifa | Tuzilishi |
|---|---|
| **Login** | Username/parol → (2FA yoqilgan bo'lsa) TOTP kod → dashboard. Xato: umumiy xabar (username enumeration yo'q) |
| **Dashboard** | 4 qator: ① KPI stat-kartalar (jami shaxslar, bugungi vazifa/tashrif, muddati o'tganlar) ② grafiklar (trend line, status pie, hudud bar) ③ ro'yxatlar (eng muammoli hududlar, faol/sust xodimlar, risk indikatorlari) ④ oxirgi faoliyat lentasi. Barcha widgetlar scope'ga mos, WS orqali jonli yangilanadi |
| **Shaxslar reestri** | DataTable: rasm, FIO, JShShIR, hudud, status badge, risk badge, mas'ullar, oxirgi tashrif. Ustun sozlash, saqlanadigan filterlar, Excel export tugmasi |
| **Shaxs profili** | Chap: rasm, QR, asosiy ma'lumot, risk ball (gauge). O'ng: tablar — Umumiy / Timeline / Muammolar / Topshiriqlar / Tashriflar / Hujjatlar. Yuqorida: status stepper (workflow bosqichi) |
| **Yangi shaxs (wizard)** | 5 qadam: Shaxsiy → Manzil (Yandex Maps'dan GPS tanlash) → Ijtimoiy holat → Sudlanganlik → Nazorat ma'lumotlari. Har qadam Zod validatsiya, JShShIR dublikat tekshiruvi jonli |
| **Topshiriqlar** | Kanban (status ustunlari) yoki jadval ko'rinishi. Kartada: prioritet, muddat (kechikkan — qizil), bajaruvchi, progress bar |
| **Tashriflar** | Kalendar + jadval. Yangi tashrif: GPS avtomatik, foto/audio/video yuklash, xulosa, keyingi tashrif sanasi |
| **GIS xarita** | To'liq ekran Yandex Maps: marker klasterlari (risk rangi), heatmap rejimi, hudud/status/risk filtri, radius qidiruv, markerdan profilga o'tish |
| **Analitika** | Filter panel (davr, hudud, tur) + ECharts to'plami: pie/bar/line/heatmap, Top-10, davrlar comparison. Har chart PNG/Excel eksport |
| **Monitoring Center** | Qorong'u fullscreen rejim, avtoaylanadigan sahifalar: jonli statistika → xarita → KPI reyting → ogohlantirishlar. Klaviatura/sichqonchasiz ishlaydi |
| **AI Yordamchi** | Chat interfeys + tayyor so'rov shablonlari ("Oxirgi 60 kun tashrif qilinmagan shaxslar"). Natija: jadval + "Hisobot yaratish" tugmasi (Word/PDF) |
| **Audit** | Jadval: vaqt, xodim, harakat, obyekt, IP, qurilma. Yozuvni ochganda old/new qiymatlar diff ko'rinishida |

## 4. State Management

| State turi | Vosita | Misollar |
|---|---|---|
| **Server state** | TanStack Query | Barcha API ma'lumotlari. Query key konvensiya: `['persons', filters]`, `['person', id]`, `['dashboard', scope]` |
| **Auth/session** | Zustand (`auth.store`) + httpOnly cookie | user, permissions, access token holati |
| **UI state** | Zustand (`ui.store`) | sidebar collapse, theme, til |
| **Form state** | React Hook Form + Zod resolver | barcha formalar; Zod sxemalar `packages/shared` dan (backend bilan bitta manba) |
| **URL state** | searchParams | jadval filtrlari/pagination — ulashiladigan havolalar uchun |
| **Real-time** | socket.io → Query invalidation | WS event kelganda tegishli query key invalidate qilinadi |

**TanStack Query qoidalari:** `staleTime` — lug'atlar 1 soat, ro'yxatlar 30s, dashboard 15s; mutatsiyadan keyin aniq invalidation; optimistic update faqat yengil amallar (read/unread, checkbox) uchun.

## 5. Frontend umumiy qoidalar

1. **Server component default** — interaktiv joygina `"use client"`.
2. **API faqat `api-client` orqali** — 401 da avtomatik refresh, ikkinchi 401 da logout.
3. **Permission-aware UI** — `can()` helper + `<Can permission="...">` komponenti.
4. **Dizayn tizimi** — faqat shadcn/ui + Tailwind tokenlar; rasmiy, ixcham davlat-idorasi uslubi; to'liq responsive (planshetgacha), Monitoring Center 4K gacha.
5. **i18n** — barcha matnlar lug'atdan; asosiy til uz-latin.
6. **A11y** — form label/aria, kontrast AA, klaviatura navigatsiyasi.
7. **Og'ir kutubxonalar** (ECharts, Yandex Maps) — dynamic import, faqat kerakli sahifada.
