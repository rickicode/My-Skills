---
name: prompt-builder
description: >
  Gunakan skill ini ketika pengguna ingin membuat, menyempurnakan, atau memperbaiki
  prompt untuk AI agent / autopilot yang akan mengerjakan task di repo manapun.
  Trigger phrases: "buatkan prompt", "buat prompt untuk ai", "sempurnakan prompt ini",
  "buat system prompt", "prompt untuk agent", "prompt untuk claude", "bantu buat prompt",
  atau ketika pengguna paste prompt mentah dan minta diperbaiki.
  Skill ini bersifat general — tidak terikat pada satu proyek atau stack tertentu.
version: 1.0.0
author: Claude artifact import via Hermes
license: MIT
metadata:
  hermes:
    tags: [prompt, prompts, agent, system-prompt, brainstorming]
    related_skills: [writing-plans, hermes-agent]
---

Skill ini memandu pembuatan prompt terstruktur untuk AI agent yang bekerja secara
autonomous pada proyek software engineering manapun. Proses dimulai dengan sesi
brainstorming untuk menggali konteks, lalu menghasilkan prompt dalam format XML
terstruktur yang ketat agar AI tidak berasumsi, tidak berhalusinasi, dan tidak
menyatakan selesai sebelum semua kondisi terpenuhi.

---

## Alur Wajib: Brainstorming Dulu, Prompt Kemudian

### Fase 1 — Buka dengan Pertanyaan Konteks (Format Pilihan)

Saat skill ini aktif, JANGAN langsung membuat prompt.
Mulai dengan menyambut dan menanyakan deskripsi task:

> "Silahkan ceritakan task atau masalah yang ingin dibuatkan promptnya.
> Apa yang ingin dikerjakan AI?"

Jika pengguna secara eksplisit hanya meminta pembuatan prompt, JANGAN melakukan inspeksi repo, membaca file, atau menganalisis codebase kecuali pengguna memintanya secara eksplisit. Gali konteks hanya lewat percakapan.

Setelah pengguna menjawab, lanjut ke fase brainstorming dengan format pilihan bertingkat (lihat Fase 2).

---

### Fase 2 — Sesi Brainstorming (Format Pilihan)

Tujuan fase ini: menggali konteks yang cukup agar prompt yang dihasilkan
spesifik, tidak ambigu, dan tidak meninggalkan ruang untuk asumsi.

**Format Wajib — Pilihan Bertingkat:**

Setiap pertanyaan WAJIB menggunakan format pilihan:
```
[No]. [Pertanyaan]
    A. [Jawaban rekomendasi] ✅
    B. [Jawaban alternatif]
    C. [Jawaban alternatif lain]
    D. Tulis sendiri
```

User cukup ketik `1A` untuk memilih rekomendasi, `1D` untuk tulis sendiri, dll.

**Panduan brainstorming:**

- Tanyakan satu pertanyaan per giliran — jangan bombardir dengan banyak pertanyaan sekaligus
- Setiap pertanyaan harus punya minimal 2 opsi + 1 rekomendasi (tanda ✅) + opsi "Tulis sendiri"
- Opsi rekomendasi harus jelas mana yang disarankan berdasarkan konteks yang sudah diketahui
- Minimal 1 putaran tanya-jawab, maksimal tidak terbatas tergantung kompleksitas task
- Hentikan brainstorming saat konteks sudah cukup untuk membuat prompt yang solid

**Contoh Format:**

```
1. Stack teknologi yang digunakan?
    A. Next.js + TypeScript + Tailwind ✅ (recommended untuk dashboard modern)
    B. React + Vite + Tailwind
    C. Astro + React
    D. Tulis sendiri

2. Apakah ada constraint khusus yang tidak boleh diubah?
    A. Tidak ada constraint khusus ✅
    B. Ada — jelaskan di bawah
    D. Tulis sendiri
```

**Area yang perlu digali (sesuaikan dengan relevansi):**

1. **Stack teknologi** — bahasa, framework, runtime yang digunakan
2. **Scope task** — apa saja yang harus dikerjakan, apa yang tidak boleh disentuh
3. **Scale target** — berapa perkiraan jumlah user/data yang akan di-handle. Jangan tanya ini hanya untuk task infrastruktur — untuk full-stack app baru sekalipun, tanyakan di awal. Rekomendasi stack sangat berbeda antara "prototype 10 user" (Next.js fullstack cukup) vs "ribuan user" (backend terpisah, queue/worker, dedicated database). Jika user bilang "ribuan user" atau "scale", jangan rekomendasikan monolithic/Next-only — langsung tawarkan arsitektur terpisah (Go/Hono/Node backend + queue + dedicated DB).
4. **Infrastruktur khusus** — server, database, third-party service, protokol khusus
4. **Kategori / domain** — fitur atau modul apa saja yang terlibat
5. **Constraint** — hal yang tidak boleh diubah, logika yang harus dipertahankan
6. **Output yang diharapkan** — deliverable apa yang harus ada di akhir
7. **Tingkat otonomi** — apakah AI boleh refactor besar, rombak arsitektur UI/UX, atau hanya fix targeted
8. **Mode kerja prompt-only** — jika pengguna hanya meminta prompt, jangan lakukan inspeksi file/repo; cukup gali konteks dari percakapan. TAPI: jika target prompt adalah codebase yang sudah ada (bukan proyek baru), audit ringan TETAP diperlukan untuk mendapatkan path file, nama komponen, dan fitur yang sudah ada — agar prompt akurat dan tidak redundan. Bedakan: "prompt-only untuk konsep baru" (skip audit) vs "prompt untuk redesign existing app" (audit ringan wajib).
9. **Mode eksekusi** — bedakan tegas antara `audit saja` vs `audit + implement/fix`; jangan asumsikan pengguna hanya ingin review jika mereka sebenarnya ingin AI langsung memperbaiki
10. **Bentuk laporan akhir** — tanya apakah pengguna ingin hanya kode jadi, kode + report perubahan, atau summary mismatch/compliance per surface/module
11. **Multi-surface / multi-codebase** — jika ada web + mobile / admin + frontend / beberapa surface lain, gali apakah semuanya harus dicek terpisah dan larang prompt mengasumsikan satu surface identik dengan surface lain

**Cara Respon User:**
- User ketik kombinasi nomor + huruf: `1A`, `2B`, `3D`, dll
- User bisa pilih beberapa sekaligus: `1A 2B 3A`
- User bisa skip pertanyaan yang tidak relevan: `skip`
- User bisa langsung selesai: `done` / `lanjut` / `cukup`

**Pitfall — Jangan Tanya Apa yang Bisa Diverifikasi Sendiri:**
Jika pertanyaan brainstorming bisa dijawab dengan mengecek dokumentasi API, repo code, atau resource teknis yang tersedia — **cek sendiri dulu, baru tanya user**. Jangan tanya user untuk hal yang bisa diverifikasi dengan tools. Contoh buruk: "Apakah TMDB support bahasa Indonesia?" → seharusnya cek TMDB docs langsung, lalu konfirmasi ke user. User bukan documentation lookup service — mereka datang untuk brainstorming konteks, bukan untuk menggali teknis yang bisa di-search.

**Pitfall — Scope Expansion:**
Saat brainstorming, scope bisa berkembang (misal: user mulai dari "mobile only" lalu bilang "oh untuk bahasa juga web"). Tangkap perluasan scope secara natural, update task_definitions untuk mencakup semua platform yang disebut, dan jangan paksa user untuk "commit" ke scope awal. Flexibilitas brainstorming lebih penting daripada rigiditas.

**Pitfall — Jangan Asumsikan Repo dari Memory:**
Saat user bilang "prompt builder" atau "buat prompt untuk [X]" yang bersifat general,
JANGAN langsung assume target repo dari memory/context sebelumnya. Memory mungkin
punya list project user (AxonRouter, Honcho, dll), tapi user mungkin mau prompt
**general-purpose** yang bisa dipakai di project manapun.

Cara handle yang benar:
1. Tanya dulu: prompt ini untuk project spesifik atau general-purpose?
2. Jika general: JANGAN tanya "untuk repo mana?" — langsung gali scope (framework, audit type, dll)
3. Jika project spesifik: baru tanya repo-nya

