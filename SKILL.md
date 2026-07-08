---
name: arsitek-kampung
description: "Use when the user wants to discover and document website or project endpoints into a markdown inventory using hybrid code inspection and conditional crawling for verified Next.js targets."
version: 1.2.1
author: Hermes Agent
license: MIT
platforms: [linux, macos]
metadata:
  hermes:
    tags: [endpoints, routes, nextjs, crawling, inventory, markdown, linkfinder]
    related_skills: [codebase-inspection, public-web-maintenance, systematic-debugging]
---

# Arsitek Kampung

> **Bundle note:** ini adalah distribusi single-file untuk installer yang saat ini hanya menyalin `SKILL.md`.
> Konten pendukung dari `references/report-template.md` dan `examples/endpoint-inventory.md` sudah di-embed ke file ini supaya tetap ikut terbawa saat install.

## Overview

Pakai skill ini saat user ingin **mencari, memetakan, dan mendokumentasikan endpoint** dari sebuah website atau project ke dalam **file markdown**.

Filosofi skill ini:
1. **hybrid by default**
2. **code inspection dulu**
3. **crawl hanya kalau perlu**
4. untuk cabang crawl, **browser/web crawling hanya boleh dipakai kalau target sudah terverifikasi Next.js**
5. untuk ekstraksi endpoint dari JavaScript pada cabang crawl Next.js, gunakan **LinkFinder**

Skill ini fokus pada **inventaris endpoint yang rapi, berbukti, dan bisa diaudit ulang**. Ini **bukan** skill pentest, brute-force, fuzzing, atau eksploitasi.

## When to Use

Pakai skill ini kalau user minta hal seperti:
- "cek endpoint project ini"
- "list semua endpoint website ini"
- "tolong petakan route project"
- "buat file md daftar endpoint"
- "cek endpoint ini lalu carikan keluarga route terkait"
- "inventaris semua public page dan API path"

Jangan pakai skill ini untuk:
- vulnerability scanning
- brute-force path guessing
- load testing
- auth bypass testing
- security exploitation workflow

## Input Expectations

Input yang umum:
- path repo lokal
- domain/website live
- endpoint awal sebagai titik masuk
- kombinasi repo + website

Output yang diharapkan:
- file markdown daftar endpoint
- endpoint dikelompokkan per kategori dan confidence
- ada bukti asal endpoint: file path, route definition, JS asset, atau URL live

## Mode Kerja

### 1. Code mode
Pakai saat target utama adalah codebase lokal.

Tool utama:
- `search_files`
- `read_file`
- `terminal` bila perlu untuk ekstraksi stack-aware

Sumber yang diperiksa:
- file route
- controller/handler registration
- helper API client
- frontend `fetch` / `axios` / `XMLHttpRequest`
- webhook callback definition
- generator path/sitemap

### 2. Crawl mode
Pakai saat target utama adalah website live dan user ingin pemetaan surface eksternal.

**Aturan keras:** tool browser/web crawling hanya boleh dipakai kalau target sudah **terverifikasi Next.js**.

Kalau target **bukan** Next.js, jangan pakai browser crawl sebagai jalur utama skill ini. Fallback ke:
- code inspection
- lightweight surface check
- dokumentasi keterbatasan hasil

### 3. Hybrid mode
Ini mode default.

Urutan kerja:
1. deteksi stack
2. scan route dan referensi endpoint dari codebase dulu
3. kalau ada target live dan Next.js sudah terverifikasi, jalankan cabang crawl
4. gabungkan semua hasil ke satu laporan markdown
5. tandai mana yang confirmed, inferred, dan candidate

## Next.js Verification Gate

Sebelum memakai browser/web crawl, verifikasi dulu bahwa target memang Next.js.

### Verifikasi dari codebase
Cari satu atau lebih sinyal berikut:
- dependency `next` di `package.json`
- `next.config.js`, `next.config.mjs`, atau `next.config.ts`
- folder `app/`
- folder `pages/`
- `app/api/**/route.ts|js`
- `pages/api/**`
- pola file seperti `generateStaticParams`, `route.ts`, `layout.tsx`, `page.tsx`

