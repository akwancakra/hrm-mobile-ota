# HRM Mobile OTA

Repository publik untuk distribusi OTA (Over-The-Air) bundle aplikasi HRM Mobile.

Mendukung **Android** dan **iOS** dengan mekanisme custom OTA via GitHub Pages.

---

## Struktur Directory

```
ota/
├── android/
│   └── production/          ← OTA untuk Android
│       ├── manifest.json
│       └── bundles/*.zip
│
└── ios/
    └── production/          ← OTA untuk iOS
        ├── manifest.json
        └── bundles/*.zip
```

---

## GitHub Pages Setup

Aktifkan GitHub Pages dari:

- **Source:** `Deploy from a branch`
- **Branch:** `main`
- **Folder:** `/ (root)`

URL manifest:

```text
Android: https://akwancakra.github.io/hrm-mobile-ota/ota/android/production/manifest.json
iOS:     https://akwancakra.github.io/hrm-mobile-ota/ota/ios/production/manifest.json
```

---

## Cara Publish OTA

### Prerequisites

- OTA repo sudah di-clone di lokal (`hrm-mobile-ota`)
- GitHub Pages sudah aktif (branch: main, folder: /)

### Publish Sekaligus Android + iOS

Dari direktori mobile app (`hrm-mobile-arsitekhijau`):

```bash
# Windows
npm run publish:ota -- -Version 12 -Message "Fix bug approval OTA"

# atau dengan path repo eksplisit
powershell -ExecutionPolicy Bypass -File scripts/publish-ota.ps1 `
  -Version 12 `
  -OtaRepoPath "D:\Codingan\crm-arsitekhijau\hrm-mobile-ota" `
  -Message "Fix bug approval, tambah pull to refresh"
```

```bash
# macOS / Linux / Git Bash
npm run publish:ota:bash -- \
  -v 12 \
  -o /path/to/hrm-mobile-ota \
  -m "Fix bug approval, tambah pull to refresh"
```

### Hanya Satu Platform

```bash
# Android only
npm run publish:ota:bash -- -v 12 -o /path/to/ota-repo --android-only

# iOS only
npm run publish:ota:bash -- -v 12 -o /path/to/ota-repo --ios-only
```

### Yang Dilakukan Script

```
1. expo export:embed --platform android    → index.android.bundle + assets
2. expo export:embed --platform ios        → main.jsbundle + assets
3. Zip masing-masing + SHA-256 hash
4. Generate manifest.json untuk Android
5. Generate manifest.json untuk iOS
6. Copy ke folder hrm-mobile-ota/ota/{platform}/production/
7. Print summary
```

### Setelah Script Selesai

Commit & push OTA repo secara manual:

```bash
cd D:\Codingan\crm-arsitekhijau\hrm-mobile-ota
git add ota/
git commit -m "publish ota v12 (channel: production)"
git push origin main
```

### Changelog & Bundle Size

Setelah publish, edit `manifest.json` untuk menambah changelog (akan muncul di popup update user):

```json
{
  "version": 12,
  "changelog": [
    "Fix bug overtime force close",
    "Tambah pull to refresh di detail page",
    "Perbaiki approval flow line approver"
  ],
  "bundleSize": 5432100
}
```

Field `changelog` dan `bundleSize` opsional. Jika tidak diisi, popup update akan menampilkan `message` saja.

---

## Cara Naik Versi App

### Kapan Perlu Naik Versi App?

| Situasi | Contoh | Update via |
|---|---|---|
| Hanya ganti JS/logic | Fix bug, UI baru, fitur JS-only | **OTA aja** |
| Ganti native code | Plugin baru, SDK upgrade | **APK/IPA baru** + OTA |
| Ganti Hermes/RN version | RN 0.81 → 0.82 | **APK/IPA baru** + OTA |

### Langkah-langkah

**1. Update `app.json`** di project mobile app:

```json
{
  "version": "1.2.1",
  "android": {
    "versionCode": 3
  },
  "ios": {
    "buildNumber": "3"
  }
}
```

**2. Update `ota.config.ts`** di mobile app:

```typescript
const RUNTIME_BASE = "1.2.1-hermes-rn081";
```

**3. Build APK/IPA baru** → upload ke Play Store / App Store

**4. Publish OTA baru** setelah APK terdistribusi:

```bash
npm run publish:ota -- -Message "Fitur baru v1.2.1"
```

### Aturan Penting

- **Runtime version** = `{platform}-{appVersion}-hermes-rn{rnMajor}{rnMinor}`
- OTA lama **tetap jalan** untuk user yang belum update APK (karena runtime version berbeda, manifest terpisah)
- Version OTA (angka di manifest) **tidak reset** — terus increment meskipun app version berubah
- User APK lama dan APK baru dapat OTA dari **channel berbeda** (manifest terpisah per runtime version)

---

## Format Manifest

```json
{
  "platform": "android",
  "channel": "production",
  "runtimeVersion": "android-1.0.0-hermes-rn081",
  "version": 12,
  "updateId": "ota-android-production-v12",
  "createdAt": "2026-06-07T10:00:00Z",
  "bundleUrl": "https://akwancakra.github.io/hrm-mobile-ota/ota/android/production/bundles/ota-android-production-v12.zip",
  "sha256": "abc123def456...",
  "mandatory": false,
  "disabled": false,
  "message": "Fix bug approval, tambah pull to refresh",
  "changelog": [
    "Fix bug approval",
    "Tambah pull to refresh"
  ],
  "bundleSize": 5432100
}
```

| Field | Wajib | Fungsi |
|---|---|---|
| `version` | Ya | Nomor versi OTA (auto-increment, tidak reset saat app version naik) |
| `runtimeVersion` | Ya | Gating — bundle hanya cocok untuk Hermes/RN version tertentu |
| `bundleUrl` | Ya | URL download bundle zip |
| `sha256` | Ya | Verifikasi integritas file download |
| `platform` | Ya | `android` atau `ios` |
| `disabled` | Tidak | Kill switch — `true` = semua client skip update |
| `mandatory` | Tidak | Saat ini belum di-enforce client-side (selalu optional) |
| `message` | Tidak | Deskripsi update (tampil di popup jika changelog kosong) |
| `changelog` | Tidak | Array string perubahan (tampil di popup) |
| `bundleSize` | Tidak | Ukuran bundle dalam bytes (tampil di popup) |

---

## Rollback

```bash
# 1. Kembalikan manifest.json ke versi sebelumnya
cd D:\Codingan\crm-arsitekhijau\hrm-mobile-ota
git checkout HEAD~1 -- ota/android/production/manifest.json
git checkout HEAD~1 -- ota/ios/production/manifest.json