Contoh salah: User bilang "prompt builder, mau buat prompt buat analisis kode siap production"
→ Agent langsung tanya "Untuk repo mana? AxonRouter, Honcho, atau AlbianX?" (SALAH — user belum sebut repo)
→ Seharusnya: "Ini prompt general-purpose untuk semua Node.js projects, atau untuk repo spesifik?"

Jika user jawab general, brainstorming fokus ke: stack/framework target, scope audit, output format.
JANGAN pernah tanya repo-specific detail untuk general-purpose prompt.

**Pitfall — Re-analysis After Code Push:**
Ketika user bilang mereka sudah push kode baru atau minta analisis ulang repo:
1. `git pull` untuk fetch perubahan terbaru
2. `git diff HEAD~1 --stat` untuk melihat file apa saja yang berubah
3. `git log --oneline -3` untuk melihat commit terbaru
4. Baca file yang berubah untuk memahami konteks baru
JANGAN hanya `git pull` dan bilang "up to date" tanpa memeriksa apakah ada perubahan relevan.
Gunakan parallel tool calls (clone/pull + baca file) untuk efisiensi.

**Pitfall — Numbering Discipline in Brainstorming:**
Saat menjalankan sesi brainstorming bernomor, JAGA nomor pertanyaan tetap berurutan dan jangan melompat. Jika user menunda satu pertanyaan untuk diskusi samping, lanjutkan kembali dengan nomor yang benar setelah diskusi selesai. Jika terlanjur salah nomor dan user mengoreksi, akui singkat, reset ke nomor yang benar, lalu lanjut tanpa defensif atau mengulang konteks berlebihan. Nomor pertanyaan adalah navigasi user — kesalahan nomor bikin sesi terasa kacau.

**Pitfall — Capture Compliance/Policy Constraints as First-Class Prompt Rules:**
Jika user menyebut constraint operasional/regulasi (contoh: WhatsApp Business API chat window, opt-in notification rules, anti-spam, approved template requirement), jangan perlakukan sebagai detail samping. Lock constraint itu ke `<system_constraints>` dan task terkait. Untuk fitur notifikasi/analisis, bedakan tegas antara on-demand, opt-in scheduled, dan unsolicited outbound; jangan otomatis mengirim insight/report hanya karena sistem bisa membuatnya.

**Pitfall — Over-asking (Paling Penting):**
User sering memberikan sinyal bahwa mereka ingin lanjut tanpa menjawab semua pertanyaan. Sinyal-sinyal ini HARUS ditangkap dan brainstorming DIHENTIKAN:
- User ketik `done` / `lanjut` / `cukup` / `skip`
- User ketik `1A 2A 3A` (pilih semua rekomendasi sekaligus)
- User ketik `lanjut buat aja` / `langsung aja` / `udah cukup`
- User ketik `proceed` / `go ahead` / `just make it`
- User ketik `yang lain terserah` / `default aja`
- Jawaban singkat seperti `oke`, `ya`, `skip`
- **User memberikan 5+ requirement spesifik di pesan pertama mereka** — ini adalah sinyal kuat bahwa mereka sudah tahu persis apa yang mau. JANGAN mulai dari pertanyaan dasar (tech stack, scope). Langsung skip ke variabel yang benar-benar belum diketahui (repo URL, autonomy level, data availability). Proaktif cari info tech stack dari PRD/TDD/docs di workspaces sebelum tanya user.
Ketika sinyal ini muncul, HENTIKAN brainstorming. Lanjut ke Fase 3 (tawarkan pilihan) atau langsung ke Fase 4 (generate prompt) dengan reasonable defaults berdasarkan konteks yang sudah terkumpul dari audit repo. Jangan memaksa user menjawab pertanyaan yang sudah bisa di-infer dari konteks yang ada.

**Pitfall — Batched Questions vs One-at-a-Time (Read User Signal):**
Default skill instructs one question per turn ("Tanyakan satu pertanyaan per giliran"). Namun beberapa user PREFER multiple questions sekaligus — mereka ingin diskusi konteks dalam satu blok, bukan bolak-balik satu-pertanyaan. Sinyal yang menunjukkan ini:
- User ketik: "diskusi dong terus tanya nya jangan satu satu, sekaligus saja"
- User ketik: "sekalian aja" / "tanya sekaligus"
- User merasa pertanyaan satu-satu terlalu lambat

**Cara handle:** Ketika user meminta batched questions:
1. Identifikasi SEMUA area yang perlu digali sekaligus
2. Tampilkan semua pertanyaan sekaligus dengan format pilihan yang sama
3. User bisa jawab sekaligus (`1A 2B 3A`) atau pilih yang relevan aja
4. Jika user meminta batched DAN sekaligus minta konfirmasi arsitektur, gabungkan: tampilkan summary arsitektur + pertanyaan terbuka dalam satu blok

Ini bukan override mutlak — beberapa user tetap prefer satu-satu. Yang penting: baca sinyal early dan adapt.

**Pitfall — Supplementary Prompts:**
Jika user meminta prompt tambahan terkait topik yang sama ("sekalian buatkan prompt untuk X", "tambah prompt untuk Y"), JANGAN restart brainstorming dari awal. Gunakan konteks dari prompt pertama sebagai basis. Cukup tanyakan: "Untuk prompt tambahan ini, ada spesifikasi khusus atau langsung saya generate berdasarkan konteks prompt pertama?" — lalu generate dengan format yang sama.

**Pitfall — Verify User Claims Before Adding Tasks:**
Saat user mendeskripsikan fitur yang "harus ada" atau "perlu ditambahkan", VERIFIKASI dulu di codebase sebelum memasukkannya sebagai task dalam prompt. Seringkali fitur yang user deskripsikan sudah ada — hanya perlu dikonfirmasi, bukan dibuat ulang. Contoh dari session HIJISTREAM:
- User minta "tambah IMDB rating di poster" → ternyata `ContentCard.jsx` sudah ada `ratingBadge` (star + rating di pojok kanan atas)
- User minta "genre tags harus bisa diklik" → ternyata `DetailHero.jsx` sudah ada `handleGenrePress` yang navigate ke `/genre/[id]`
- User minta "More Like This grid ke bawah" → ternyata sudah `flexWrap: 'wrap'` (vertical grid)
Tanpa verifikasi ini, prompt akan berisi task redundant yang membuat AI agent mubazir mengerjakan hal yang sudah benar. Selalu `read_file` komponen yang relevan, lalu presentasikan ke user: "Ini sudah ada/belum" sebelum finalize task list.

**Pitfall — Deep Repo Audit untuk Multi-Repo Tasks:**
Jika task melibatkan multiple repos (integrate A ke B), WAJIB audit kedua repo SEBELUM generate prompt. Baca struktur directory, file utama, arsitektur, dan pola yang ada. Prompt yang dihasilkan HARUS memuat detail spesifik dari kedua repo (nama file, path, ukuran, pola yang ada) — bukan asumsi generik. AI agent yang menerima prompt harus bisa langsung bekerja tanpa perlu audit sendiri. Contoh: jika repo A punya `studio/flow_graph/executor.py` (75KB), prompt harus sebutkan file itu spesifik, bukan hanya "flow executor".

**Pitfall — Hardcoded Paths dalam Prompt:**
JANGAN pernah hardcode path apapun dalam prompt (misal: `/workspaces/X`, `./repo-name/`, `/home/user/project`). Gunakan placeholder `<REPO_NAME>` dan tambahkan `<setup_instructions>` block yang menginstruksikan AI agent untuk:
1. Clone repo terlebih dahulu
2. Detect actual path via `ls`/`pwd`/`find` setelah clone
3. Replace placeholder dengan detected path sebelum eksekusi

Contoh `<setup_instructions>`:
```xml
<setup_instructions>
  BEFORE executing any task, clone both repositories:
    1. git clone [REPO_URL_1]
    2. git clone [REPO_URL_2]
  After cloning, DETECT the actual paths by inspecting the filesystem.
  Do NOT assume any path — always verify via ls/pwd after clone.
  All file references below use placeholders like <REPO_A> and <REPO_B>.
  Replace them with actual detected paths at execution time.
</setup_instructions>
```

