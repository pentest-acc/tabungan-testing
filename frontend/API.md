# Dokumentasi API — Sistem Tabungan Qurban

Base URL: `http://localhost:5000/api`

Semua respons berbentuk:

```json
{ "success": true, "message": "Berhasil", "data": { } }
```

Endpoint terproteksi membutuhkan header: `Authorization: Bearer <token>`.

Role: `jamaah`, `admin_biasa`, `kepala_admin`. Kepala admin memiliki seluruh akses admin biasa.

---

## Auth

| Method | Endpoint | Akses | Deskripsi |
| --- | --- | --- | --- |
| POST | `/auth/register` | Publik | Registrasi jamaah |
| POST | `/auth/login` | Publik | Login jamaah & admin (satu pintu) |
| GET | `/auth/me` | Login | Profil user yang sedang login |

**POST /auth/register** — body:

```json
{ "username": "fauzi", "password": "rahasia123", "nama_lengkap": "Ahmad Fauzi", "no_telp": "081234567890", "alamat": "Bekasi" }
```

**POST /auth/login** — body `{ "username", "password" }`, respons:

```json
{ "data": { "token": "<jwt>", "user": { "id_jamaah": "JMH-...", "nama_lengkap": "...", "role": "jamaah" } } }
```

## Admin (khusus `kepala_admin`)

| Method | Endpoint | Deskripsi |
| --- | --- | --- |
| GET | `/admin?q=` | Daftar admin (search opsional) |
| POST | `/admin` | Daftarkan admin biasa — `{ username, password, nama_lengkap }` |
| PUT | `/admin/:id_admin` | Ubah admin biasa (password opsional) |
| DELETE | `/admin/:id_admin` | Hapus admin biasa |

## Jamaah

| Method | Endpoint | Akses | Deskripsi |
| --- | --- | --- | --- |
| GET | `/jamaah?q=` | Admin | Daftar jamaah |
| GET | `/jamaah/:id_jamaah` | Admin | Detail jamaah |
| POST | `/jamaah` | Admin | Daftarkan jamaah (yang kesulitan membuat akun) |
| PUT | `/jamaah/profil` | Jamaah | Perbarui profil sendiri |
| PUT | `/jamaah/:id_jamaah` | Admin | Ubah data jamaah |
| DELETE | `/jamaah/:id_jamaah` | Admin | Hapus akun + keanggotaan + permintaan |

## Sapi Qurban

| Method | Endpoint | Akses | Deskripsi |
| --- | --- | --- | --- |
| GET | `/sapi?q=` | Login | Katalog sapi (search penanda) |
| GET | `/sapi/:id_sapi` | Login | Detail sapi |
| POST | `/sapi` | Admin | Tambah — `{ penanda_sapi, bobot_estimasi, harga_sapi?, harga_porsi? }` (default harga 23.800.000 / porsi 3.400.000) |
| PUT | `/sapi/:id_sapi` | Admin | Ubah data sapi |
| DELETE | `/sapi/:id_sapi` | Admin | Hapus (ditolak jika dipakai kelompok) |

## Kelompok Qurban

| Method | Endpoint | Akses | Deskripsi |
| --- | --- | --- | --- |
| GET | `/kelompok?q=&status=` | Login | Daftar kelompok + sapi + tabungan + jumlah_anggota |
| GET | `/kelompok/:id_kelompok` | Login | Detail + daftar anggota |
| GET | `/kelompok/:id_kelompok/anggota` | Login | Anggota kelompok |
| POST | `/kelompok` | Admin | Buat kelompok — **Tabungan_Qurban dibuat otomatis** (sisa tagihan awal = harga sapi). Body: `{ id_sapi, nomor_kelompok, tanggal_mulai, tanggal_berakhir, status? }` |
| PUT | `/kelompok/:id_kelompok` | Admin | Ubah kelompok |
| DELETE | `/kelompok/:id_kelompok` | Admin | Bubarkan + hapus tabungan, anggota, permintaan, transaksi terkait |
| POST | `/kelompok/:id_kelompok/gabung` | Jamaah | Ajukan permintaan bergabung — `{ catatan? }` |

## Permintaan Bergabung (admin)

| Method | Endpoint | Deskripsi |
| --- | --- | --- |
| GET | `/permintaan?status=` | Daftar permintaan + data jamaah & kelompok |
| PUT | `/permintaan/:id_permintaan/terima` | Terima → masuk Detail_Kelompok; kuota maks. **7 anggota** (permintaan ditolak otomatis saat kuota penuh) |
| PUT | `/permintaan/:id_permintaan/tolak` | Tolak permintaan |

## Tabungan

| Method | Endpoint | Akses | Deskripsi |
| --- | --- | --- | --- |
| GET | `/tabungan/saya` | Jamaah | Tabungan kelompok + tagihan pribadi: `total_terkumpul`, `sisa_tagihan_kelompok`, `harga_porsi`, `total_dibayar`, `sisa_tagihan_pribadi`, `kelompok` |
| GET | `/tabungan/kelompok/:id_kelompok` | Login | Tabungan sebuah kelompok |

## Transaksi

| Method | Endpoint | Akses | Deskripsi |
| --- | --- | --- | --- |
| GET | `/transaksi?jenis=&status=` | Admin | Monitoring seluruh transaksi |
| GET | `/transaksi/saya` | Jamaah | Riwayat transaksi sendiri |
| GET | `/transaksi/:id_transaksi` | Login | Detail transaksi |
| POST | `/transaksi/bayar` | Jamaah | Setor dana. JSON atau `multipart/form-data` dengan file gambar `bukti` (diupload ke Cloudinary). Body: `{ id_tabungan, total_bayar, metode_bayar, jenis_transaksi: "tunai"\|"cicil" }`. Otomatis menambah `total_terkumpul` & mengurangi `sisa_tagihan_kelompok` |
| PUT | `/transaksi/:id_transaksi/status` | Admin | Update `status_pembayaran` — `{ status: "pending"\|"success"\|"failed" }` (saldo tabungan ikut dikoreksi) |
| DELETE | `/transaksi/:id_transaksi` | Admin | Batalkan transaksi / refund (saldo dikembalikan) |

## Laporan (admin)

| Method | Endpoint | Deskripsi |
| --- | --- | --- |
| GET | `/laporan/ringkasan` | Statistik: `total_jamaah`, `total_admin`, `total_sapi`, `total_kelompok`, `total_transaksi`, `total_dana`, `total_tabungan_terkumpul`, `total_sisa_tagihan`, `kelompok_per_status` |
| GET | `/laporan/kelompok` | Progres tabungan per kelompok |

## Kode Error

| Status | Arti |
| --- | --- |
| 401 | Token tidak ada / tidak valid / kredensial salah |
| 403 | Role tidak berhak |
| 404 | Data tidak ditemukan |
| 409 | Konflik (username dipakai, kuota penuh, sapi dipakai, dll.) |
| 422 | Validasi input gagal (`errors: [{ field, message }]`) |