# 2. Commit & push
git commit -m "rollback OTA v12 -> v11 (bug di v12)"
git push origin main
```

**Cara kerja rollback:**
- Client akan mendownload bundle sesuai version di manifest
- Jika manifest kembali ke v11, client akan download v11 lagi (SHA-256 berbeda)
- Prosesnya sama seperti update biasa — download → verify → install → restart

### Kill Switch

Set `disabled: true` di `manifest.json`:

```json
{
  "disabled": true
}
```

→ Commit → push → **semua client skip update** sampai `disabled` diubah ke `false`.

Cocok untuk situasi darurat (bundle bermasalah) tanpa harus rollback penuh.

---

## Multi-CDN (Fallback Mirrors)

App mendukung **multiple CDN** dengan urutan prioritas. Jika primary CDN (GitHub Pages) gagal, app akan otomatis coba fallback CDN berikutnya.

### Menambah Mirror Baru

**1. Clone/mirror OTA repo ke CDN lain:**

```bash
# Contoh: deploy ke Netlify
git clone https://github.com/akwancakra/hrm-mobile-ota.git
# Upload folder 'ota/' ke Netlify / Cloudflare Pages / VPS kamu
```

**2. Update `ota.config.ts`** di mobile app:

```typescript
const FALLBACK_CDNS: string[] = [
  "https://hrm-ota-mirror.netlify.app",
  "https://ota.vps-kamu.com",
];
```

### Auto-Sync via GitHub Action (Opsional)

Buat `.github/workflows/sync-ota.yml` di repo `hrm-mobile-ota`:

```yaml
name: Sync OTA to Mirror
on:
  push:
    branches: [main]
    paths: ["ota/**"]
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Netlify
        run: curl -X POST -d {} https://api.netlify.com/build_hooks/TOKEN
```

### Urutan Prioritas

1. **Primary:** GitHub Pages (`akwancakra.github.io`)
2. **Fallback 1:** Mirror CDN (jika ada)
3. **Fallback N:** Mirror berikutnya (jika ada)

App akan coba CDN pertama. Jika gagal (HTTP error / timeout / JSON invalid), fallback ke CDN berikutnya. Jika **semua** gagal, update di-skip dan log error ke console.

---

## Keamanan

Repo ini hanya berisi artifact OTA yang aman untuk publik:

- Bundle JavaScript (sudah di-minify, tanpa source code asli)
- Manifest JSON (metadata update)

**Jangan** simpan source code aplikasi, `.env`, keystore, atau credential di repo ini.

### Safety Mechanisms

| Mekanisme | Fungsi |
|---|---|
| **Runtime version gating** | Cegah bundle RN version mismatch |
| **SHA-256 verification** | Cegah bundle corrupt / MITM |
| **Platform guard** | Bundle Android hanya untuk Android, iOS hanya untuk iOS |
| **`disabled` flag** | Instant kill switch |
| **`__DEV__` guard** | OTA tidak pernah jalan di development mode |
| **Max 3 versions** | Cegah storage penuh di device |
| **Silent failure** | Update gagal tidak mengganggu user |

---

## Troubleshooting

### OTA Tidak Jalan

1. Cek `disabled` di manifest → harus `false`
2. Cek `runtimeVersion` di manifest → harus cocok dengan `ota.config.ts`
3. Cek `platform` di manifest → harus sesuai device
4. Cek GitHub Pages → pastikan URL manifest bisa diakses via browser
5. Cek console log app → `[OTA] Update check failed: ...`

### Bundle Gagal Download

Jika user di region dengan akses lambat ke GitHub Pages:

1. Tambah CDN mirror yang lebih dekat ke region user (lihat **Multi-CDN** di atas)
2. Atau kecilkan bundle size dengan code splitting

### Force Close Setelah Update

Rollback ke versi sebelumnya (lihat **Rollback** di atas), lalu fix bug di bundle baru.
