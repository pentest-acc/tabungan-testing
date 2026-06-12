# Backend — Sistem Tabungan Qurban

REST API Express + MongoDB untuk aplikasi Tabungan Qurban. Kontrak endpoint cocok langsung dengan frontend React di folder `../frontend`.

## Teknologi

Node.js, Express.js, MongoDB + Mongoose, JWT, Bcrypt, Multer + Cloudinary (bukti pembayaran), Express Validator, Dotenv.

## Menjalankan

```bash
cd backend
npm install
cp .env.example .env     # isi MONGO_URI & JWT_SECRET (Cloudinary opsional)
npm run seed             # isi data awal + akun demo
npm run dev              # atau: npm start
```

Akun demo hasil seed:

| Role | Username | Password |
| --- | --- | --- |
| Kepala Admin | `kepala` | `kepala123` |
| Admin Biasa | `admin` | `admin123` |
| Jamaah | `zaid`, `saptur`, `abi`, `fauzi`, `siti` | `jamaah123` |

## Struktur Folder

```
backend/
├── server.js                  # Entry point
├── package.json
├── .env.example
├── API.md                     # Dokumentasi seluruh endpoint
└── src/
    ├── app.js                 # Setup Express + routes + error handler
    ├── config/
    │   ├── db.js              # Koneksi MongoDB
    │   └── cloudinary.js      # Upload bukti pembayaran
    ├── models/                # Skema sesuai dokumen MongoDB
    │   ├── Admin.js           # role: admin_biasa | kepala_admin
    │   ├── Jamaah.js
    │   ├── SapiQurban.js      # harga default 23.8jt / porsi 3.4jt
    │   ├── KelompokQurban.js  # status: aktif | selesai | expired
    │   ├── TabunganQurban.js  # 1:1 dengan kelompok
    │   ├── DetailKelompok.js  # keanggotaan (maks. 7)
    │   ├── PermintaanGabung.js
    │   └── Transaksi.js       # jenis: tunai | cicil
    ├── controllers/           # auth, admin, jamaah, sapi, kelompok,
    │                          #   permintaan, tabungan, transaksi, laporan
    ├── routes/                # Route per modul + index.js
    ├── middleware/
    │   ├── auth.js            # JWT authenticate + authorize (RBAC)
    │   ├── validate.js        # Express-validator runner
    │   ├── upload.js          # Multer (memory, gambar maks 2MB)
    │   └── errorHandler.js    # Error terpusat + 404
    ├── services/
    │   ├── tabunganService.js # Tabungan otomatis, catat/batalkan pembayaran
    │   └── kelompokService.js # Kuota 7, enrich data, sinkron status
    ├── validators/index.js    # Aturan validasi semua input
    ├── utils/                 # generateId, response helper
    └── seed/seed.js           # Data awal
```

## Aturan Bisnis Utama

- **Kelompok dibuat → Tabungan_Qurban otomatis dibuat** (1:1), sisa tagihan awal = harga sapi.
- **Maksimal 7 anggota per kelompok**; saat kuota penuh permintaan bergabung ditolak otomatis.
- Permintaan bergabung berstatus `pending` → admin **terima** (masuk `Detail_Kelompok`) atau **tolak**.
- Pembayaran (`tunai`/`cicil`) otomatis menambah `total_terkumpul` dan mengurangi `sisa_tagihan_kelompok`; pembatalan/refund mengembalikan saldo.
- Password di-hash Bcrypt; seluruh endpoint terproteksi JWT + role-based access control.
