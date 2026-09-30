# Deployment dan worker

Site dipublikasikan sebagai pratinjau privat. Untuk pelanggan dan callback provider, aktifkan akses publik ketika integrasi siap, atau deploy standalone Cloudflare. Frontend statis saja tidak cukup. Runtime secret dikelola pada environment backend; bukan frontend/source. Source ini bukan aplikasi bot Express yang dijalankan dengan node src/bot.js.

## Background worker

Jalankan `scripts/reconcile-worker.mjs` pada VPS Node 22/24 dengan systemd/PM2. Worker memeriksa pembayaran pending, memulihkan status settled dan provisioning terputus setelah 120 detik, melakukan retry aman, suspend panel kedaluwarsa, mencabut root admin kedaluwarsa, dan mengirim pengingat tiga hari sebelumnya. Loop setiap 15 detik menunggu putaran sebelumnya selesai.

Isi env worker: `FELIX_ORIGIN` URL HTTPS aplikasi, `WORKER_TOKEN` sama dengan secret backend. `OAI_SITES_SERVICE_TOKEN` diperlukan hanya untuk Site privat bila token akses layanan tersedia. Jangan pakai cookie pengguna sebagai token otomatisasi. Worker produksi belum dijalankan dalam deliverable karena konfigurasi/kredensial belum diberikan.

Contoh systemd:

```ini
[Unit]
Description=FELIX PANEL reconciliation
After=network-online.target

[Service]
Type=simple
User=felix
WorkingDirectory=/opt/felix-panel
EnvironmentFile=/etc/felix-panel-worker.env
ExecStart=/usr/bin/node /opt/felix-panel/scripts/reconcile-worker.mjs
Restart=on-failure
RestartSec=10
NoNewPrivileges=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Env worker izin 0600. Aktifkan `systemctl enable --now felix-panel-worker`. Worker memeriksa pesanan yang ada; tidak menagih ulang. Callback hanya mengantrekan pekerjaan, sehingga worker harus aktif untuk proses otomatis.

## Email

`EMAIL_RELAY_URL` menerima POST `{to,subject,text}` dengan Bearer `EMAIL_RELAY_TOKEN`. Relay milik Anda harus mengirim melalui SMTP/provider email. Jika belum dikonfigurasi, UI menampilkan layanan email belum tersedia; verifikasi tetap opsional. Email gagal tidak ditandai sukses.

## Refund dan perpanjangan

Refund dikreditkan ke saldo website dengan alasan audit, satu kali. Hanya pesanan berbayar yang belum menghasilkan layanan. Bila server ditemukan di remote setelah timeout, refund ditolak hingga rekonsiliasi. Akses admin yang pernah masuk aktivasi harus direkonsiliasi sebelum refund. Tidak ada refund rekening/gateway otomatis.

Perpanjangan memakai server lama, unsuspend setelah pembayaran/persetujuan, dan menambah masa aktif dari akhir periode berjalan atau sekarang. Panel hanya disuspend ketika habis; penghapusan data otomatis tidak diterapkan.

PDF dihasilkan backend dengan perlindungan kepemilikan. Font Helvetica Latin dasar; nama non-Latin diganti tanda tanya pada PDF. Invoice layar mendukung Unicode penuh. Leaderboard periode mingguan/bulanan memakai rentang berjalan 7/30 hari.

## Verifikasi dan batas

Pengujian backend dengan SQLite serta provider simulasi mencakup registrasi, role, checkout, voucher, saldo, deposit, webhook signed/replay, provisioning/retry, reseller, keterkaitan pelanggan, PDF, garansi, dan leaderboard. TypeScript dan build lulus. Tidak ada pembayaran, provisioning, atau email live yang diklaim telah diuji. QA visual browser dan validasi konteks WebMCP tidak tersedia pada sesi ini. WebMCP staging paket menggunakan feature detection dan tidak melakukan pembelian otomatis.
