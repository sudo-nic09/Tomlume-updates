# Tomlume updates

Public feed for the Tomlume Android app (`app.tomlume.reader`). This repo holds
no source code - only the two small documents the app reads and the APK
releases it installs from.

## Files

`policy.json` - the remote pause. Checked on launch, when the app comes back to
the foreground, and every 30 minutes while open. The last response is cached on
the device, so an announced pause also applies offline.

```json
{
  "locked": false,
  "after": "2026-12-01",
  "message": "Shown on the pause screen.",
  "applyToUnlock": false
}
```

- `locked: true` is the intent; `after` (a `YYYY-MM-DD` local date or an ISO
  timestamp) is when it takes effect. `locked: false` releases it.
- `applyToUnlock: false` leaves unlock builds alone, which is the current
  behaviour and matches the shipped app.

`update.json` - the update manifest. Checked on launch, every six hours, and
when the app is foregrounded (at most once every 30 minutes). A native
background job also checks every six hours while the app is closed and posts
one notification per new `versionCode`; it only reads this file and never
downloads the APK by itself.

```json
{
  "versionCode": 5,
  "versionName": "1.4",
  "apkUrl": "https://github.com/sudo-nic09/Tomlume-updates/releases/download/v1.4/Tomlume-1.4.apk",
  "sha256": "56f49126aee3623ad4f924485dd1d0818178757b98da15feae551d8ac5ff48fe",
  "notes": "One short line shown in the app."
}
```

The app only offers an update when `versionCode` is strictly greater than the
installed one. Nothing downloads until the reader taps **Update**; the SHA-256
is required, and the APK must also carry the same package name and signing key
as the installed app before Android's installer is opened. An update installs
over the existing app and keeps every book, note and highlight.

Releases up to 1.3 were signed with a temporary debug key. Version 1.4 moves to
Tomlume's permanent release key, so 1.3 installs need one manual install of
`Tomlume-1.4.apk`; every release from 1.4 onward updates in place.

## Releasing

1. Build the unlock APK in the Tomlume source repo (`npm run build:android:unlock`,
   then `./gradlew assembleRelease`), copy
   `android/app/build/outputs/apk/release/app-release.apk` to `apk/tomlume.apk`,
   and run `npm run update:manifest -- apk/tomlume.apk`.
2. Create a release here tagged `v<versionName>` with the APK uploaded as
   `Tomlume-<versionName>.apk`. `apkUrl` is generated from exactly that tag and
   file name, so the two cannot drift apart.
3. Commit the regenerated `update.json` to `main`.

Both files are served over `https://raw.githubusercontent.com/sudo-nic09/Tomlume-updates/main/`.
