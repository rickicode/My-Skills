---
name: code-audit
description: >
  Gunakan skill ini setiap kali ada permintaan untuk mengaudit, mereview, memvalidasi,
  atau memastikan kualitas kode — termasuk frasa seperti "audit kode", "cek bug",
  "production ready", "pastikan tidak ada mock", "validasi implementasi", "cek
  kelengkapan fitur", "tidak boleh ada bug", "review dulu", "cek dulu kodenya",
  "pastikan sudah bener", "semua harus real", "perbaiki semua bug", "autofix",
  atau kalimat sejenis. Skill ini menangani tiga skenario: proyek baru dari PRD,
  rewrite dari referensi kode, dan proyek tanpa dokumen apapun (AI memetakan sendiri).
  Skill ini membedakan audit-only dan audit+fix: review/cek hanya melaporkan, sedangkan fix/autofix/perbaiki menjalankan perbaikan, verifikasi, dan re-audit.
---

# Code Audit & Autofix Skill

Skill ini menjalankan workflow audit kode dengan dua mode eksekusi:

1. **AUDIT_ONLY** — untuk permintaan review/cek/audit saja. Tidak mengubah file.
2. **AUDIT_AND_FIX** — untuk permintaan fix/autofix/perbaiki/production-ready. Menjalankan audit → autofix → verifikasi → re-audit.

Jangan mengedit file jika user hanya meminta review/cek/audit tanpa meminta fix.

---

## FASE 0 — Deteksi Konteks

Baca konteks percakapan dan tentukan mode. Gunakan decision tree ini:

```
Ada kode yang bisa diakses?
├── TIDAK → Minta kode. Stop sampai ada.
└── YA ↓

Ada indikasi ini adalah rewrite / refactor dari kode lain?
├── YA + referensi kode lama TIDAK ada → TANYA SATU KALI:
│   "Ada kode referensi / kode lama yang jadi acuan?"
│   ├── Ada → MODE B
│   └── Tidak ada → MODE C
├── YA + referensi SUDAH ada → MODE B
└── TIDAK (proyek dari nol) ↓

Ada PRD / spesifikasi yang diberikan?
├── YA → MODE A
└── TIDAK → TANYA SATU KALI:
    "Ada PRD atau dokumen spesifikasi?"
    ├── Ada → MODE A
    └── Tidak ada → MODE C
```

**Aturan tanya**: Maksimal satu pertanyaan. Jika kode ada dan tidak ada indikasi
rewrite maupun PRD → langsung MODE C tanpa tanya. Jangan tanya PRD lagi jika konteks
sudah cukup untuk Mode C.

---

## FASE 1 — Persiapan per Mode

### MODE A — PRD-Based
1. Parse semua fitur dari PRD → buat daftar eksplisit
2. Map tiap fitur ke kode: `✅ Ada | ❌ Missing | ⚠️ Partial | 🔄 Berbeda dari spec`
3. Fitur `❌ Missing` = CRITICAL otomatis

### MODE B — Reference-Based
1. Ekstrak semua fungsi/endpoint/komponen dari kode referensi
2. Cek padanannya di kode baru: `✅ Equivalent | ❌ Missing | 🔄 Changed | ✨ Added`
3. Fungsi kritis `❌ Missing` = CRITICAL otomatis
4. Fungsi `🔄 Changed` → investigasi: bug atau perubahan disengaja?

### MODE C — Self-Discovery
Lakukan tiga langkah ini sebelum audit. Tulis hasilnya secara eksplisit di output.

**C1 — Rekonstruksi Intent** (jawab dari kode, bukan asumsi):
- Aplikasi ini melakukan apa?
- Siapa penggunanya?
- Data apa yang paling kritis?
- Apa alur utama (happy path)-nya?

**C2 — Inventarisasi Kode**:
```
INVENTARIS:
- [file › fungsi]: complete | suspect | incomplete — [alasan]
```
Tandai `suspect` jika ada: return default tanpa logika, TODO/FIXME,
data hardcoded yang harusnya dinamis, fungsi dipanggil tapi tidak ada implementasi.

**C3 — Rekonstruksi "PRD Implisit"**:
Dari kode yang ada, derive fitur yang seharusnya ada secara logis.
Contoh: ada `createOrder` → seharusnya ada `cancelOrder`, `getOrderHistory`.
```
FITUR YANG DIHARAPKAN:
- [fitur]: ✅ Ada di [file] | ❌ Missing | ⚠️ Partial
```

---

## FASE 2 — Audit per Dimensi

Jalankan semua dimensi. Baca `references/audit-dimensions.md` untuk kriteria detail.