**Pitfall — Reference Repo (Bukan Integration Target):**
Kadang user memberikan URL repo sebagai referensi desain/fitur (bukan untuk di-integrate). Contoh: "cek repo OmniRoute, saya mau buat fitur serupa di AxonRouter." Ini BUKAN prompt-only mode dan BUKAN multi-repo integration — ini adalah konteks desain. Saat user memberikan reference repo URL:
1. Clone repo tersebut (atau akses via GitHub API) sebagai bagian dari context gathering
2. Audit struktur, komponen UI, pattern, dan API yang relevan dengan task
3. Sertakan detail spesifik dari reference repo di prompt (file paths, component names, API patterns)
4. Tapi JANGAN copy-paste kode reference — prompt harus menghasilkan implementasi yang mengikuti style/target repo, bukan reference repo
Jangan tanya user "mau saya clone repo referensi-nya?" — langsung lakukan saat user memberikan URL. Gunakan `git clone --depth 1` untuk kecepatan karena kita hanya butuh membaca file, bukan full git history.

**Pitfall — Parallel Multi-Repo Audit:**
Saat ada reference repo + target repo, JANGAN serial audit (clone A, baca A, baru clone B, baca B).
Gunakan parallel tool calls: clone repos secara bersamaan, lalu baca file kunci dari kedua repo
dalam satu blok tool call. Ini menghemat waktu secara signifikan untuk repositori besar.
Contoh: `git clone` repo A dan repo B dalam satu `execute_code` block, lalu `read_file` dari
kedua repo secara paralel di call berikutnya.

**Pitfall — Architecture Diagram as Brainstorming Input:**
Saat brainstorming, user mungkin langsung memberikan **arsitektur lengkap dalam bentuk diagram** (ASCII art, Mermaid, atau deskripsi visual) alih-alih menjawab pertanyaan satu per satu. Ini BUKAN jawaban singkat — ini adalah design input yang sangat berharga. Cara handle:
1. Baca dan pahami diagram secara menyeluruh — identifikasi semua komponen, alur data, dan dependency
2. Konfirmasi pemahaman kamu dengan merangkum kembali diagram dalam bentuk teks (component list + flow)
3. Tanyakan detail yang tidak jelas dari diagram (naming, tech choices, constraints)
4. JANGAN abaikan diagram dan kembali ke pertanyaan awal — diagram adalah konteks utama
5. Sertakan detail diagram di `<task_context>` dan `<architecture>` blocks dalam prompt

**Pitfall — Naming/Branding Discussion:**
Kadang user ingin mendiskusikan nama app/produk sebelum generate prompt. Ini valid dan harus didukung:
1. Usulkan beberapa nama dengan penjelasan singkat (maksimal 5-7 opsi)
2. Tunggu user pilih atau beri ide sendiri
3. Setelah nama fix, gunakan konsisten di seluruh prompt
4. JANGAN anggap nama sebagai hal sepele — nama produk yang baik membantu AI agent memahami konteks domain

**Pitfall — Security Sections Harus Practical, Bukan Paranoid:**
Saat generate prompt yang termasuk security audit (production readiness, code review, dll),
JANGAN buat security task yang terlalu ketat/paranoid. User cenderung lebih suka pendekatan
practical: fix yang jelas berbahaya, document yang borderline.

Panduan untuk security task dalam prompt:
- **Fix**: hardcoded secrets, critical npm audit vulns, stack traces exposed to clients, missing .gitignore
- **Quick check (not blocking)**: CORS, basic security headers, obvious injection patterns
- **Document as recommendations** (jangan block audit): rate limiting, CSP, input sanitization semua endpoints, prototype pollution, open redirect
- **Skip entirely** untuk prototype/internal tools: full penetration testing patterns, WAF config, compliance checklists

Kata kunci untuk objective task: "Fix obvious security issues" BUKAN "Find and fix ALL security vulnerabilities".
Tone: "yang penting nggak irresponsible untuk ship" BUKAN "zero tolerance untuk semua vulnerability".

**Pitfall — Rule-Based Prompt Generation:**
Saat generate prompt untuk AI agent, JANGAN buat prompt yang terlalu detail sampai mendikte implementasi spesifik. User lebih suka pendekatan rule-based:
- **Buat**: Aturan, constraint, dan batasan yang harus dipatuhi
- **Jangan buat**: Langkah-langkah implementasi spesifik (nama file, nama variabel, urutan kode)
- **Biarkan**: AI agent menganalisis codebase dan memutuskan implementasi terbaik sendiri

Contoh BAIK: "Setiap AI role harus punya provider sendiri yang bisa di-assign admin"
Contoh BURUK: "Buat tabel `ai_role_provider` dengan kolom `role_name VARCHAR(50)`, `provider_id UUID REFERENCES admin_provider(id)`, dst."

Prinsipnya: prompt = aturan + constraint, bukan blueprint implementasi. AI agent yang menerima prompt harus bisa analisis codebase sendiri dan memilih pendekatan terbaik.

**Pitfall — Scope Accumulation During Brainstorming:**
Saat brainstorming, scope sering berkembang secara bertahap (misal: user mulai dari "Provider Topology"
lalu menambah "route rename", "tunnel redesign", "service installer fix"). Setiap kali user
menambah scope, TUNGGU sampai brainstorming selesai, lalu presentasikan konsolidasi scope
sebelum generate prompt. Format presentasi:
- Ringkasan semua task yang akan di-cover (tabel singkat)
- Konfirmasi: "Ini semua sudah lengkap, atau ada yang mau ditambah?"
Ini mencegah prompt yang incomplete karena scope terus berubah saat generating.

**Pitfall — No Ambiguity in Final Prompt Tech Choices (CRITICAL):**
Saat generate prompt, JANGAN biarkan "or" alternatives di stack/tech choices ketika user sudah
memilih spesifik. User frustasi kalau prompt yang dihasilkan masih bilang "Hono or Gin" padahal
mereka sudah pilih Hono, atau "React or Svelte" padahal mereka pilih React/TanStack.

Contoh SALAH (prompt yang dihasilkan):
```xml
<backend language="Go" framework="Hono or Gin" />
<admin framework="React or Svelte" />
```

Contoh BENAR:
```xml
<backend language="Go" framework="Hono (Go port)" description="Use github.com/hugomd/hono-go" />
<admin framework="React + TanStack Router" description="With shadcn/ui components" />
```

Rule: Setelah user memilih spesifik dari options, LOCK pilihan itu di prompt.
Tulis nama framework/library yang PERSIS, sertakan npm package atau import path jika memungkinkan.
Prompt yang ambiguous = AI agent waste waktu deciding what user already decided.

**Pitfall — Scope Transformation (Beyond Accumulation):**
Terkadang scope tidak hanya bertambah — tapi BERUBAH FUNDAMENTAL. Contoh: user mulai dengan
"buat blog pake Astro" (seperti CMS biasa), tapi setelah brainstorming berkembang, ternyata
yang diminta adalah "AI auto-blogging platform fully automated dengan multi-agent pipeline".

Sinyal scope transformation:
- Arsitektur berubah (dari "manual CMS" → "auto-blogging platform")
- Role components berubah (dari "admin nulis" → "AI generate, admin monitor")
- Flow berubah (dari "CRUD manual" → "keyword → AI pipeline → auto-publish")

**Cara handle:**
1. SAAT scope mulai berubah, JANGAN langsung generate prompt
2. Re-summarize dengan arsitektur baru: "Sepertinya scope-nya udah berkembang cukup jauh dari blog biasa jadi auto-blogging platform. Ini arsitektur baru yang gw pahami..."
3. Tampilkan perbandingan: "Yang awalnya X, sekarang jadi Y. Konfirmasi arsitektur baru dulu sebelum lanjut."
4. Update SEMUA task_definitions untuk refleksi scope baru — jangan mix old scope + new scope
5. Validasi: semua tasks harus konsisten dengan arsitektur terbaru, bukan campuran lama-baru

Ini lebih kritis dari scope accumulation karena prompt yang mix arsitektur lama+baru akan confict.

**Referensi — Pola Multi-Agent:**
Jika task melibatkan arsitektur multi-agent (Generator-Judge, Pipeline, Specialist Routing, dll),
lihat `references/multi-agent-patterns.md` untuk pola yang sudah teruji beserta XML representation-nya.

---

**Pitfall — Embedding Reference Materials in Prompt (CRITICAL):**

