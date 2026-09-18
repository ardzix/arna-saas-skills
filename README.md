# Arnatech SaaS AI Skills & Platform Reference

Repository ini adalah sumber bersama untuk dua kebutuhan:

1. **AI-agent skills dan reference context** yang membantu agen pada platform AI apa pun mengikuti kontrak Arnatech saat merancang, meninjau, atau mengubah layanan.
2. **Dokumentasi manusia** untuk memahami batas kepemilikan layanan, tenancy, SSO, device, dan pembayaran sebelum membuat keputusan atau implementasi.

Repository ini tidak menggantikan source code, OpenAPI layanan, ataupun prosedur operasional produksi. Gunakan sebagai kontrak arsitektur lintas layanan yang harus dipenuhi oleh perubahan tersebut.

## Mulai sebagai pembaca dokumentasi

Untuk perubahan yang melibatkan lebih dari satu layanan, baca [platform contract](arnatech-platform/references/platform-contract.md) terlebih dahulu. Dokumen itu menjelaskan pemilik kapabilitas dan aturan inti berikut:

- shared-pool tenancy dengan `organization_id` dan `tenant_id`;
- central SSO berbasis PKCE dan app-local session;
- device identity untuk kiosk, POS, scanner, atau photobooth;
- kepemilikan Commerce, File Manager, Payment Router, dan Pulsar;
- kontrak HTTP dan baseline delivery.

Untuk QRIS, webhook, rekonsiliasi invoice, atau migrasi event, lanjutkan ke [payment-event contract](arnatech-payment-events/references/event-contract.md). Payment Router menerima webhook provider, Commerce menjadi pemilik status invoice/order/entitlement, dan Pulsar membawa fakta pembayaran yang versioned serta idempoten.

### Peta isi

| Area | Baca untuk memahami | Folder skill AI |
| --- | --- | --- |
| Arsitektur lintas layanan, tenancy, SSO, device | [Platform contract](arnatech-platform/references/platform-contract.md) | `arnatech-platform` |
| Backend API, worker, integrasi layanan | [Service skill](arnatech-service/SKILL.md) | `arnatech-service` |
| Web frontend, BFF, SSO, public route | [Web SSO skill](arnatech-web-sso/SKILL.md) | `arnatech-web-sso` |
| QRIS/Xendit, Commerce, Payment Router, Pulsar | [Payment-event contract](arnatech-payment-events/references/event-contract.md) | `arnatech-payment-events` |

### Contoh: photobooth publik

