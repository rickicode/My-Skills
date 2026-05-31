# Referensi: Fix Patterns

Pattern fix siap pakai yang digunakan AI saat Fase 4 (Autofix).
Dibaca sebelum memulai autofix untuk memastikan fix yang dihasilkan konsisten dan benar.

---

## FP-01 — Stub / Fungsi Tidak Terimplementasi

**Deteksi**: Fungsi hanya return default, throw "not implemented", atau body kosong.

**Prinsip fix**:
- Implementasi nyata harus sesuai dengan nama fungsi dan context penggunaannya
- Trace semua pemanggil fungsi ini untuk memahami expected input/output
- Jika logika bisnis tidak bisa dipastikan dari kode → buat implementasi dengan
  `// AUTOFIX: skeleton — perlu review logika bisnis` dan isi dengan logika paling masuk akal

```js
// SEBELUM — stub
async function calculateTotal(items) {
  return 0; // TODO: implement
}

// SESUDAH — implementasi nyata
async function calculateTotal(items) {
  // AUTOFIX: implementasi kalkulasi total dari stub
  if (!items || items.length === 0) return 0;
  return items.reduce((sum, item) => {
    const itemTotal = (item.price ?? 0) * (item.quantity ?? 1);
    return sum + itemTotal;
  }, 0);
}
```

---

## FP-02 — Missing Try/Catch pada Async

**Deteksi**: `async function` atau `.then()` tanpa error handling.

**Prinsip fix**:
- Wrap seluruh async operation dalam try/catch
- Error harus di-log dengan konteks (nama fungsi, parameter kritis)
- Re-throw dengan error yang informatif, atau return error object yang konsisten
- Jangan telan error diam-diam

```js
// SEBELUM — tidak ada error handling
async function saveOrder(orderData) {
  const order = await db.orders.create(orderData);
  await updateStock(order.items);
  return order;
}

// SESUDAH — dengan error handling
async function saveOrder(orderData) {
  // AUTOFIX: tambah try/catch untuk async operation
  try {
    const order = await db.orders.create(orderData);
    await updateStock(order.items);
    return order;
  } catch (error) {
    console.error('[saveOrder] Gagal menyimpan order:', {
      error: error.message,
      orderData: { userId: orderData.userId, itemCount: orderData.items?.length }
    });
    throw new Error(`Gagal menyimpan order: ${error.message}`);
  }
}
```

---

## FP-03 — Operasi Non-Atomik (Missing Transaction)

**Deteksi**: Beberapa operasi DB berurutan tanpa transaksi, di mana gagalnya satu
akan meninggalkan data inkonsisten.

**Prinsip fix**:
- Identifikasi semua operasi yang harus berhasil atau gagal bersama
- Bungkus dalam `db.transaction()` (atau equivalent ORM yang dipakai)
- Pastikan semua operasi dalam transaksi menggunakan `trx` bukan `db` langsung

```js
// SEBELUM — non-atomik
async function processCheckout(userId, cartItems) {
  for (const item of cartItems) {
    await db.inventory.decrement({ id: item.productId }, item.qty);
  }
  const order = await db.orders.create({ userId, status: 'pending' });
  const orderItems = await db.orderItems.bulkCreate(
    cartItems.map(i => ({ orderId: order.id, ...i }))
  );
  return order;
}

// SESUDAH — atomik dengan transaksi
async function processCheckout(userId, cartItems) {
  // AUTOFIX: bungkus dalam transaksi agar atomik
  return await db.transaction(async (trx) => {
    for (const item of cartItems) {
      await db.inventory.decrement(
        { id: item.productId },
        item.qty,
        { transaction: trx }
      );
    }
    const order = await db.orders.create(
      { userId, status: 'pending' },
      { transaction: trx }
    );
    await db.orderItems.bulkCreate(
      cartItems.map(i => ({ orderId: order.id, ...i })),
      { transaction: trx }
    );
    return order;
  });
}
```

---

## FP-04 — Missing Edge Case Guard

**Deteksi**: Operasi pada nilai yang bisa null/undefined/empty tanpa guard.

**Prinsip fix**:
- Tambahkan guard clause di awal fungsi (early return pattern)
- Gunakan optional chaining `?.` dan nullish coalescing `??`
- Untuk array: cek `.length` sebelum akses index atau `.reduce()`