JANGAN embed referensi kode, API docs, atau source code langsung ke dalam prompt XML.
Prompt yang terlalu panjang karena embedding referensi akan:
- Memakan context window AI agent
- Membuat prompt sulit di-maintain
- Membuat user frustrasi karena prompt terlalu besar

**Cara yang PALING DISUKAI — Private GitHub Repo:**

Saat prompt membutuhkan referensi eksternal (API docs, source code, data files),
buat **private GitHub repo** berisi semua file referensi, lalu arahkan AI agent untuk clone:

```xml
<reference_downloads>
  <instruction>Clone the reference repository FIRST before starting any task. All files are the source of truth.</instruction>
  <repo>https://github.com/OWNER/repo-name</repo>
  <commands>
    git clone https://github.com/OWNER/repo-name.git references
  </commands>
  <files>
    <file name="reference-api.md" description="Complete API reference with endpoints, code examples" />
    <file name="reference-source.py" description="VERIFIED working implementation - use as source of truth" />
    <file name="reference-data.json" description="Data schema and sample data" />
  </files>
  <critical_note>DO NOT start coding without reading these files first.</critical_note>
</reference_downloads>
```

Dan tambahkan Rule wajib di `<system_constraints>`:

```xml
<rule id="N" priority="CRITICAL">
  MANDATORY REFERENCE DOWNLOAD - BEFORE STARTING ANY TASK:
  Clone the reference repository FIRST:
    git clone https://github.com/OWNER/repo-name.git references
  After cloning, READ each file in the references/ directory completely before writing any code.
  The reference files are the source of truth - DO NOT guess or assume.
</rule>
```

**Cara yang BENAR (alternative — paste.rs):**

Jika GitHub repo tidak memungkinkan, gunakan paste.rs:

```xml
<reference_downloads>
  <commands>
    mkdir -p references && cd references
    curl -sL https://paste.rs/XXXXX > reference-api.md
    curl -sL https://paste.rs/YYYYY > reference-source.py
  </commands>
</reference_downloads>
```

**Cara yang SALAH (user akan komplain):**
```xml
<!-- JANGAN lakukan ini - prompt jadi terlalu panjang -->
<reference>
  <content>
    (paste 18KB of API documentation langsung di sini)
  </content>
</reference>
```

**Workflow lengkap (preferred):**
1. Buat private GitHub repo
2. Upload semua file referensi ke repo
3. Sertakan repo URL di `<reference_downloads>`
4. Tambahkan rule wajib clone di `<system_constraints>`
5. AI agent clone repo, baca file, lalu kerja

**Workflow alternative (paste.rs):**
1. Upload referensi ke paste.rs
2. Sertakan curl commands di `<reference_downloads>`
3. AI agent download sendiri

**Naming rule:**
- JANGAN pakai abbreviasi yang tidak jelas (misal: "SCCM" untuk Social Campaign Manager)
- Gunakan nama yang descriptive dan mudah dipahami
- User pernah komplain: "sccm itu apa" — abbreviasi membingungkan

**Pitfall — Reference Existing Prompts:**
Jika target project sudah punya AI system prompt di codebase (misal: `src/lib/ai/prompts/generator.ts`, `judge.ts`, `brainstorming.ts`, atau file prompt lainnya), JANGAN buat prompt baru dari nol. Sebaliknya:
1. Baca dan pahami prompt yang sudah ada di codebase
2. Sertakan reference ke prompt existing di generated prompt (path file, nama export)
3. Jika prompt existing perlu perbaikan, buat catatan di generated prompt sebagai improvement suggestion
4. JANGAN replace atau duplicate prompt yang sudah ada kecuali user meminta secara eksplisit

Ini memastikan generated prompt menghormati arsitektur AI yang sudah ada di project dan tidak membuat redundansi.

### Fase 3 — Tawarkan Pilihan (Format Pilihan)

Di akhir setiap putaran brainstorming, tawarkan pilihan dengan format yang sama:

```
Konteks sudah cukup untuk mulai membuat prompt?
    A. Ya, langsung buat prompt ✅
    B. Lanjut diskusi dulu
    C. Saya mau tambah scope
```

Jika pengguna memilih A → lanjut ke Fase 3.5 (target repo), lalu Fase 4.
Jika pengguna memilih B → kembali ke Fase 2.
Jika pengguna memilih C → tanyakan scope tambahan, lalu kembali ke Fase 2.

---

### Fase 3.5 — Wajib Tanya Target Repo

Sebelum generate PRD + TDD + Prompt, WAJIB tanya apakah output prompt ini untuk:

```
Repo target untuk proyek ini?
    A. Buat repo GitHub baru untuk proyek ini ✅
    B. Implement ke repo GitHub existing — saya akan kasih URL repo + branch
    C. Prompt-only dulu, repo nanti
    D. Tulis sendiri
```

**Jika user pilih A — repo baru:**
- Tanya nama repo, visibility (`private` default kecuali user minta public), dan owner/org jika belum jelas.
- Di prompt final, sertakan `<repo_strategy mode="new_repo">` yang menginstruksikan AI agent untuk create/clone repo baru sebelum implementasi.
- Di prompt final, WAJIB sertakan instruksi untuk **copy/curl Code Audit Pack dari repo My-Skills** ke repo baru di path `skills/code-audit/` sebelum implementasi dimulai. Jangan suruh AI menulis ulang/mengarang isi skill dari nol.
- Sertakan rule bahwa setelah implementasi selesai dan pushed, AI agent WAJIB menjalankan audit ulang terhadap repo hasil push menggunakan PRD + TDD + Prompt + injected Code Audit Pack sebagai source of truth.

**Jika user pilih B — repo existing:**
- Minta URL repo + branch target jika belum diberikan.
- Jika repo bisa diakses, lakukan audit ringan terlebih dahulu untuk mendeteksi stack, struktur, file penting, dan prompt existing.
- Di prompt final, sertakan `<repo_strategy mode="existing_repo">` dengan repo URL, branch, dan instruksi `git pull` sebelum kerja.
- Di prompt final, WAJIB sertakan instruksi untuk **copy/curl Code Audit Pack dari repo My-Skills** ke repo existing di path `skills/code-audit/` sebelum implementasi dimulai. Jika folder sudah ada, update/overwrite file pack dari source terbaru. Jangan suruh AI menulis ulang/mengarang isi skill dari nol.
- Sertakan rule bahwa setelah perubahan di-commit/push, AI agent WAJIB audit ulang repo terbaru (`git pull`, inspect changed files, run verification) menggunakan PRD + TDD + Prompt + injected Code Audit Pack untuk memastikan implementasi sesuai.

**Jika user pilih C — prompt-only:**
- Tetap sertakan `<repo_strategy mode="deferred">` agar AI agent tahu repo belum ditentukan.
- Prompt final harus meminta AI agent menunggu repo URL atau membuat repo baru saat eksekusi, bukan mengasumsikan path lokal.

**Audit ulang setelah push adalah mandatory untuk mode A dan B:**
Tambahkan rule di `<system_constraints>` dan `<completion_gate>`:
- Setelah kode di-push ke GitHub, clone/pull fresh copy dari remote.
- Pastikan `skills/code-audit/` ikut ter-commit dan berisi semua file Code Audit Pack.
- Audit ulang fresh copy terhadap PRD, TDD, Prompt XML, dan `skills/code-audit/PROMPT.md` + `skills/code-audit/references/*.md`.
- Buat compliance matrix: `requirement | implementation evidence | status | gaps`.
- Jika ada mismatch, bug, missing requirement, mock/dummy data yang tidak diizinkan, missing audit pack file, atau verification failure: fix, push ulang, lalu ulangi audit dari fresh clone/pull.
- Task baru boleh COMPLETE jika audit ulang fresh remote copy menyatakan semua requirement sesuai, Code Audit Pack lengkap, dan semua verification command exit 0.

---

### Fase 4 — Generate PRD + TDD + Prompt

Setelah brainstorming selesai, WAJIB buat **3 artefak konsisten** dari konteks yang sama:

1. **PRD (Product Requirements Document)** — menjelaskan produk, user, fitur, scope, non-goals, UX flow, acceptance criteria, dan release criteria.
2. **TDD (Technical Design Document)** — menjelaskan arsitektur teknis, stack final yang sudah dipilih, data model, API surface, worker/job flow, integrasi eksternal, security practical, observability, dan verification plan.
3. **Agent Prompt XML** — prompt eksekusi untuk AI agent, berisi task_context, system_constraints, task_definitions, dan execution_order.