**Wajib per temuan**: nama file + nama fungsi + kutipan snippet kode (maks 5 baris)
**Label confidence**: `[PASTI]` / `[DUGAAN KUAT]` / `[PERLU CEK]`

| # | Dimensi | Kritis? |
|---|---------|---------|
| D1 | Kelengkapan Implementasi — tidak ada stub/mock/TODO | ✅ Ya |
| D2 | Ketepatan Algoritma & Logika Bisnis | ✅ Ya |
| D3 | Penanganan Edge Case | ✅ Ya |
| D4 | Error Handling & Resiliensi | ✅ Ya |
| D5 | Integritas Data & Atomisitas | ✅ Ya |
| D6 | Keamanan | ✅ Ya |
| D7 | Performa & Efisiensi | ⚠️ Kontekstual |
| D8 | Keterbacaan & Maintainability | ❌ Advisory |
| D9 | Kesesuaian PRD/Referensi | ✅ Ya (Mode A/B) |

---

## FASE 2.5 — Mode Eksekusi

Tentukan execution mode dari wording user:

- **AUDIT_ONLY**: user bilang audit, review, cek, analisa, cari bug, validasi — tanpa minta edit. Stop setelah laporan audit + action items.
- **AUDIT_AND_FIX**: user bilang fix, autofix, perbaiki, langsung benerin, production ready, tidak boleh ada bug. Lanjut ke autofix, verification, dan re-audit.

Jika ambiguous, default ke AUDIT_ONLY agar tidak mengubah file tanpa izin.

---

## FASE 3 — Laporan Audit

Cetak laporan dengan format WAJIB berikut sebelum memulai autofix:

```
═══════════════════════════════════════════════════
                 LAPORAN AUDIT KODE
═══════════════════════════════════════════════════
MODE      : [A – PRD-Based / B – Reference-Based / C – Self-Discovery]
SCOPE     : [daftar file/modul]
REFERENSI : [nama PRD / nama file referensi / "Direkonstruksi dari kode"]

VERDICT   : [PRODUCTION READY ✅ / CONDITIONAL ⚠️ / BLOCKED ❌]
RISK LEVEL: [CRITICAL / HIGH / MEDIUM / LOW]

───────────────────────────────────────────────────
RINGKASAN EKSEKUTIF
───────────────────────────────────────────────────
[3–5 kalimat: kondisi kode, apa yang baik, apa bermasalah, akan diperbaiki sekarang]

───────────────────────────────────────────────────
TABEL TEMUAN
───────────────────────────────────────────────────
ID  | Sev      | File › Fungsi        | Masalah              | Fix Plan
----|----------|---------------------|----------------------|----------
F01 | CRITICAL | [file] › [fungsi]   | [masalah + label]    | [apa yang akan diubah]
F02 | HIGH     | ...                 | ...                  | ...

───────────────────────────────────────────────────
STATUS PER DIMENSI
───────────────────────────────────────────────────
D1 Kelengkapan Implementasi  : ✅ PASS / ❌ FAIL / ⚠️ WARNING / N/A → [catatan]
D2 Algoritma & Logika        : [status] → [catatan]
D3 Edge Case                 : [status] → [catatan]
D4 Error Handling            : [status] → [catatan]
D5 Integritas Data           : [status] → [catatan]
D6 Keamanan                  : [status] → [catatan]
D7 Performa                  : [status] → [catatan]
D8 Maintainability           : [status] → [catatan]
D9 Kesesuaian PRD/Referensi  : [status] → [catatan] ← N/A jika Mode C

───────────────────────────────────────────────────
RINGKASAN AUTOFIX
───────────────────────────────────────────────────
Total temuan  : [X]
Akan difix    : CRITICAL ([X]) + HIGH ([X]) + MEDIUM ([X]) + LOW ([X])
Skip (manual) : [PERLU CEK] ([X]) — perlu keputusan manusia
Memulai fix...
═══════════════════════════════════════════════════
```

---

## FASE 4 — Autofix (hanya AUDIT_AND_FIX)

Jika execution mode = AUDIT_ONLY, stop setelah laporan audit dan jangan edit file.
Jika execution mode = AUDIT_AND_FIX, eksekusi fix setelah laporan.

### Aturan Autofix:

