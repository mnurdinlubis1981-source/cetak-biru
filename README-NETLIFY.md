# Cetak Biru — versi Netlify + database online

Paket ini mengubah website Cetak Biru dari PHP/MySQL XAMPP menjadi:

- HTML/CSS/JavaScript untuk halaman web
- Netlify Functions sebagai backend API
- PostgreSQL sebagai database online
- Cookie login terpisah untuk pelanggan (`cb_customer`) dan admin (`cb_admin`), sehingga login admin tidak mengambil alih akun pelanggan di halaman utama

## Isi penting

- `index.html` — website utama
- `assets/app.js` — JavaScript website, sudah diarahkan ke `/api/...`
- `admin/index.html` — dashboard admin
- `netlify/functions/` — backend API
- `netlify/database/migrations/0001_initial.sql` — migration database PostgreSQL
- `database/schema.sql` — skema SQL yang sama, untuk database Postgres eksternal
- `.env.example` — contoh variabel yang harus diisi
- `netlify.toml` — konfigurasi Netlify

## Cara 1: Netlify Database

1. Buat akun/login di Netlify.
2. Buat site baru dan hubungkan repository GitHub yang berisi seluruh folder paket ini.
3. Setelah project tersedia di Netlify, buka **Data & Storage → Database** lalu buat database jika belum otomatis dibuat.
4. Netlify Database adalah PostgreSQL terkelola. Migration di `netlify/database/migrations/` akan diterapkan oleh Netlify pada siklus deploy.
5. Di **Project configuration → Environment variables**, buat:
   - `AUTH_SECRET` = string acak panjang (minimal 32 karakter)
   - `ADMIN_EMAIL` = email admin pertama
   - `ADMIN_PASSWORD` = password admin pertama
6. Redeploy site setelah environment variables diubah.
7. Buka `/api/health`. Jika tampil `API dan database terhubung.`, backend sudah tersambung.
8. Buka `/admin/` lalu login menggunakan `ADMIN_EMAIL` dan `ADMIN_PASSWORD`. Pada login admin pertama, akun admin dibuat otomatis di database.
9. Setelah itu pelanggan dapat membuka halaman utama dan mendaftar/login seperti biasa.

## Cara 2: Postgres eksternal

Cara ini dipakai kalau menu Netlify Database tidak tersedia pada akun kamu.

1. Buat database PostgreSQL di provider yang kamu pilih, misalnya Supabase atau Neon.
2. Jalankan seluruh isi `database/schema.sql` pada SQL editor provider tersebut.
3. Di Netlify Environment Variables, isi:
   - `DATABASE_URL` = connection string PostgreSQL provider
   - `AUTH_SECRET` = string acak panjang minimal 32 karakter
   - `ADMIN_EMAIL` = email admin pertama
   - `ADMIN_PASSWORD` = password admin pertama
4. Deploy ulang.
5. Tes `/api/health`.
6. Login admin di `/admin/`.

## Deploy dari GitHub

Upload seluruh folder proyek ini sebagai satu repository. Di Netlify pilih **Add new project → Import an existing project** lalu pilih repository tersebut. `netlify.toml` sudah mengatur folder Functions di `netlify/functions` dan site publish di root folder.

## Endpoint yang tersedia

- `GET /api/health`
- `GET /api/products`
- `POST /api/register`
- `POST /api/login`
- `GET /api/me`
- `POST /api/logout`
- `POST /api/create-order`
- `GET /api/orders`
- `GET /api/order-detail?id=...`
- `POST /api/cancel-order`
- `POST /api/admin-login`
- `GET /api/admin-me`
- `POST /api/admin-logout`
- `GET /api/admin-orders`
- `POST /api/admin-update-order-status`

## Catatan

Keranjang tetap disimpan di browser dengan `localStorage` per akun. Akun, produk, pesanan, rincian pesanan, dan status pesanan disimpan di database online.

Jangan menaruh `AUTH_SECRET`, `ADMIN_PASSWORD`, atau `DATABASE_URL` di file HTML/JavaScript. Simpan semuanya sebagai Environment Variables di Netlify.

Untuk penggunaan publik berskala besar, tambahkan rate limiting, verifikasi email/reset password, backup database, dan pengelolaan hak akses admin yang lebih lengkap.
