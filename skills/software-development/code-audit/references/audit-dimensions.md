# Referensi: Kriteria Audit per Dimensi (D1–D9)

---

## D1 — Kelengkapan Implementasi

**Tujuan**: Tidak ada kode yang hanya berpura-pura bekerja.

### Red Flags — Tandai FAIL jika ditemukan:

```js
// Stub / fungsi kosong
function calculateDiscount() { return 0; }
function sendEmail() { return true; }
async function syncInventory() {}

// TODO / placeholder yang belum diselesaikan
// TODO: implement this later
// FIXME: this is a hack

// Data hardcoded yang harusnya dinamis
const TAX_RATE = 0.11;                      // ok jika konstanta resmi
const users = [{ id: 1, name: "Admin" }];   // RED FLAG jika ini "data produksi"
const STORE_ID = "store_001";               // RED FLAG jika multi-store

// Mock yang tertinggal di production path
if (process.env.NODE_ENV !== "test") {
  // logic asli tidak pernah jalan saat test
}
Math.random() > 0.5 ? successFlow() : failFlow();  // logika random = mock

// Import yang tidak dipakai (indikasi fitur tidak jadi diimplementasi)
import { PaymentGateway } from './payment'; // tapi tidak pernah dipanggil
```

### Untuk MODE A — Cek per fitur di PRD:
Setiap fitur di PRD harus bisa di-trace ke fungsi/komponen nyata di kode.
"Ada di kode" artinya: ada fungsi, dipanggil, return nilai nyata, dan terhubung ke UI/API.

### Untuk MODE C — Pattern yang dicurigai "belum selesai":
- Fungsi yang hanya ada definisi tapi tidak pernah dipanggil dari manapun
- Komponen UI yang dirender tapi tidak ada event handler-nya
- Database schema yang punya kolom tapi tidak ada kode yang mengisinya
- Route/endpoint yang ada tapi controller-nya hanya `res.json({ status: 'ok' })`

---

## D2 — Ketepatan Algoritma & Logika Bisnis

**Tujuan**: Kode menghasilkan output yang benar secara matematis dan logis.

### Checklist:
- [ ] Kalkulasi finansial benar (urutan operasi: diskon → pajak, bukan terbalik)
- [ ] Kondisi boolean tidak terbalik (`>` vs `>=`, `&&` vs `||`, negasi ganda)
- [ ] Loop tidak off-by-one (`<` vs `<=`, index mulai dari 0 atau 1)
- [ ] Floating point: apakah aman? (gunakan integer sen/rupiah untuk uang)
- [ ] Pembulatan: `Math.floor`, `Math.ceil`, atau `Math.round`? Mana yang tepat?
- [ ] Date/time: timezone dihandle? Format konsisten?
- [ ] Sorting: ascending/descending sesuai kebutuhan?
- [ ] Pagination: offset/limit dihitung dengan benar?

### Contoh Bug Umum:

```js
// SALAH — urutan operasi: diskon harusnya dari harga asli
const total = (price * 1.11) * (1 - discount);  // diskon dari harga+pajak
// BENAR
const total = price * (1 - discount) * 1.11;    // diskon dulu, pajak kemudian

// SALAH — off-by-one
for (let i = 0; i <= arr.length; i++) { arr[i].name }  // crash di i === length
// BENAR
for (let i = 0; i < arr.length; i++) { ... }

// SALAH — float precision
0.1 + 0.2 === 0.3  // false di JavaScript!
// BENAR — untuk uang, gunakan integer (dalam sen/rupiah terkecil)
const totalCents = Math.round(price * 100) + Math.round(tax * 100);

// SALAH — logika terbalik
if (!isLoggedIn && hasPermission) { grantAccess(); }
// harusnya: if (isLoggedIn && hasPermission)

// SALAH — tanggal tanpa timezone
new Date("2024-01-15")  // bisa jadi hari sebelumnya di timezone tertentu
// BENAR
new Date("2024-01-15T00:00:00+07:00")  // eksplisit timezone
```

### Verifikasi Manual untuk Logika Kritis:
Untuk kalkulasi finansial, tulis contoh hitungan eksplisit dan verifikasi:
```
Contoh: Harga Rp100.000, diskon 10%, PPN 11%
Expected: 100.000 × (1 - 0.1) = 90.000 → × 1.11 = 99.900
Kode menghasilkan: [trace nilai di kode]
Match? [Ya / Tidak]
```

---

## D3 — Penanganan Edge Case

**Tujuan**: Tidak crash atau hasil salah pada input di luar kondisi normal.