**Yang WAJIB difix otomatis di mode AUDIT_AND_FIX (SEMUA severity — CRITICAL, HIGH, MEDIUM, LOW yang [PASTI] / [DUGAAN KUAT] dan aman):**
- Stub/mock/placeholder → implementasi nyata
- Try/catch missing → tambahkan dengan error handling yang proper
- Logika bisnis salah → perbaiki kalkulasi/kondisi
- Edge case → tambahkan guard clause
- N+1 query → refactor ke batch/join
- Secret hardcoded → pindah ke env variable dengan placeholder `process.env.NAMA_VAR`
- SQL injection / XSS → ganti ke parameterized query / textContent
- Operasi non-atomik → bungkus dalam transaksi
- Auth/ownership check missing → tambahkan middleware/guard
- Nama variabel tidak deskriptif → rename (LOW)
- Magic number → ekstrak ke konstanta bernama (LOW)
- Fungsi terlalu panjang → pecah ke sub-fungsi (LOW)
- Kode comment-out tanpa penjelasan → hapus atau beri komentar alasan (LOW)

**Yang TIDAK difix otomatis (butuh keputusan manusia):**
- Temuan `[PERLU CEK]` — AI tidak cukup yakin, catat sebagai catatan manual
- Perubahan yang memengaruhi arsitektur besar (misal: ganti seluruh library)
- Fitur `❌ Missing` di Mode A yang belum ada sama sekali — AI bisa buat skeleton
  implementasi tapi tandai dengan `// AUTOFIX: skeleton — perlu review logika bisnis`

### Cara Menulis Fix:

Untuk setiap temuan yang difix, gunakan format:

```
──────────────────────────────────────
FIX F01 [CRITICAL] — [file] › [fungsi]
Masalah : [deskripsi singkat]
Tindakan: [apa yang diubah]
──────────────────────────────────────
[kode yang sudah diperbaiki — lengkap, bukan diff]
```

**Aturan penulisan kode fix:**
- Tulis **seluruh fungsi/blok** yang difix, bukan hanya baris yang berubah
- Kode harus langsung bisa dipakai (copy-paste ready)
- Jika fix satu fungsi memengaruhi fungsi lain, fix keduanya
- Pertahankan naming convention dan style yang ada di kode asli
- Tambahkan komentar `// AUTOFIX: [alasan]` di baris yang diubah untuk traceability

### Setelah Semua Fix:

Cetak ringkasan penutup:

```
═══════════════════════════════════════════════════
               AUTOFIX SELESAI
═══════════════════════════════════════════════════
Difix     : [X] temuan (CRITICAL: X, HIGH: X, MEDIUM: X, LOW: X)
Manual    : [X] temuan [PERLU CEK] — perlu keputusan manusia

CATATAN MANUAL (jika ada):
- [FM01] [file › fungsi]: [mengapa tidak bisa difix otomatis + rekomendasi]

Status akhir: [SIAP DEPLOY ✅ / PERLU REVIEW MANUAL ⚠️]
═══════════════════════════════════════════════════
```

---

## FASE 5 — Verification dan Re-audit

Untuk AUDIT_AND_FIX:
1. Jalankan test/typecheck/build/lint sesuai tooling repo nyata.
2. Tampilkan command, exit code, dan ringkasan output nyata.
3. Jika gagal, fix penyebabnya lalu ulangi verifikasi.
4. Re-audit file/domain yang berubah menggunakan D1-D9.
5. Status akhir hanya boleh `SIAP DEPLOY` jika tidak ada CRITICAL/HIGH unresolved dan verification pass.

---

## Aturan Anti-Halusinasi

1. **Jangan nilai file yang tidak ada** → tulis `N/A – file tidak tersedia`
2. **Setiap temuan harus ada bukti kode** → kutip snippet, dilarang "sepertinya ada masalah"
3. **Label confidence wajib**: `[PASTI]` / `[DUGAAN KUAT]` / `[PERLU CEK]`
4. **Fitur tidak ditemukan = MISSING** — dilarang berasumsi ada di tempat lain
5. **Mode C: rekonstruksi berbasis kode nyata** — dilarang menambah asumsi domain
6. **Fix hanya apa yang ditemukan** — dilarang menambah fitur baru yang tidak ada di temuan
7. **Jangan overpromise** — bagian yang tidak bisa diverifikasi (runtime, third-party) → nyatakan
8. **Kode fix harus lengkap dan valid** — dilarang menulis fix parsial atau pseudocode

---

## Referensi

- `references/audit-dimensions.md` — Kriteria detail D1–D9 + contoh red flag dan fix
- `references/severity-guide.md` — Panduan menentukan CRITICAL/HIGH/MEDIUM/LOW + verdict
- `references/fix-patterns.md` — Pattern fix siap pakai per kategori bug

Baca `references/audit-dimensions.md` sebelum Fase 2.
Baca `references/fix-patterns.md` sebelum Fase 4.