**PRD dan TDD adalah source of truth untuk prompt.** Jangan generate prompt yang task/stack/scope-nya berbeda dari PRD/TDD. Semua pilihan yang sudah di-lock saat brainstorming harus muncul konsisten di ketiga artefak.

**Urutan wajib:**
1. Draft PRD dulu
2. Draft TDD berdasarkan PRD
3. Draft Agent Prompt XML berdasarkan PRD + TDD
4. Self-check konsistensi PRD ↔ TDD ↔ Prompt sebelum delivery

**Consistency check wajib sebelum kirim/upload:**
- Stack final sama persis di PRD, TDD, dan Prompt — tidak ada alternatif `or` kalau user sudah memilih spesifik
- Scope task sama, tidak ada fitur yang hilang atau tiba-tiba bertambah
- Non-goals/constraints dari PRD masuk ke TDD dan system_constraints prompt
- API/DB/worker concepts di TDD direfleksikan di task_definitions prompt
- Verification plan di TDD selaras dengan Rule 10 di prompt
- Tidak ada placeholder kosong seperti `[tbd]`, `[sesuaikan]`, `[nama]`
- Tidak ada hardcoded local path; gunakan placeholder repo/path yang aman
- `repo_strategy` sudah terisi sesuai jawaban user: `new_repo`, `existing_repo`, atau `deferred`
- Untuk `new_repo`/`existing_repo`, prompt berisi instruksi inject Code Audit Pack ke `skills/code-audit/` sebelum implementasi
- Untuk `new_repo`/`existing_repo`, post-push fresh remote audit rule dan completion gate sudah masuk ke Prompt XML
- Completion gate memverifikasi `skills/code-audit/` lengkap di fresh remote copy

Jika salah satu check gagal, revisi artefak terkait dulu sebelum tampilkan ringkasan ke user.

Isi setiap bagian berdasarkan konteks yang sudah digali — jangan ada bagian kosong
atau diisi dengan placeholder.

Jika output final panjang, JANGAN langsung upload ke paste.rs. Ikuti alur ini:

1. **Tampilkan ringkasan isi PRD + TDD + prompt** di chat (fitur, arsitektur, task/domain, jumlah rules, verification gates)
2. **Tanyakan konfirmasi** ke user: "PRD + TDD + Prompt sudah jadi, ada yang mau ditambah/diubah? Atau langsung upload ke paste.rs?"
3. **Tunggu jawaban user** — jika user minta revisi, update PRD/TDD/prompt yang terdampak, jalankan consistency check lagi, lalu ulangi dari langkah 1
4. **Setelah konfirmasi**, baru upload ke `https://paste.rs/` dengan raw body (BUKAN multipart form, karena `paste.rs` akan menampilkan header multipart sebagai bagian dari isi paste). Contoh: `curl --data-binary @file https paste.rs`
5. Kirim link paste ke user
6. **WAJIB sertakan instruksi eksekusi** — format response setelah upload harus seperti ini:

```
**Link Prompt:** https://paste.rs/<id>

**Cara eksekusi — AI agent tinggal ikuti:**
Read this instruction to complete the task: https://paste.rs/<id>

Download the instruction file first:
curl -sL https://paste.rs/<id> > /tmp/prompt.md

Then read the full instruction from /tmp/prompt.md and execute ALL tasks sequentially as defined in <execution_order>, following every rule in <system_constraints>.

**Yang dihasilkan:** [ringkasan singkat output yang diharapkan]
```

Format ini memastikan AI agent yang menerima prompt tidak perlu bertanya "apa yang harus gw lakukan?" — semuanya sudah ada dalam satu blok response.

Hanya kirim inline penuh (tanpa paste.rs) jika prompt pendek ATAU pengguna secara eksplisit memintanya.

---

## Format Prompt yang Dihasilkan

### Blok 1: `<task_context>`

Berisi metadata proyek dan konteks teknis yang relevan:

```xml
<task_context>
  <project name="[nama proyek]" />
  <scope>[deskripsi singkat scope task]</scope>
  <execution_mode>autonomous</execution_mode>
  <repo_strategy mode="new_repo|existing_repo|deferred">
    <decision_source>Chosen during Fase 3.5 target repo question.</decision_source>
    <target_repo url="[repo url if known]" branch="[branch]" visibility="private|public" />
    <code_audit_pack required="true" target_path="skills/code-audit">
      <source_repo>https://github.com/rickicode/My-Skills</source_repo>
      <source_path>skills/software-development/code-audit</source_path>
      <required_files>
        <file>README.md</file>
        <file>SKILL.md</file>
        <file>PROMPT.md</file>
        <file>references/audit-dimensions.md</file>
        <file>references/fix-patterns.md</file>
        <file>references/severity-guide.md</file>
      </required_files>
      <install_commands>
        mkdir -p skills/code-audit/references
        BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"
        curl -fsSL "$BASE/README.md" -o skills/code-audit/README.md
        curl -fsSL "$BASE/SKILL.md" -o skills/code-audit/SKILL.md
        curl -fsSL "$BASE/PROMPT.md" -o skills/code-audit/PROMPT.md
        curl -fsSL "$BASE/references/audit-dimensions.md" -o skills/code-audit/references/audit-dimensions.md
        curl -fsSL "$BASE/references/fix-patterns.md" -o skills/code-audit/references/fix-patterns.md
        curl -fsSL "$BASE/references/severity-guide.md" -o skills/code-audit/references/severity-guide.md
      </install_commands>
      <instruction>
        For new_repo and existing_repo modes, copy or curl every required file from My-Skills into the target repository at skills/code-audit/ before implementation starts. Do NOT generate these files manually and do NOT summarize them. Commit and push these copied files with the project changes. Do not rely on external memory only.
      </instruction>
    </code_audit_pack>
    <post_push_audit required="true">
      After implementation is pushed, clone or pull a fresh copy from remote, verify skills/code-audit/ contains every required Code Audit Pack file, then audit the repo against PRD + TDD + Prompt XML + skills/code-audit/PROMPT.md + skills/code-audit/references/*.md. Produce a compliance matrix, fix any mismatch, push again, and repeat until fully compliant.
    </post_push_audit>
  </repo_strategy>
  <repos>
    <!-- Sertakan hanya jika task melibatkan multiple repos atau reference repo -->
    <repo name="[nama]" url="[url]" access="private|public" branch="[branch]" purpose="target|reference" />
    <repo name="[nama]" url="[url]" access="private|public" branch="[branch]" purpose="target|reference" />
  </repos>
  <stack>
    <!-- Sesuaikan dengan stack yang ditemukan saat brainstorming -->
    <backend language="[bahasa]" />
    <frontend framework="[framework]" description="[deskripsi]" />
    <!-- Tambah layer lain jika ada: admin, mobile, dll -->
  </stack>
  <infrastructure>
    <!-- Hanya isi jika ada infrastruktur khusus yang perlu AI ketahui -->
    <!-- Contoh: CHR MikroTik, self-hosted runner, protokol khusus, dll -->
  </infrastructure>
  <domains>
    <!-- Daftar semua domain / kategori yang relevan dengan task ini -->
    <!-- Sertakan deskripsi singkat tiap domain agar AI tidak salah interpretasi -->
  </domains>
</task_context>
```

---

### Blok 2: `<system_constraints>`

Berisi aturan ketat yang wajib dipatuhi AI. Selalu sertakan semua rules berikut.
Tambahkan rules ekstra jika ada constraint spesifik dari brainstorming.