### Verifikasi dari website live
Cari satu atau lebih sinyal berikut:
- `/_next/` di HTML atau asset URL
- `/_next/static/` di script/style asset
- source HTML / script chunk khas Next.js
- pola route/data khas App Router atau Pages Router

Kalau Next.js tidak terverifikasi, tulis jelas di laporan bahwa **cabang crawl dilewati oleh policy skill**.

## Aturan LinkFinder

Untuk cabang crawl, gunakan **LinkFinder** (`linkfinder.py`) dari repo berikut:
- `https://github.com/GerbenJavado/LinkFinder`

### Kenapa LinkFinder dipakai di skill ini
LinkFinder berguna untuk mengekstrak kandidat endpoint dari JavaScript, terutama untuk target Next.js yang banyak menyimpan referensi path/API di bundle client.

Yang dicari biasanya:
- path API yang dipanggil dari frontend
- route-like string dalam bundle JS
- endpoint internal relatif yang tidak kelihatan dari HTML render biasa

### Kapan LinkFinder boleh dipakai
Gunakan LinkFinder hanya jika **semua** kondisi ini terpenuhi:
1. mode saat ini `crawl` atau fase crawl dari `hybrid`
2. target sudah terverifikasi Next.js
3. ada JS asset / bundle / script URL yang relevan untuk dianalisis

### Kapan LinkFinder jangan dipakai
Jangan pakai LinkFinder kalau:
- task murni code-only
- target belum/ tidak terverifikasi Next.js
- daftar endpoint sudah cukup jelas dari file route
- workflow akan jadi terlalu noisy tanpa bukti kuat

### LinkFinder install cepat
Contoh setup minimal tanpa clone penuh repo:

```bash
mkdir -p .runtime/linkfinder
cd .runtime/linkfinder
curl -fsSLo linkfinder.py https://raw.githubusercontent.com/GerbenJavado/LinkFinder/master/linkfinder.py
python3 -m pip install jsbeautifier
python3 linkfinder.py -h
```

### Contoh command LinkFinder

#### 1. Satu file JS remote, output ke CLI
```bash
python3 linkfinder.py -i https://example.com/_next/static/chunks/app.js -o cli
```

#### 2. Filter hanya kandidat `/api/`
```bash
python3 linkfinder.py -i https://example.com/_next/static/chunks/app.js -r ^/api/ -o cli
```

#### 3. Simpan hasil HTML
```bash
python3 linkfinder.py -i https://example.com/_next/static/chunks/app.js -o results.html
```

#### 4. Analisis domain bila memang perlu
```bash
python3 linkfinder.py -i https://example.com -d
```

### Cara membaca hasil LinkFinder
Hasil LinkFinder **jangan langsung dianggap endpoint confirmed-live**.

Perlakukan dulu sebagai:
- `linkfinder-discovered` jika baru muncul dari JS
- naikkan confidence hanya jika cocok dengan route code, asset live, atau bukti lain

## Sumber Endpoint yang Wajib Dicek

### Dari code
- `routes/*.php`
- `app/api/**/route.*`
- `pages/api/**`
- Express/Nest router/controller
- FastAPI/Flask decorator route
- Go mux/router registration
- helper `fetch`, `axios`, `graphql`, `useSWR`
- webhook/callback handler
- generator path, sitemap, atau feed

### Dari surface live
- homepage links
- sitemap.xml
- robots.txt
- menu / footer / static links
- callback/API path yang terekspos di public page
- JS bundle / chunk / route assets

## Aturan Klasifikasi

Setiap endpoint di laporan harus punya label asal / confidence.

### Label confidence yang direkomendasikan
- `confirmed-code` — route/handler jelas ada di code
- `confirmed-live` — teramati langsung dari surface live
- `linkfinder-discovered` — ditemukan dari JS via LinkFinder
- `inferred-dynamic` — route dinamis terduga dari konvensi code
- `unverified` — kandidat ada, tapi belum bisa dipastikan