```js
// SEBELUM — tidak ada guard
function getOrderSummary(order) {
  const itemCount = order.items.length;
  const total = order.items.reduce((sum, i) => sum + i.price, 0);
  const firstItem = order.items[0].name;
  return { itemCount, total, firstItem };
}

// SESUDAH — dengan guard
function getOrderSummary(order) {
  // AUTOFIX: tambah guard untuk null/empty
  if (!order) return null;
  const items = order.items ?? [];
  const itemCount = items.length;
  const total = items.reduce((sum, i) => sum + (i.price ?? 0), 0);
  const firstItem = items[0]?.name ?? '-';
  return { itemCount, total, firstItem };
}
```

---

## FP-05 — SQL Injection / Parameterized Query

**Deteksi**: String interpolasi langsung ke query SQL.

**Prinsip fix**:
- Ganti interpolasi dengan parameterized query sesuai library yang dipakai
- Jangan ubah logika query, hanya cara passing parameter

```js
// SEBELUM — vulnerable
const user = await db.raw(`SELECT * FROM users WHERE email = '${email}'`);
const product = await db.raw(
  `SELECT * FROM products WHERE name LIKE '%${search}%'`
);

// SESUDAH — parameterized
// AUTOFIX: ganti string interpolation ke parameterized query
const user = await db.raw('SELECT * FROM users WHERE email = ?', [email]);
const product = await db.raw(
  'SELECT * FROM products WHERE name LIKE ?',
  [`%${search}%`]
);

// Atau jika pakai Sequelize/Prisma ORM:
const user = await User.findOne({ where: { email } });
const products = await Product.findAll({
  where: { name: { [Op.like]: `%${search}%` } }
});
```

---

## FP-06 — XSS via innerHTML

**Deteksi**: `element.innerHTML = userInput` atau template literal ke innerHTML.

**Prinsip fix**:
- Ganti `innerHTML` dengan `textContent` untuk teks biasa
- Jika butuh render HTML: gunakan library sanitizer (DOMPurify)
- Di React: hindari `dangerouslySetInnerHTML`, jika terpaksa pakai sanitize dulu

```js
// SEBELUM — XSS vulnerable
element.innerHTML = userInput;
div.innerHTML = `<span>${user.comment}</span>`;

// SESUDAH — safe
// AUTOFIX: ganti innerHTML ke textContent untuk mencegah XSS
element.textContent = userInput;

// Jika memang perlu render HTML (misal: rich text dari CMS):
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

---

## FP-07 — Hardcoded Secret / Credential

**Deteksi**: API key, password, secret, token langsung di kode.

**Prinsip fix**:
- Pindah ke environment variable
- Tulis placeholder yang jelas
- Tambahkan validasi bahwa env var ada saat startup

```js
// SEBELUM — hardcoded
const JWT_SECRET = "supersecretkey123";
const DB_URL = "postgresql://admin:password@localhost:5432/mydb";
const MIDTRANS_KEY = "SB-Mid-server-xxx";

// SESUDAH — env variable
// AUTOFIX: pindah secret ke environment variable
const JWT_SECRET = process.env.JWT_SECRET;
const DB_URL = process.env.DATABASE_URL;
const MIDTRANS_KEY = process.env.MIDTRANS_SERVER_KEY;

// Tambahkan validasi startup (letakkan di file config/env.js atau app entry):
const REQUIRED_ENV = ['JWT_SECRET', 'DATABASE_URL', 'MIDTRANS_SERVER_KEY'];
for (const key of REQUIRED_ENV) {
  if (!process.env[key]) {
    throw new Error(`Environment variable ${key} tidak ditemukan. Cek file .env`);
  }
}
```

---

## FP-08 — Missing Auth / Ownership Check

**Deteksi**: Endpoint yang mengakses/modifikasi data tanpa cek autentikasi atau
kepemilikan (IDOR — Insecure Direct Object Reference).

**Prinsip fix**:
- Auth check: pastikan middleware auth dipasang sebelum handler
- Ownership check: bandingkan `resource.userId` dengan `req.user.id`
- Kembalikan 403 (bukan 404) jika ownership tidak cocok

```js
// SEBELUM — tidak ada auth + tidak ada ownership check
app.get('/api/orders/:id', async (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order);
});

app.delete('/api/orders/:id', async (req, res) => {
  await Order.delete(req.params.id);
  res.json({ success: true });
});

// SESUDAH — dengan auth middleware + ownership check
// AUTOFIX: tambah auth middleware dan ownership check
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ error: 'Order tidak ditemukan' });
    if (order.userId !== req.user.id) {
      return res.status(403).json({ error: 'Akses ditolak' });
    }
    res.json(order);
  } catch (error) {
    console.error('[GET /orders/:id]', error.message);
    res.status(500).json({ error: 'Terjadi kesalahan server' });
  }
});