### Edge Case Universal:
```js
// NULL / UNDEFINED
const name = user.profile.name;           // crash jika profile null
const name = user?.profile?.name ?? '-';  // aman

// ARRAY KOSONG
const first = items[0].price;             // crash jika items = []
const first = items[0]?.price ?? 0;       // aman

// DIVIDE BY ZERO
const avg = total / count;                // NaN / Infinity jika count = 0
const avg = count > 0 ? total / count : 0;

// STRING KOSONG / WHITESPACE ONLY
if (input) { process(input); }            // lolos jika input = "   "
if (input?.trim()) { process(input); }    // benar
```

### Edge Case Spesifik Domain POS / Keuangan:
```
- Harga = 0 (item gratis)
- Diskon = 0% dan 100%
- Quantity = 0 (hapus item?) dan negatif (return?)
- Total transaksi = 0
- Refund > nilai transaksi asli
- Stok = 0 saat checkout
- Dua user checkout item terakhir bersamaan (race condition stok)
- Payment gagal setelah order dibuat
- Printer struk tidak terhubung
- Barcode tidak ditemukan di database
```

### Edge Case Form / Input:
```
- Submit dua kali cepat (double-click, double-submit)
- Paste 10.000 karakter ke input text
- Karakter spesial: <script>, ', ", \n, emoji
- Input yang hanya spasi
- Angka negatif di field quantity/harga
- Tanggal tidak valid (31 Februari)
```

---

## D4 — Error Handling & Resiliensi

**Tujuan**: Error tidak menyebabkan crash atau data korup — dan bisa di-debug.

### Pola yang Harus Ditandai FAIL:

```js
// Error ditelan — BERBAHAYA
try { await saveOrder(); } catch (e) {}
try { await saveOrder(); } catch (e) { console.log(e); }  // log tapi tidak handle

// Unhandled promise — BERBAHAYA
fetchProducts().then(setProducts);  // tanpa .catch()
// atau di async function tanpa try/catch:
async function loadData() {
  const data = await api.get('/products');  // tidak ada try/catch
  setData(data);
}

// Error message tidak berguna
catch (e) { throw new Error("Terjadi kesalahan"); }
catch (e) { res.json({ error: "Error" }); }

// Async tanpa await — data tidak tersimpan tapi dianggap sukses
function createOrder(data) {
  db.save(data);           // tidak di-await!
  return { success: true }; // selalu sukses padahal save belum selesai
}
```

### Yang Harus Ada:

```js
// BENAR — error handling lengkap
async function processPayment(orderId, amount) {
  try {
    const result = await paymentGateway.charge(amount);
    await db.updateOrderStatus(orderId, 'paid');
    return { success: true, transactionId: result.id };
  } catch (error) {
    logger.error('Payment failed', { orderId, amount, error: error.message });
    await db.updateOrderStatus(orderId, 'payment_failed');
    throw new PaymentError(`Pembayaran gagal: ${error.message}`, { orderId });
  }
}
```

### Checklist:
- [ ] Semua `async/await` punya `try/catch`
- [ ] Error message menyebut konteks (fungsi apa, data apa, kenapa)
- [ ] Error di-log dengan level yang tepat (error untuk bug, warn untuk expected failure)
- [ ] UI menampilkan pesan yang layak — bukan blank screen / spinner selamanya
- [ ] Sensitive data tidak masuk ke log (password, nomor kartu, token)
- [ ] Timeout diset untuk operasi jaringan yang bisa hang

---

## D5 — Integritas Data & Atomisitas

**Tujuan**: Data tidak pernah dalam keadaan setengah-tersimpan.

### Red Flag — Operasi Non-Atomik:

```js
// BERBAHAYA — jika langkah 2 gagal, stok sudah berkurang tapi order tidak dibuat
async function checkout(cartItems, userId) {
  for (const item of cartItems) {
    await db.decrementStock(item.productId, item.qty);  // langkah 1
  }
  const order = await db.createOrder(userId, cartItems); // langkah 2 — bisa gagal!
  await db.createOrderItems(order.id, cartItems);        // langkah 3 — bisa gagal!
}

// BENAR — atomic dengan transaksi
async function checkout(cartItems, userId) {
  return await db.transaction(async (trx) => {
    for (const item of cartItems) {
      await trx.decrementStock(item.productId, item.qty);
    }
    const order = await trx.createOrder(userId, cartItems);
    await trx.createOrderItems(order.id, cartItems);
    return order;
  });
}
```

### Checklist:
- [ ] Create + update yang saling tergantung dibungkus transaksi
- [ ] Delete tidak meninggalkan orphan record (foreign key constraint atau cascade)
- [ ] Stok/inventory diupdate atomik dengan pembuatan order
- [ ] Tidak ada operasi baca-ubah-tulis tanpa locking (race condition)
- [ ] Soft delete diimplementasi konsisten (tidak ada yang hard delete kecuali disengaja)