### Kategori endpoint yang direkomendasikan
- `public-page`
- `api`
- `auth`
- `admin`
- `webhook`
- `static-or-support`

## Format Laporan Markdown

Simpan hasil inventaris ke file markdown.

Struktur minimal yang direkomendasikan:

```md
# Endpoint Inventory

## Summary
- Target: /opt/data/workspace/app + https://example.com
- Mode: hybrid
- Stack: Next.js 15 + API routes
- Next.js verified: yes
- Crawl branch used: yes
- Output generated at: 2026-07-07 17:00 UTC

## Public Pages
- GET /
- GET /about
- GET /pricing

## API Endpoints
- GET /api/users
- POST /api/login
- POST /api/contact

## Dynamic / Inferred Routes
- GET /blog/{slug}
- GET /products/{id}

## LinkFinder Discoveries
- /api/search
- /api/revalidate
- /internal/feed

## Evidence
- Code:
  - app/api/users/route.ts
  - app/blog/[slug]/page.tsx
  - lib/api-client.ts
- Live:
  - https://example.com/sitemap.xml
  - https://example.com/_next/static/chunks/app.js

## Gaps / Unverified
- `/api/revalidate` muncul di JS tapi belum ditemukan route definitifnya
- keluarga route `/products/{id}` inferred dari page dynamic, belum dicrawl semua variannya
```

## One-Shot Recipes

### Recipe A — Repo-only audit
1. deteksi framework
2. cari file route
3. cari referensi `fetch` / `axios` / `api/`
4. satukan hasil
5. tulis `endpoint-inventory.md`

### Recipe B — Hybrid Next.js audit
1. scan codebase dulu
2. verifikasi sinyal Next.js
3. identifikasi JS bundle relevan
4. jalankan LinkFinder pada asset penting
5. dedupe hasil dan cocokkan ke code
6. tulis laporan final dengan label confidence

### Recipe C — Mulai dari satu endpoint
Kalau user memberi satu endpoint seperti `/api/orders`, lakukan:
1. cari endpoint exact match di code
2. cari keluarga path terkait: `/api/order`, `/api/orders/*`, `orders/[id]`
3. cari pemanggilan endpoint itu di frontend/backend
4. masukkan endpoint inti + relasinya ke laporan markdown

## Workflow

1. **Tentukan scope target**
   - repo, website, atau keduanya

2. **Tentukan mode**
   - `code`, `crawl`, atau `hybrid`
   - default ke `hybrid` bila repo dan website sama-sama tersedia

3. **Deteksi stack dulu**
   - jangan langsung crawl
   - cari framework/runtime dan lokasi route utamanya

4. **Scan code dulu**
   - file route
   - handler/controller
   - panggilan endpoint dari frontend/backend

5. **Terapkan Next.js verification gate**
   - kalau lolos dan crawl memang in-scope, lanjut
   - kalau tidak, skip browser/web crawl dan tulis alasannya

6. **Pakai LinkFinder hanya untuk cabang crawl**
   - targetkan JS asset yang relevan
   - kumpulkan kandidat endpoint
   - dedupe hasil
   - simpan sebagai `linkfinder-discovered` dulu

7. **Normalisasi dan gabungkan temuan**
   - satukan route code, URL live, dan hasil JS
   - rapikan slash, query string, dan duplikat

8. **Tulis laporan markdown**
   - wajib ada summary, inventory, evidence, dan gaps

9. **Verifikasi output**
   - path file jelas
   - endpoint sudah dikelompokkan
   - keputusan pakai/skip crawl terdokumentasi

## Common Pitfalls