```xml
<system_constraints>

  <rule id="1" priority="CRITICAL">
    NEVER declare the task as done, complete, finished, or any equivalent
    until ALL completion gates in this block are fully and explicitly satisfied.
    Partial completion is NOT completion.
    Assumed completion is NOT completion.
    Stated completion without proof is NOT completion.
  </rule>

  <rule id="2" priority="CRITICAL">
    NEVER assume, infer, or hallucinate the state of ANY entity — including but not limited to:
      - files, folders, modules, imports, interfaces, structs
      - functions, methods, variables, types, generics
      - configs, environment variables, dependencies
      - API responses, database states, third-party service states
      - test results, build outputs, lint results
    If you have not explicitly read, executed, or observed it in the current context window,
    it does not exist. Treat all unverified state as UNKNOWN, not as assumed.
  </rule>

  <rule id="3" priority="CRITICAL">
    NEVER generate, modify, or delete anything based on what you EXPECT to be there.
    Observation must precede action. Always in this order:
      1. Read / Inspect
      2. Reason
      3. Act
      4. Verify result
    Skipping any step in this sequence is a violation.
  </rule>

  <rule id="4" priority="CRITICAL">
    NEVER fabricate command output, test results, build logs, or any verification artifact.
    If a command was not run, its output does not exist.
    Do NOT paraphrase, summarize, truncate, or simulate output.
    Only raw, verbatim stdout/stderr is accepted as proof.
  </rule>

  <rule id="5" priority="CRITICAL">
    NEVER skip, merge, abbreviate, or parallelize verification steps.
    Each step must be executed sequentially, in full, independently.
    Speed and token efficiency are NOT valid reasons to shortcut verification.
  </rule>

  <rule id="6" priority="CRITICAL">
    NEVER reuse a previous verification result as proof for the current state.
    Every declaration of completion requires a fresh, current verification run.
    Stale results are invalid.
  </rule>

  <rule id="7" priority="CRITICAL">
    NEVER resolve a conflict, ambiguity, or blocker by guessing.
    If resolution requires information not present in context:
      - State what is missing explicitly
      - State why it cannot be inferred
      - Request only the minimum information needed to proceed
  </rule>

  <rule id="8" priority="CRITICAL">
    After all work is complete, perform exactly 3 independent self-review cycles.
    Each cycle is fully isolated — no inherited state, assumptions, or conclusions from prior cycles.
    If any issue is found at any point during any cycle:
      - Fix the issue immediately
      - Restart ALL 3 cycles from cycle 1
    Format each cycle strictly as:
      [REVIEW CYCLE N/3]
      → Scope checked
      → Issues found (list explicitly, or state "none")
      → Fixes applied (list explicitly, or state "none")
      → Cycle status: PASSED / RESTARTING FROM CYCLE 1
  </rule>

  <rule id="9" priority="CRITICAL">
    The existence of ANY bug, error, regression, or unintended behavior —
    regardless of severity, scope, or perceived impact —
    classifies the entire task as INCOMPLETE.
    There is no such thing as an acceptable or ignorable bug.
  </rule>

  <rule id="10" priority="CRITICAL">
    Run the full verification suite appropriate to the project's language and tooling.
    Detect and adapt to the actual stack in use — do NOT hardcode assumptions about tools.
    Verification must cover at minimum:
      - Tests (unit, integration, or equivalent)
      - Type checking (if applicable to the stack)
      - Build / compile
      - Linting / static analysis (if configured)

    <!-- Saat generate prompt, ganti blok ini dengan perintah spesifik sesuai stack -->
    <!-- Contoh untuk Go + Astro + React: -->
    <!--
      [BACKEND — Go]
        go build ./...    → exit 0, zero errors
        go vet ./...      → exit 0, zero warnings
        go test ./...     → exit 0, all tests pass
        staticcheck ./... → exit 0, zero findings (if configured)

      [FRONTEND — Astro]
        astro check       → exit 0, zero type errors
        astro build       → exit 0, zero errors, zero warnings

      [FRONTEND — React]
        tsc --noEmit      → exit 0, zero type errors
        npm run build     → exit 0, zero errors, zero warnings
    -->

    Acceptance threshold across ALL stacks:
      errors   → 0 (absolute zero tolerance)
      warnings → 0 (absolute zero tolerance)

    If any threshold is violated in any stack:
      - Resolve ALL issues in that stack
      - Re-run the FULL verification suite for ALL stacks from the beginning
      - Do NOT selectively re-run only the affected stack
  </rule>

  <rule id="11" priority="CRITICAL">
    <!-- Sertakan rule ini hanya jika proyek multi-stack dengan API contract lintas layer -->
    Cross-stack contract consistency must be explicitly verified.
    Every API contract defined in the backend must exactly match what is consumed
    in all frontend layers — field names, types, response shapes, error formats,
    and HTTP status codes must be identical.
    Any mismatch is a bug and must be fixed before proceeding.
  </rule>

  <rule id="12" priority="CRITICAL">
    CODE AUDIT PACK INJECTION IS MANDATORY when repo_strategy is "new_repo" or "existing_repo".
    Before implementing project tasks, create or update the target repository folder `skills/code-audit/` by copying/curling every required file from:
      https://github.com/rickicode/My-Skills/tree/main/skills/software-development/code-audit
    Do NOT write these files manually and do NOT create placeholders; the exact upstream file contents must be copied into the target repo.
    Required files:
      - skills/code-audit/README.md
      - skills/code-audit/SKILL.md
      - skills/code-audit/PROMPT.md
      - skills/code-audit/references/audit-dimensions.md
      - skills/code-audit/references/fix-patterns.md
      - skills/code-audit/references/severity-guide.md
    Read `skills/code-audit/PROMPT.md` and all `skills/code-audit/references/*.md` before auditing or fixing.
    Do not continue if any required file is missing or empty.
  </rule>

  <rule id="13" priority="CRITICAL">
    POST-PUSH REMOTE AUDIT IS MANDATORY when repo_strategy is "new_repo" or "existing_repo".
    After pushing implementation changes to GitHub:
      1. Clone or pull a fresh copy from the remote repository.
      2. Verify `skills/code-audit/` exists in the fresh remote copy and all required Code Audit Pack files are non-empty.
      3. Audit the fresh remote copy against the PRD, TDD, this Prompt XML, and the injected Code Audit Pack (`skills/code-audit/PROMPT.md` + `skills/code-audit/references/*.md`).
      4. Produce a compliance matrix: requirement | implementation evidence | status | gaps.
      5. Run the full verification suite from the fresh remote copy.
      6. If any mismatch, missing requirement, missing audit pack file, unauthorized mock/dummy data, bug, or verification failure exists: fix it, push again, then repeat this rule from step 1.
    Completion is forbidden until the fresh remote audit proves the pushed code matches the PRD, TDD, Prompt XML, and injected Code Audit Pack.
  </rule>

  <!-- Tambahkan rules ekstra di sini jika ada constraint spesifik dari brainstorming -->
  <!-- Contoh: "jangan ubah logika bisnis X", "jangan sentuh file Y", dll -->

  <completion_gate>
    Task status may ONLY transition to COMPLETE when ALL of the following conditions
    are explicitly and verifiably true — not assumed, not inferred, not estimated:

      ✓ All assigned work has been executed and verified
      ✓ review_cycles_completed == 3
      ✓ No unresolved findings exist across all 3 cycles
      ✓ bugs_found == 0
      ✓ All verification commands across all stacks → EXIT 0, errors == 0, warnings == 0
      ✓ Cross-stack API contracts verified and consistent (if applicable)
      ✓ Code Audit Pack injected into `skills/code-audit/` with all required files when repo_strategy is new_repo or existing_repo
      ✓ Post-push fresh remote audit completed and compliant when repo_strategy is new_repo or existing_repo
      ✓ Compliance matrix shows every PRD, TDD, Prompt XML, and Code Audit Pack requirement as implemented with evidence
      ✓ Full verbatim output of every verification command displayed

    <!-- Tambah kondisi spesifik dari task jika ada -->
    <!-- Contoh: ✓ Documentation generated and complete -->
    <!-- Contoh: ✓ SKILL.md created at correct path -->

    Failure of ANY single condition above resets task status to INCOMPLETE.
    Completion cannot be declared by inference. It must be proven.
    There are no exceptions. There are no edge cases. There is no override.
  </completion_gate>

</system_constraints>
```

---

### Blok 3: `<task_definitions>`

Setiap task punya `id`, `domain`, `objective`, dan langkah atau requirements yang eksplisit.
Pecah menjadi sub_task jika domain besar atau mencakup backend + UI + script sekaligus.

**Panduan menulis task:**
- Mulai setiap audit task dengan: "Read all [domain]-related files without exception"
- Sebutkan komponen UI yang harus ada secara spesifik, bukan hanya "buat UI-nya"
- Untuk infrastruktur khusus, sertakan `<context>` yang menjelaskan constraint teknis
- Jika ada domain yang diketahui dari brainstorming, sebutkan semua — jangan biarkan
  AI menebak mana yang perlu dicek
- Jika ada constraint "jangan ubah X", taruh di dalam task yang relevan sekaligus
  di system_constraints

```xml
<task_definitions>

  <task id="1" domain="[nama domain]">
    <objective>
      [Tujuan yang jelas dan spesifik]
    </objective>
    <steps>
      <step>[Langkah 1]</step>
      <step>[Langkah 2]</step>
      ...
    </steps>
  </task>

  <task id="2" domain="[nama domain]">
    <objective>[...]</objective>
    <sub_task id="2.1" name="[nama]">
      <context>[Constraint teknis khusus jika ada]</context>
      <requirements>
        <requirement>[...]</requirement>
      </requirements>
    </sub_task>
    <sub_task id="2.2" name="[nama]">
      ...
    </sub_task>
  </task>

  <!-- Lanjutkan untuk semua domain yang ditemukan saat brainstorming -->

</task_definitions>
```

---

### Blok 4: `<execution_order>`

```xml
<execution_order>
  Execute tasks sequentially: 1 → 2 → 3 → ... → N
  Do NOT proceed to the next task if the current task has unresolved findings.
  All tasks must reach a verified PASSED state before moving forward.
</execution_order>
```

---

## Referensi Verification Commands per Stack

Gunakan tabel ini saat mengisi Rule 10. Sesuaikan dengan stack yang terdeteksi.

| Stack       | Commands                                              |
|-------------|-------------------------------------------------------|
| Go          | `go build ./...`, `go vet ./...`, `go test ./...`, `staticcheck ./...`, `golangci-lint run` |
| Node.js     | `npm test`, `npm run build`, `eslint`                 |
| Deno        | `deno check`, `deno lint`, `deno test`                |
| Bun         | `bun test`, `bun run build`                           |
| TypeScript  | `tsc --noEmit`, `npm run build`, `eslint`             |
| React       | `tsc --noEmit`, `npm run build`, `eslint`             |
| Astro       | `astro check`, `astro build`, `eslint`                |
| Python      | `pytest`, `mypy` / `pyright`, `ruff`, `black --check`, `isort --check` |
| Rust        | `cargo build`, `cargo test`, `cargo clippy`           |
| Java        | `mvn compile`, `mvn test`, `mvn package`              |
| Kotlin      | `gradle build`, `gradle test`                         |
| Swift       | `swift build`, `swift test`                           |
| Dart/Flutter| `dart analyze`, `flutter test`, `dart format --set-exit-if-changed .` |
| PHP         | `composer install`, `phpunit`, `phpstan`              |
| Ruby        | `bundle exec rspec`, `rubocop`                        |
| Elixir      | `mix compile --warnings-as-errors`, `mix test`, `mix format --check-formatted` |
| Haskell     | `stack build`, `stack test`, `hlint`                  |
| Lua         | `luacheck .`, `busted`                                |

Jika stack tidak ada di tabel, gunakan tooling standar yang relevan untuk bahasa tersebut.

---

## Delivery Output

- **JANGAN upload ke paste service tanpa konfirmasi user terlebih dahulu.**
- **Langkah wajib sebelum upload:**
  1. Tampilkan ringkasan isi PRD + TDD + Prompt XML di chat (fitur, arsitektur, task/domain, jumlah rules, verification gates)
  2. Tanyakan: "Ada yang mau ditambah/diubah? Atau langsung upload?"
  3. Tunggu jawaban user, baru lakukan upload
- Jika artefak pendek, boleh kirim langsung inline di chat
- Jika artefak panjang DAN sudah dikonfirmasi user, upload ke paste service lalu kirim link
- Jangan membanjiri chat dengan blok PRD/TDD/Prompt yang sangat panjang kecuali pengguna secara eksplisit meminta versi inline penuh

**Format Respon Wajib Setelah Upload/Kirim PRD, TDD, & Prompt:**

Setelah berhasil (baik dikirim inline maupun via paste.rs), response wajib memuat:
- Link (atau teks) PRD
- Link (atau teks) TDD
- Link (atau teks) Prompt XML
- Instruksi cara eksekusi prompt untuk AI agent.

Contoh untuk format paste.rs:
```
**Artefak Proyek:**
- PRD: https://paste.rs/<id_prd>
- TDD: https://paste.rs/<id_tdd>
- Prompt XML: https://paste.rs/<id_prompt>

**Cara eksekusi — AI agent tinggal ikuti:**
Read this PRD, TDD, and instruction prompt to [deskripsi singkat task]:
PRD: https://paste.rs/<id_prd>
TDD: https://paste.rs/<id_tdd>
Prompt: https://paste.rs/<id_prompt>

Download the prompt file, read all instructions inside, use the PRD and TDD as the technical and product truth, and execute every task sequentially as defined in <execution_order>, following every rule in <system_constraints>.

**Yang dihasilkan:** [ringkasan singkat output yang diharapkan]
```

Format ini memastikan AI agent yang menerima prompt punya akses penuh ke product requirements (PRD) dan technical design (TDD), lalu tahu persis apa yang harus dieksekusi lewat Prompt XML.

Hanya kirim inline penuh (tanpa paste.rs) jika teks pendek ATAU pengguna secara eksplisit memintanya.

**Template — SaaS Starter Kit Prompt:** `templates/saas-starter-kit-prompt.md` — known-good template for Go/Echo + React/TanStack starter kit prompts. Includes plugin architecture patterns, verification commands, and completion gate template.

**Paste Service — Primary: paste.rs**
- Command: `curl --data-binary @file https://paste.rs/`
- JANGAN pakai multipart form
- Return format: `https://paste.rs/<id>` (JANGAN tambahkan .txt suffix)

**Paste Service — Fallback: dpaste.org**
- Jika paste.rs down atau rate-limited
- Command: `curl -X POST -d "content=@file&syntax=auto&expiry_days=7" https://dpaste.org/api/`
- Return format: `https://dpaste.org/<id>/`

**Paste Service — Fallback: GitHub Gist**
- Jika paste.rs dan dpaste.org tidak tersedia
- Command: `gh gist create file.md --public --desc "prompt output"`
- Return format: `https://gist.github.com/<user>/<id>`

**Batch upload:** Jika user meminta multiple prompts sekaligus, generate semua prompt dulu, tampilkan ringkasan gabungan, lalu upload satu per satu. Tampilkan semua link di akhir.

---

## Post-Prompt Actions — Repo Scaffolding

Setelah prompt di-generate dan di-upload, user sering meminta "Buat repo nya juga" — mereka ingin
GitHub repo yang sudah di-scaffold dengan struktur directory, config files, dan boilerplate agar
AI agent yang menerima prompt bisa langsung mulai implementasi tanpa setup dari nol.

**Kapan lakukan:**
- User bilang "buat repo nya juga" / "create the repo" / "setup repo"
- Prompt sudah final dan uploaded

**Apa yang harus di-scaffold (minimal):**
1. GitHub repo (private, via `gh repo create`)
2. Root files: `README.md`, `.gitignore`, `.env.example`, `docker-compose.yml`
3. **Code Audit Pack injected ke `skills/code-audit/`** berisi `README.md`, `SKILL.md`, `PROMPT.md`, dan semua `references/*.md` dari `rickicode/My-Skills/skills/software-development/code-audit`
4. Directory structure sesuai stack di prompt (`/backend`, `/frontend`, `/admin`, dll)
5. Config files per stack: `go.mod`, `package.json`, `astro.config.mjs`, `vite.config.ts`, dll
6. Entry point files: `main.go`, `index.astro`, `main.tsx` (minimal, cukup untuk build jalan)
7. Database migration files (jika ada schema di prompt)
8. Dockerfiles per service
9. `.env.example` dengan semua variabel yang disebut di prompt

**Yang TIDAK perlu di-scaffold:**
- Implementasi lengkap (itu tugas AI agent menerima prompt)
- Business logic, API handlers, UI components detail
- Test files

**Commit message pattern:**
```
feat: initial project scaffolding

Monorepo structure for [Project Name]:

[Stack 1]:
- Key files created

[Stack 2]:
- Key files created

Docker Compose setup for local development
.env.example with all required variables documented
Code Audit Pack injected under skills/code-audit/ for post-push audit/re-audit workflows
```

**Flow:**
1. Prompt uploaded → deliver link ke user
2. User minta repo → scaffold + push
3. Inject/update Code Audit Pack ke `skills/code-audit/`, commit, dan push bersama scaffold
4. Sertakan repo URL + prompt link bersamaan di final response

---

## Hal yang Tidak Boleh Ada dalam Prompt yang Dihasilkan

- Instruksi yang memungkinkan AI berasumsi tentang state yang belum diverifikasi
- Completion criteria yang bisa dipenuhi tanpa bukti nyata (verbatim output)
- Task scope yang ambigu seperti "perbaiki semua yang perlu diperbaiki" tanpa
  menyebut domain spesifik
- Perintah yang membolehkan AI skip verifikasi dengan alasan efisiensi atau kecepatan
- Placeholder yang tidak diisi: "[nama]", "[tbd]", "[sesuaikan]" tidak boleh ada
  di prompt final yang dikirim ke AI
- Rules yang saling bertentangan satu sama lain
- **Hardcoded paths** (misal: `/workspaces/X`, `./repo-name/`) — gunakan placeholder
  seperti `<REPO_NAME>` dan instruksikan AI untuk clone + detect path sendiri
  setelah clone. AI agent harus verify path via `ls`/`pwd`/`find` sebelum operasi.

---

## Multi-Prompt Orchestration

Untuk task kompleks yang menghasilkan multiple prompts, buat **Master Orchestrator** — prompt yang mendownload dan mengeksekusi prompt-prompt lain secara berurutan.

**Kapan menggunakan:**
- Task punya 2+ fase terpisah (implementasi + hardening/testing)
- User meminta prompt untuk area yang berbeda (fitur + infrastruktur)
- Prompt pertama terlalu panjang untuk satu file

**Kapan TIDAK menggunakan:**
- Task bisa selesai dalam satu prompt (< 5 tasks)
- Tidak ada dependency antar prompt
- User tidak meminta multiple prompts

**Contoh konkret — Task: Tambahkan payment gateway ke e-commerce**

Prompt 1 (`prompt-payment-implementation.md`):
```
Goal: Implement Midtrans payment integration
Tasks:
- Add payment service under src/services/payment/
- Create /api/payment/create endpoint
- Add payment form component in frontend
Verification: go build, go test, npm run build
```

Prompt 2 (`prompt-payment-hardening.md`):
```
Goal: Harden payment flow for production
Tasks:
- Add idempotency key handling
- Implement retry logic for failed webhooks
- Add integration tests with Midtrans sandbox
- Security audit: no secrets in logs
Verification: go test ./..., npm run build, npm run lint
```

Master Orchestrator (`prompt-master-payment.md`):
```markdown
# Master Orchestrator: Payment Gateway Integration

## System Constraints
- Execute prompts sequentially
- Do NOT proceed if previous prompt failed
- Each prompt's constraints are PRIMARY

## Execution Order
1. Download prompt-payment-implementation.md to project root
2. Download prompt-payment-hardening.md to project root
3. Execute prompt-payment-implementation.md (read file → run all tasks)
4. Execute prompt-payment-hardening.md (read file → run all tasks)
5. Final verification: run ALL commands from both prompts
```

**Aturan Master Orchestrator:**
- Prompt 1 & 2 di-download ke root repo (main folder), BUKAN /tmp
- Master prompt berisi sendiri system constraints + completion gates
- Tapi constraint di individual prompts adalah PRIMARY — master hanya orchestrator
- Jangan duplicate constraints dari prompt individual ke master
- Task di master hanya "read prompt file → execute all tasks inside"
- Master harus include final verification yang menjalankan SEMUA command dari SEMUA prompt

---

## Web UI Integration Prompts

Jika task melibatkan pembuatan halaman web / form / UI di dalam aplikasi web yang sudah ada:

**Elemen wajib dalam prompt:**
1. **Route/path** halaman baru (e.g., `/gopay-plus`)
2. **Component structure** — komponen apa saja yang harus dibuat/modify
3. **Form fields** (jika ada form) — tiap field: name, type, label, placeholder, required, validasi
4. **State management** — bagaimana state di-handle (local state, global store, server state)
5. **Error handling** — bagaimana error ditampilkan ke user
6. **API integration** — endpoint yang di-call (method, path, request/response shape)
7. **Responsive design** — behavior di mobile vs desktop
8. **Accessibility** — minimum: proper labels, keyboard navigation, ARIA attributes

**Pattern — Form + Background Processing:**
```
Form submit → POST /api/xxx → returns job_id + status
Background task processes...
Client polls GET /api/xxx/status/{job_id}
Status changes: idle → processing → waiting_input → success/failed
```

**Pattern — Interactive Input (OTP, multi-step):**
```
Status = waiting_otp → JS shows OTP input field
User enters OTP → POST /api/xxx/otp {job_id, otp_code}
Background task continues...
```

**Pattern — Simple CRUD Page:**
```
List page → GET /api/xxx → render table
Create page → form → POST /api/xxx
Edit page → GET /api/xxx/:id → form prefill → PUT /api/xxx/:id
Delete → confirm dialog → DELETE /api/xxx/:id
```

**Pattern — Dashboard with Real-time Updates:**
```
Initial load → GET /api/xxx/dashboard
Subscribe → WebSocket / SSE endpoint
Updates push to UI without refresh
```

**Pitfall — CLI vs Web:**
Jika user awalnya describe sistem sebagai CLI/service, tapi aplikasi target adalah web-based, TANYAK konfirmasi: "Sistem ini mau diintegrasikan sebagai CLI tool atau halaman web di aplikasi yang sudah ada?" Jangan asumsikan CLI padahal aplikasi web.

**Pitfall — Framework Mismatch:**
Jangan asumsikan React untuk semua web project. Cek dulu frontend framework yang digunakan (React, Vue, Svelte, Astro, Next.js, Nuxt, dll). Setiap framework punya pola component dan state management yang berbeda.

**Pitfall — AI Agent Framework Reliability (Balancing Framework Fit vs AI Stability):**
Ketika merekomendasikan stack frontend, pertimbangkan seberapa stabil AI coding agent menghasilkan kode untuk framework tersebut. React/Next.js/TanStack memiliki coverage training data terluas — AI coding agents jarang error untuk import path, hooks, state management, dan routing patterns. Svelte/SvelteKit (terutama Svelte 5 runes) lebih sering memicu: import confusion antara Svelte 4 vs 5, wrong rune syntax, missing lifecycle API, dan routing convention errors. SolidJS, Astro (untuk complex app bukan content site), dan Vue juga lebih rawan hallucination dibanding React.

Cara handle dalam brainstorming:
1. Jika user tidak menyebut preferensi framework spesifik → rekomendasikan React/TanStack atau Next.js sebagai default paling aman
2. Jika user menyebut Svelte/Solid/Astro → respect pilihan mereka, tapi beri tahu risk: "SvelteKit memang ringan, tapi AI lebih sering salah untuk runes/API baru. Siap kalau perlu benerin hasil AI-nya."
3. Jika prioritas user adalah "AI agent harus build sangat stabil dengan minimal error" → jangan rekomendasikan Svelte/SvelteKit meskipun user bilang ringan — jelaskan tradeoff-nya.
4. Prompt final harus tetap memakai framework yang user pilih — jangan override. Tapi task definitions di prompt bisa include hint: "If you encounter Svelte 5 rune errors, refer to the official Svelte 5 migration guide."

**Pitfall — read_file Line Number Corruption:**
JANGAN gunakan output `read_file()` atau `execute_code` + `read_file()` langsung sebagai input ke `write_file()`. Output `read_file` memiliki format `     1|content` (line numbers + pipe). Jika ditulis ulang, line numbers menjadi bagian dari file content dan corrupt file. Solusi:
- Untuk membaca file: gunakan `terminal("cat file")` atau `terminal("head -N file")`
- Untuk modify file: gunakan `patch()` atau `terminal("sed/awk")`
- Jika harus read+write: strip line numbers dulu dengan string processing sebelum write
- Contoh corrupt: `     1|     1|     1|<task_context>` (triple line numbers tertulis di file)
