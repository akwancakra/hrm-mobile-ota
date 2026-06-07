# HRM Mobile OTA

Repository publik untuk distribusi OTA (Over-The-Air) bundle aplikasi HRM Mobile.

Mendukung **Android** (saat ini) dan **iOS** (segera) dengan mekanisme custom OTA.

## Struktur Directory

```
ota/
├── android/
│   └── production/          ← Custom OTA untuk Android (aktif)
│       ├── manifest.json
│       └── bundles/*.zip
│
├── ios/
│   └── production/          ← Custom OTA untuk iOS (segera)
│       ├── manifest.json
│       └── bundles/*.zip
│
└── expoupdates/
    └── production/          ← Expo Updates self-hosted (mendatang)
        └── ...
```

## GitHub Pages

Aktifkan GitHub Pages dari:

- Source: `Deploy from a branch`
- Branch: `main`
- Folder: `/ (root)`

URL manifest:

```text
Android: https://akwancakra.github.io/hrm-mobile-ota/ota/android/production/manifest.json
iOS:     https://akwancakra.github.io/hrm-mobile-ota/ota/ios/production/manifest.json
```

## Keamanan

Repo ini hanya berisi artifact OTA yang aman untuk publik:

- Bundle JavaScript (sudah di-minify, tanpa source code asli)
- Manifest JSON (metadata)

Jangan simpan source code aplikasi, `.env`, keystore, atau credential di repo ini.

## Rollback

Rollback cukup dengan mengembalikan `manifest.json` ke versi bundle sebelumnya, lalu commit dan push.

Jika perlu mematikan OTA sementara, ubah `disabled` menjadi `true` di `manifest.json`.