1. Mengklaim "semua endpoint" padahal baru cek satu sumber.
2. Langsung crawl sebelum verifikasi Next.js.
3. Menganggap hasil LinkFinder otomatis confirmed-live.
4. Melewatkan route dinamis seperti `[slug]`, `[id]`, `[...catchAll]`.
5. Lupa nested router atau framework-specific registration.
6. Tidak memisahkan public page, API, auth, admin, dan webhook.
7. Hanya bikin list mentah tanpa evidence.
8. Menimpa file output tanpa menyebut path laporannya.
9. Mengabaikan referensi endpoint di frontend-heavy app.
10. Membiarkan noise crawl mengalahkan bukti code yang lebih kuat.


## Support Files

Distribusi bundle ini **meng-embed** file pendukung langsung ke dalam `SKILL.md` karena beberapa installer komunitas hanya menyalin satu file.

Sumber canonical tetap ada di repo utama:
- `https://github.com/bothiu/arsitek-kampung`

### Embedded: `references/report-template.md`

```md
# Template Laporan Endpoint Inventory

Gunakan template ini saat menulis output final dari skill **Arsitek Kampung**.

```md
# Endpoint Inventory

## Summary
- Target:
- Mode:
- Stack:
- Next.js verified:
- Crawl branch used:
- Output generated at:

## Public Pages
- GET /

## API Endpoints
- GET /api/example
- POST /api/example

## Dynamic / Inferred Routes
- GET /resource/{id}

## LinkFinder Discoveries
- /api/internal-search

## Evidence
- Code:
  - path/to/route.file
  - path/to/client/file
- Live:
  - https://example.com/sitemap.xml
  - https://example.com/_next/static/chunks/app.js

## Gaps / Unverified
- endpoint kandidat yang belum punya bukti kuat
- route dinamis yang belum diverifikasi penuh
```

## Catatan Pengisian
- Pisahkan endpoint berdasarkan kategori.
- Jangan campur hasil confirmed dengan inferred tanpa label.
- Untuk hasil dari LinkFinder, mulai dari label `linkfinder-discovered`.
- Kalau crawl dilewati karena target bukan Next.js, tulis alasannya di `Summary` atau `Gaps / Unverified`.
```

### Embedded: `examples/endpoint-inventory.md`

```md
# Endpoint Inventory

## Summary
- Target: /opt/data/workspace/demo-next-app + https://demo.example.com
- Mode: hybrid
- Stack: Next.js 15 + Route Handlers
- Next.js verified: yes
- Crawl branch used: yes
- Output generated at: 2026-07-07 17:00 UTC

## Public Pages
- GET /
- GET /about
- GET /pricing
- GET /blog

## API Endpoints
- GET /api/users
- POST /api/login
- POST /api/contact
- GET /api/blog

## Dynamic / Inferred Routes
- GET /blog/{slug}
- GET /products/{id}

## LinkFinder Discoveries
- /api/search
- /api/revalidate
- /internal/feed

## Evidence
- Code:
  - app/api/users/route.ts
  - app/api/login/route.ts
  - app/blog/[slug]/page.tsx
  - lib/api-client.ts
- Live:
  - https://demo.example.com/sitemap.xml
  - https://demo.example.com/_next/static/chunks/app.js

## Gaps / Unverified
- `/api/revalidate` muncul di JS bundle tapi route definitif belum ditemukan.
- `/internal/feed` muncul dari bundle frontend dan belum terkonfirmasi sebagai endpoint live.
```

## Verification Checklist

- [ ] Scope target jelas: repo, website, atau keduanya
- [ ] Mode kerja tertulis: `code`, `crawl`, atau `hybrid`
- [ ] Stack detection selesai sebelum ekstraksi
- [ ] Code route inspection sudah dilakukan
- [ ] Next.js verification selesai sebelum crawl/browser dipakai
- [ ] LinkFinder hanya dipakai saat policy terpenuhi
- [ ] Setiap endpoint punya label confidence/source
- [ ] Laporan markdown ditulis ke output path yang jelas
- [ ] Section `Evidence` berisi file path dan/atau URL
- [ ] Temuan yang belum pasti diberi label `unverified`