---

## D6 — Keamanan

**Tujuan**: Tidak ada celah yang bisa dieksploitasi.

### Critical Red Flags:

```js
// HARDCODED SECRET — CRITICAL
const JWT_SECRET = "mysecretkey123";
const DB_PASSWORD = "admin123";
const API_KEY = "sk-prod-abc...";

// SQL INJECTION — CRITICAL
const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;
// BENAR: gunakan parameterized query
const query = db.raw('SELECT * FROM users WHERE email = ?', [req.body.email]);

// XSS — CRITICAL
element.innerHTML = userInput;
res.send(`<h1>Hello ${req.query.name}</h1>`);
// BENAR: sanitize atau gunakan text content
element.textContent = userInput;

// IDOR — akses tanpa cek kepemilikan — CRITICAL
app.get('/api/orders/:id', async (req, res) => {
  const order = await Order.findById(req.params.id);
  // tidak cek apakah order.userId === req.user.id !
  res.json(order);
});

// AUTH TIDAK DICEK — CRITICAL
app.delete('/api/products/:id', async (req, res) => {
  // tidak ada middleware auth!
  await Product.delete(req.params.id);
  res.json({ success: true });
});
```

### Checklist:
- [ ] Semua secret dari environment variable, bukan hardcoded
- [ ] Semua input user di-sanitize/validate sebelum diproses
- [ ] Parameterized query dipakai — tidak ada string interpolation ke SQL
- [ ] Setiap endpoint sensitif punya middleware auth
- [ ] Setiap endpoint data user punya cek kepemilikan (userId match)
- [ ] CORS tidak `*` untuk production (kecuali public API)
- [ ] Password di-hash (bcrypt/argon2) — tidak disimpan plain text
- [ ] Sensitive data tidak masuk response yang tidak perlu (password hash, token)

---

## D7 — Performa & Efisiensi

**Tujuan**: Tidak ada bottleneck yang meledak saat production load.

### N+1 Query — Paling Sering Terjadi:

```js
// RED FLAG — N+1: 1 query untuk orders + N query untuk tiap order
const orders = await Order.findAll();
for (const order of orders) {
  order.items = await OrderItem.findAll({ where: { orderId: order.id } });
}

// BENAR — eager loading atau join
const orders = await Order.findAll({
  include: [{ model: OrderItem }]
});
```

### Checklist:
- [ ] Loop yang melakukan query database di dalamnya = N+1, harus diperbaiki
- [ ] Endpoint list punya pagination (limit/offset atau cursor)
- [ ] Query yang filter/sort punya index di kolom yang dipakai
- [ ] Tidak ada `SELECT *` untuk tabel dengan banyak kolom
- [ ] Tidak ada `console.log` berlebihan di production path
- [ ] Tidak ada blocking sync call di dalam async handler

---

## D8 — Keterbacaan & Maintainability

*(Advisory — tidak memblokir deploy)*

### Yang Dinilai:
- Nama variabel/fungsi deskriptif (`getActiveOrders` bukan `getData`)
- Fungsi tidak > 50 baris tanpa alasan kuat
- Magic number diganti konstanta (`TAX_RATE = 0.11` bukan `* 0.11` langsung)
- Kode yang di-comment-out tanpa penjelasan dihapus
- Komentar ada untuk logika non-obvious

---

## D9 — Kesesuaian PRD / Referensi

*(Hanya Mode A dan Mode B)*

### Mode A — Output Wajib:
```
KESESUAIAN PRD
Fitur di PRD                    | Status          | Lokasi di Kode / Catatan
--------------------------------|-----------------|-------------------------
[nama fitur dari PRD]           | ✅ Implemented  | [file › fungsi]
[nama fitur dari PRD]           | ❌ Missing      | Tidak ditemukan
[nama fitur dari PRD]           | ⚠️ Partial      | Ada tapi [apa yang kurang]
[nama fitur dari PRD]           | 🔄 Modified     | Berbeda dari spec, konfirmasi?
```

### Mode B — Output Wajib:
```
KESESUAIAN REFERENSI
Fungsi/Behavior di Referensi    | Status             | Catatan
--------------------------------|--------------------|--------
[nama fungsi/behavior]          | ✅ Equivalent      | -
[nama fungsi/behavior]          | ❌ Missing         | [dampak]
[nama fungsi/behavior]          | 🔄 Changed         | [apakah disengaja?]
[nama fungsi/behavior]          | ✨ Added (baru)    | [tidak ada di referensi]
```