Layar tamu dapat tetap tanpa login, tetapi device photobooth wajib terdaftar dan terikat pada satu organisasi dan tenant. Operator memasangkan device melalui SSO Device Authorization Grant; device hanya memperoleh token untuk API photobooth. Backend photobooth, bukan device atau browser tamu, membuat order Commerce dengan konteks `organization_id`, `tenant_id`, `device_id`, dan event. Lihat [device identities and public terminals](arnatech-platform/references/platform-contract.md#device-identities-and-public-terminals).

## Pakai dengan platform AI

Setiap folder `arnatech-*` adalah skill mandiri: ia memiliki `SKILL.md` sebagai instruksi untuk AI dan, bila diperlukan, `references/` sebagai sumber aturan yang lebih rinci. Platform AI yang mendukung format skill berbasis `SKILL.md` dapat memuat folder-folder tersebut sebagai skill terpisah. Ikuti mekanisme instalasi platform tersebut dan pilih folder yang relevan, bukan root repository sebagai satu skill.

Platform yang belum mendukung skill juga tetap dapat memakai repository ini sebagai documentation context. Berikan AI dokumen yang sesuai dengan tugas, lalu instruksikan untuk mematuhinya. Contoh prompt yang platform-netral:

```text
Gunakan Arnatech SaaS Platform Reference berikut sebagai kontrak wajib.
Baca platform contract terlebih dahulu, kemudian service skill untuk implementasi backend.
Jangan mengubah tenancy, SSO, Commerce, File Manager, atau kontrak Pulsar tanpa
menyatakan dampak dan rencana migrasinya.
```

Untuk tugas lintas layanan, lampirkan atau tautkan [platform contract](arnatech-platform/references/platform-contract.md). Tambahkan [payment-event contract](arnatech-payment-events/references/event-contract.md) untuk pembayaran. Untuk tugas khusus, gunakan folder skill yang sesuai pada tabel di atas.

### Platform AI yang mendukung skills

Daftarkan masing-masing folder berikut sebagai satu skill sesuai dokumentasi platform AI yang digunakan:

```text
arnatech-platform/
arnatech-service/
arnatech-web-sso/
arnatech-payment-events/
```

Skill dipilih otomatis hanya bila platform mendukung discovery dari deskripsi. Untuk perubahan penting, sebutkan nama skill dan dokumen referensi yang harus dibaca dalam prompt agar perilakunya deterministik.

### Codex

Codex mendeteksi folder skill yang berisi `SKILL.md`. Gunakan instalasi skill bawaan Codex bila tersedia: panggil `$skill-installer`, lalu minta instalasi skill dari repository ini. Setelah terpasang, buka session baru atau restart Codex bila skill belum terlihat.

Alternatif manual adalah menyalin setiap folder `arnatech-*` langsung ke user skill directory Codex, yaitu `~/.agents/skills`. Jangan menyalin folder repository sebagai satu skill tunggal; setiap folder di bawah ini adalah skill tersendiri.

### Windows PowerShell

```powershell
$checkoutPath = Join-Path $env:TEMP 'arna-saas-skills'
$skillsPath = Join-Path $env:USERPROFILE '.agents\skills'

git clone git@github.com:ardzix/arna-saas-skills.git $checkoutPath
New-Item -ItemType Directory -Force $skillsPath
Copy-Item -Recurse $checkoutPath\arnatech-* $skillsPath
```

### macOS / Linux

```bash
git clone git@github.com:ardzix/arna-saas-skills.git /tmp/arna-saas-skills
mkdir -p ~/.agents/skills
cp -R /tmp/arna-saas-skills/arnatech-* ~/.agents/skills/
```

Skill Codex dapat dipilih otomatis dari deskripsi tugasnya, tetapi gunakan invocation eksplisit untuk perubahan penting:

```text
$arnatech-platform
Rancang kontrak tenant dan SSO untuk layanan baru.

$arnatech-service
Tambahkan API backend yang memakai tenant context dan Commerce.

$arnatech-web-sso
Implementasikan dashboard dengan central SSO.

$arnatech-payment-events
Rancang alur QRIS dan consumer Pulsar yang idempoten.
```

Di aplikasi desktop ChatGPT gunakan skill picker (`@`) bila tersedia; di Codex CLI atau ekstensi IDE gunakan `$skill-name` atau `/skills`. Skill memuat instruksi penuh hanya saat dipilih, sehingga referensi tetap ringkas pada session biasa.

Panduan lokasi skill, pemanggilan eksplisit, dan instalasi skill dari repository dijelaskan oleh [OpenAI Docs: Build skills](https://learn.chatgpt.com/docs/build-skills).

## Prinsip perubahan

- Jangan membuat identity, invoice, entitlement, file storage, atau webhook receiver kedua di layanan konsumen.
- Jangan mempercayai `organization_id`, `tenant_id`, role, atau status bayar yang datang dari browser, QR publik, header, atau callback provider tanpa verifikasi pada pemilik data.
- Pertahankan kontrak publik lama selama masa migrasi yang terdokumentasi; versioning dan idempotency wajib untuk event pembayaran.
- Jangan memasukkan secret, private key, app token personal, atau data produksi ke repository ini.

Saat mengubah kontrak, perbarui dokumen referensi yang relevan dan skill yang mengarahkan implementasinya dalam commit yang sama.