app.delete('/api/orders/:id', requireAuth, async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ error: 'Order tidak ditemukan' });
    if (order.userId !== req.user.id) {
      return res.status(403).json({ error: 'Akses ditolak' });
    }
    await Order.delete(req.params.id);
    res.json({ success: true });
  } catch (error) {
    console.error('[DELETE /orders/:id]', error.message);
    res.status(500).json({ error: 'Terjadi kesalahan server' });
  }
});
```

---

## FP-09 — N+1 Query

**Deteksi**: Loop yang melakukan query database di setiap iterasinya.

**Prinsip fix**:
- Kumpulkan semua ID terlebih dahulu, lalu batch fetch dalam satu query
- Atau gunakan eager loading / JOIN jika ORM mendukung
- Map hasil batch ke dalam struktur yang dibutuhkan

```js
// SEBELUM — N+1 query
async function getOrdersWithProducts(userId) {
  const orders = await Order.findAll({ where: { userId } });
  for (const order of orders) {
    order.items = await OrderItem.findAll({ where: { orderId: order.id } });
    for (const item of order.items) {
      item.product = await Product.findById(item.productId); // N+1 di sini!
    }
  }
  return orders;
}

// SESUDAH — eager loading jika ORM mendukung
// AUTOFIX: ganti N+1 dengan eager loading
async function getOrdersWithProducts(userId) {
  const orders = await Order.findAll({
    where: { userId },
    include: [{
      model: OrderItem,
      include: [{ model: Product }]
    }]
  });
  return orders;
}

// Alternatif manual batch jika ORM tidak support include bertingkat:
// 1. Fetch semua orders
// 2. Ambil orderIds dan productIds unik
// 3. Fetch OrderItem dan Product sekali per tabel
// 4. Gabungkan hasil dengan map di memory
```

---

## FP-10 — Missing Input Validation

**Deteksi**: Data dari user/request langsung dipakai tanpa validasi format/tipe/range.

**Prinsip fix**:
- Validasi di layer paling atas (controller/handler), sebelum masuk ke service/DB
- Cek: tipe data, required fields, range nilai, format (email, tanggal, dll.)
- Return 400 dengan pesan yang jelas jika validasi gagal

```js
// SEBELUM — tidak ada validasi
app.post('/api/products', async (req, res) => {
  const product = await Product.create(req.body);
  res.json(product);
});

// SESUDAH — dengan validasi
// AUTOFIX: tambah input validation sebelum proses
app.post('/api/products', requireAuth, async (req, res) => {
  const { name, price, stock, categoryId } = req.body;

  // Validasi required fields
  const missing = [];
  if (!name?.trim()) missing.push('name');
  if (price === undefined || price === null) missing.push('price');
  if (stock === undefined || stock === null) missing.push('stock');
  if (missing.length > 0) {
    return res.status(400).json({
      error: `Field wajib tidak ada: ${missing.join(', ')}`
    });
  }

  // Validasi tipe dan range
  if (typeof price !== 'number' || price < 0) {
    return res.status(400).json({ error: 'price harus berupa angka non-negatif' });
  }
  if (!Number.isInteger(stock) || stock < 0) {
    return res.status(400).json({ error: 'stock harus berupa integer non-negatif' });
  }

  try {
    const product = await Product.create({ name: name.trim(), price, stock, categoryId });
    res.status(201).json(product);
  } catch (error) {
    console.error('[POST /products]', error.message);
    res.status(500).json({ error: 'Gagal membuat produk' });
  }
});
```

---

## Panduan Umum untuk Fix yang Tidak Ada Pattern-nya

Jika menemukan bug yang tidak masuk kategori di atas:

1. **Pahami intent** — baca nama fungsi, komentar, dan pemanggil untuk tahu apa yang seharusnya terjadi
2. **Fix minimal** — perbaiki hanya yang perlu, jangan refactor lebih dari yang dibutuhkan
3. **Pertahankan style** — ikuti naming convention, indentation, dan pattern yang ada di kode
4. **Tandai dengan komentar** — `// AUTOFIX: [alasan singkat perubahan]`
5. **Tulis fungsi lengkap** — bukan diff atau snippet — agar langsung bisa dipakai
6. **Jika tidak yakin** — tandai sebagai `[PERLU CEK]` dan masukkan ke catatan manual, jangan fix dengan asumsi
