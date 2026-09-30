# Integrasi

AustinPay: dokumentasi https://austinstore.id/api-docs dibaca 30 September 2026. Adapter ada di `lib/adapters.ts`. Pterodactyl belum diuji live; sumber resmi instalasi Pterodactyl tidak dapat diambil di sesi ini, sehingga kompatibilitas harus divalidasi dengan panel Anda.

## AustinPay

Isi `AUSTIN_API_KEY`. Header `X-API-Key` secara resmi masih didukung dan dipakai agar key tidak muncul pada URL/log. Jika Anda membuat API Secret, isi `AUSTIN_API_SECRET`, karena signature menjadi wajib. HMAC-SHA256 hex dari `METHOD\nPATH\nRAW_BODY\nTIMESTAMP`, dikirim melalui `X-Signature` dan `X-Timestamp`; path tanpa query.

QRIS menggunakan POST `/api/deposit/create` dengan `{amount}`. ID disimpan dari `deposit.transaction_id`. `deposit.amount` merupakan nominal total tepat termasuk fee dan unique_code. UI memakai qr_image base64 dan expired_at provider. Status diperiksa lewat GET `/api/deposit/check/:transactionId`; hanya paid diterima. Interval minimal 5 detik per transaksi. Create dibatasi 5/menit global.

Wallet merchant AustinPay terpisah dari saldo pengguna website. Transaksi provider dipetakan ke pesanan atau deposit lokal. Deposit pengguna dikreditkan sebesar nominal dasar satu kali; pembelian QRIS tidak ikut menambah saldo pengguna.

Wajib whitelist IP. Cloudflare Worker tidak menjamin IP keluar tetap. Untuk produksi, jalankan `scripts/austin-relay.mjs` pada VPS ber-IP tetap, di belakang Nginx HTTPS. Isi key pada env relay dan whitelist IP VPS. Isi `AUSTIN_RELAY_URL=https://domain-relay` di backend. Relay bind localhost, membatasi endpoint dan key, serta mempertahankan raw body/signature. Jangan whitelist seluruh internet. Jangan log key, secret, headers, atau body transaksi.

## Webhook

Isi `AUSTIN_WEBHOOK_SECRET` memakai webhook secret AustinPay (berbeda dari API secret). Daftarkan `https://domain/api/webhook/austinpay` di dashboard provider. Endpoint wajib dapat diakses AustinPay; Site privat memerlukan publikasi dengan akses publik atau deployment standalone untuk callback.

Signature `X-AustinPay-Signature` diverifikasi dari raw body memakai HMAC-SHA256 dan perbandingan waktu konstan. Header event wajib sesuai body event. Handler deposit.paid mengantrekan pemeriksaan di database dan membalas 2xx tanpa menunggu provisioning, agar sesuai batas waktu 10 detik. Worker memeriksa status resmi provider lalu melakukan settlement. Replay tidak menggandakan saldo atau server. Callback tanpa pemetaan dicatat untuk rekonsiliasi.

## Kegagalan dan rekonsiliasi

Timeout/response rusak saat create membuat status unknown. Tidak ada retry create otomatis. Admin mencocokkan nominal dasar, waktu, dan pengguna pada catatan merchant, lalu menautkan ID melalui Rekonsiliasi di dashboard. Backend memeriksa history dan check API; admin tidak dapat menandai pembayaran lunas secara langsung. Jangan membuat pembayaran baru tanpa memeriksa transaksi unknown.

## Pterodactyl

Isi `PTERO_URL` sebagai origin HTTPS panel dan `PTERO_KEY` sebagai Application API key dengan hak akun/server/node yang diperlukan. `PTERO_LOCATION` adalah ID lokasi. Isi `PTERO_EGGS` atau Admin → Pengaturan → Lokasi & Egg: ID egg, docker_image, startup, dan environment harus sesuai egg lokal Node.js/Python. Deploy memilih node tersedia pada lokasi tersebut. Pilihan node spesifik belum disediakan di UI.

Akun memakai external_id `felix-<user-id>`, server `felix-<order-id>`. Retry selalu mencari external_id sebelum create. Akun email yang sudah ada namun belum tertaut tidak diambil alih otomatis; admin perlu memverifikasi penautan external_id. Jangan mengubah external_id setelah dipakai aplikasi.

Reseller mengisi email pelanggan dan bukti buyer. Jika pelanggan memiliki akun website, pelanggan harus memverifikasi email dan menerima keterkaitan sebelum admin memproses. Pelanggan tanpa akun website memperoleh akun panel tersendiri; panel tercatat pada reseller. Akun website yang dibuat belakangan membutuhkan penautan manual setelah verifikasi, bukan sekadar mencocokkan email.

Root admin Pterodactyl selalu memerlukan persetujuan dan tidak memberi akses web_admin. Worker mencabut akses setelah kedaluwarsa bila tidak ada pembelian admin lain yang masih aktif. Cakupan root admin mencakup seluruh instalasi panel.

`DATA_KEY` minimal 32 karakter acak, mengamankan password awal dengan AES-GCM. Jangan rotasi tanpa migrasi ciphertext. Password hanya ditampilkan setelah panel selesai; pengguna diminta segera menggantinya.
