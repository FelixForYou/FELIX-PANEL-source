# FELIX PANEL

Website auto order Pterodactyl berbahasa Indonesia: React 19, Vinext, TypeScript, Cloudflare Worker, D1 SQL, R2, AustinPay. Biru neomorphic, sidebar desktop, bottom navigation mobile, dark mode.

Panduan: [Instalasi](docs/INSTALASI.md), [Integrasi](docs/INTEGRASI.md), [Deployment dan worker](docs/DEPLOYMENT.md).

Struktur proyek:

- `app/page.tsx` dan `app/globals.css`: UI interaktif.
- `app/api/[...path]/route.ts`: API, autentikasi, RBAC, upload, admin, webhook.
- `lib/backend.ts`: transaksi, saldo, idempotensi, provisioning.
- `lib/adapters.ts`: koneksi provider.
- `lib/crypto.ts` dan `lib/pdf.ts`: password, signature, enkripsi, invoice.
- `db/schema.ts` dan `drizzle/`: schema dan migrasi.
- `tests/run.mjs`: pengujian provider simulasi.
- `scripts/reconcile-worker.mjs`: worker rekonsiliasi.
- `scripts/austin-relay.mjs`: relay IP keluar tetap opsional.

```bash
pnpm install --frozen-lockfile
pnpm exec tsc --noEmit
node --experimental-transform-types tests/run.mjs
pnpm build
```

Database produksi tidak memuat data dummy. Harga contoh berlabel pratinjau, paket awal nonaktif. Kredensial live, whitelist IP, egg, email, dan worker perlu disiapkan pemilik.
