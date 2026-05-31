# Referensi: Panduan Severity & Verdict

---

## Skala Severity

### 🔴 CRITICAL — Deploy DIBLOKIR

Kondisi yang memenuhi salah satu:
- Aplikasi crash pada alur utama (happy path) dalam kondisi normal
- Data bisa corrupt, hilang, atau ganda secara permanen
- Celah keamanan langsung bisa dieksploitasi (SQLi, hardcoded secret, no auth)
- Fitur inti di PRD tidak ada sama sekali di kode (Mode A)
- Fungsi kritis hilang dari kode baru dibanding referensi (Mode B)
- Transaksi finansial non-atomik yang bisa menyebabkan inkonsistensi data
- Race condition yang pasti terjadi pada concurrent usage normal

### 🟠 HIGH — Harus Fix Segera (sebelum atau sesaat setelah deploy)

- Crash pada edge case yang umum terjadi (array kosong, input null)
- Error ditelan tanpa logging — tidak bisa debug production issue
- N+1 query pada endpoint yang sering diakses
- Validasi input tidak ada — data invalid masuk database
- Auth dicek tapi kepemilikan tidak dicek (IDOR)
- Race condition yang mungkin terjadi saat load lebih dari normal

### 🟡 MEDIUM — Fix Sebelum Rilis Berikutnya

- Edge case yang jarang terjadi tidak dihandle
- Error message tidak informatif
- Performa buruk tapi hanya terasa di dataset besar
- Kode yang membingungkan dan berpotensi menjadi bug saat dimodifikasi

### 🔵 LOW — Advisory

- Nama variabel kurang deskriptif
- Fungsi terlalu panjang tapi masih bisa dipahami
- Magic number yang sebaiknya dijadikan konstanta
- Komentar yang bisa ditambahkan

---

## Tabel Penentu Severity

| Dampak \ Kemungkinan | Pasti Terjadi | Sering | Jarang | Sangat Jarang |
|----------------------|---------------|--------|--------|---------------|
| Data hilang/corrupt  | CRITICAL      | CRITICAL | HIGH | MEDIUM |
| Celah keamanan       | CRITICAL      | CRITICAL | HIGH | HIGH |
| Aplikasi crash       | CRITICAL      | HIGH   | MEDIUM | LOW |
| Output salah         | HIGH          | HIGH   | MEDIUM | LOW |
| Performa buruk       | HIGH          | MEDIUM | LOW    | LOW |
| UX buruk             | MEDIUM        | LOW    | LOW    | LOW |

---

## Aturan Khusus per Mode

### Mode A (PRD-Based):
- Fitur di PRD → **tidak ada** di kode = **CRITICAL** (otomatis)
- Fitur di PRD → **partial** = **HIGH**
- Fitur di kode → **tidak ada di PRD** = **MEDIUM** (catat, minta konfirmasi)

### Mode B (Reference-Based):
- Fungsi kritis di referensi → **hilang** di kode baru = **CRITICAL**
- Behavior yang **berubah tanpa disengaja** = **HIGH**
- Behavior berubah tapi **tidak didokumentasikan** = **MEDIUM**

### Mode C (Self-Discovery):
- Fitur yang "seharusnya ada" berdasarkan rekonstruksi intent tapi tidak ada = **HIGH**
  (bukan CRITICAL karena tidak ada sumber kebenaran eksternal)
- Pola stub/incomplete yang jelas = **CRITICAL** jika di alur utama, **HIGH** jika di alur sekunder

---

## Verdict Keseluruhan

### ✅ PRODUCTION READY
- Zero temuan CRITICAL
- Semua temuan HIGH sudah ada rencana fix
- Kondisi ini disetujui untuk deploy

### ⚠️ CONDITIONAL
- Zero CRITICAL
- Ada temuan HIGH — bisa deploy dengan komitmen fix dalam X hari
- Semua temuan MEDIUM dan LOW sudah tercatat

### ❌ BLOCKED
- Ada satu atau lebih temuan CRITICAL
- Deploy tidak boleh dilakukan sampai semua CRITICAL selesai dan diverifikasi
