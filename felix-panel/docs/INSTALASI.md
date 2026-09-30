# Instalasi FELIX PANEL

Gunakan Node.js 24 dan pnpm. Jalankan `pnpm install --frozen-lockfile`. Salin `.env.example` menjadi `.env`; isi secret di backend, jangan commit nilai asli. Contoh egg wajib disesuaikan dengan instalasi Anda, bukan dianggap konfigurasi universal.

Jalankan `pnpm build`, lalu terapkan SQL berurutan pada database lokal:

```bash
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_nosy_wolverine.sql
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0001_smart_sunspot.sql
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0002_third_lily_hollister.sql
```

`pnpm start` menjalankan build lokal. Cookie session memakai HttpOnly, Secure, SameSite=Lax, sehingga gunakan HTTPS untuk domain lokal kustom. Runtime managed Sites menggunakan preview supervisor sesuai lingkungan.

## Admin pertama

Isi `ADMIN_EMAIL` dengan email pemilik dan `ADMIN_SETUP_TOKEN` dengan kode acak panjang. Daftarkan akun dengan email tersebut, buka Akun Saya, lalu masukkan kode aktivasi. Kode dan email wajib cocok; hanya bekerja jika belum ada admin website. Hapus token setup dari runtime setelah berhasil. Pendaftar pertama tidak otomatis menjadi admin. Tidak ada password admin bawaan.

Buka Admin → Paket, siapkan katalog awal, atur harga/spec/garansi, dan aktifkan hanya produk yang siap dijual. Harga pratinjau bersifat contoh. Pengaturan persetujuan tersedia di Admin → Pengaturan. Akses admin Pterodactyl tetap selalu memerlukan persetujuan manual.

## Penyimpanan

Manifest menggunakan binding `DB` dan `BUCKET`. Sites membuat resource aktual dan menerapkan migrasi saat publikasi. Stack ini tidak memakai DATABASE_URL. Deploy Cloudflare standalone memerlukan D1 dan R2 sendiri dengan konfigurasi Wrangler dari build; jangan pakai resource ID placeholder di produksi.

Saldo menggunakan D1 batch dan SQL trigger untuk mencegah debit/kredit ganda serta saldo negatif. Schema berada di `db/schema.ts`; jangan ubah migrasi yang sudah diterapkan. Buat migrasi baru dengan `pnpm db:generate` setelah perubahan schema. Cadangkan D1 dan R2 berkala.

Upload PNG/JPG maksimal 3 MB, magic bytes diperiksa, disimpan di R2, hanya dapat dibaca pemilik dan admin. Payload besar dibatasi sebelum parsing. Browser storage hanya untuk tema dan pilihan checkout sementara, bukan data transaksi.
