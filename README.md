<p align="center">
  <img src="docs/images/feature-graphic.png" alt="OpenClone — Clone apps. Keep control." width="100%">
</p>

<h1 align="center"><img src="docs/images/app-icon.png" width="48" alt="" valign="middle"> OpenClone</h1>

<p align="center">
  An open-source Android app cloning utility.<br>
  Create a separate installable app from an app already on your device — no root, no ads, no account, no server.
</p>

<p align="center">
  <a href="https://raw.githubusercontent.com/bicknickiticky/openclone/main/apk/OpenClone-0.4.1-debug.apk"><b>⬇ Download the latest APK</b></a>
  &nbsp;·&nbsp;
  <a href="https://github.com/bicknickiticky/openclone/releases">All releases</a>
  &nbsp;·&nbsp;
  <a href="https://github.com/bicknickiticky/openclone">Source code</a>
</p>

<p align="center">
  <a href="https://github.com/bicknickiticky/openclone/actions/workflows/build.yml"><img alt="Build" src="https://github.com/bicknickiticky/openclone/actions/workflows/build.yml/badge.svg"></a>
  <a href="https://github.com/bicknickiticky/openclone/releases"><img alt="Latest release" src="https://img.shields.io/github/v/release/bicknickiticky/openclone?label=release&color=ffffff&labelColor=242424"></a>
  <img alt="Platform: Android" src="https://img.shields.io/badge/platform-Android-3DDC84?logo=android&logoColor=white">
  <img alt="Minimum Android: 8.0" src="https://img.shields.io/badge/minAndroid-8.0%20(API%2026)-242424">
  <img alt="Kotlin" src="https://img.shields.io/badge/Kotlin-1.9.24-7F52FF?logo=kotlin&logoColor=white">
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-ffffff?labelColor=242424"></a>
  <img alt="No ads" src="https://img.shields.io/badge/ads-none-242424">
  <a href="https://github.com/bicknickiticky/openclone/releases"><img alt="Downloads" src="https://img.shields.io/github/downloads/bicknickiticky/openclone/total?label=downloads&color=ffffff&labelColor=242424"></a>
</p>

---

## Screenshots

| App list | Clone configuration |
|:---:|:---:|
| <img src="docs/images/01-daftar-aplikasi.png" width="260" alt="OpenClone app list"> | <img src="docs/images/02-konfigurasi-clone.png" width="260" alt="OpenClone clone configuration"> |
| **Clone history** | **Settings** |
| <img src="docs/images/03-riwayat-clone.png" width="260" alt="OpenClone clone history"> | <img src="docs/images/04-pengaturan.png" width="260" alt="OpenClone settings"> |

---

## What is OpenClone?

OpenClone builds a new, separately installable APK from an app that is already installed on your Android device. The clone gets its own package name and is signed with an OpenClone-managed key, so Android treats it as a different app — a second account, a second profile, or a sandboxed copy without touching the original installation.

Everything runs on the phone. OpenClone has no account system, no ads, no analytics, and no remote cloning service. The manifest does not request the `INTERNET` permission.

> OpenClone changes the **app package identity**. It does **not** spoof the device, patch DEX or native code, bypass signature verification, or bypass Play Integrity. See [Compatibility and limitations](#compatibility-and-limitations).

## Features

### Discover

- Browse every installed app, system apps included, with a clear label for each.
- Search by app name or package name.
- See the app version and whether it is delivered as split APKs.
- Icons load lazily per visible row and are cached, so the list stays responsive on phones with hundreds of apps.

### Configure the clone

- Suggested clone label and package name (`com.example.app.clone1`, `.clone2`, …), skipping names that are already taken.
- Editable label and package name, validated before the clone starts.
- Optional custom launcher icon picked through Android's system file picker (no gallery permission).
- Advanced panel showing the source app version and the split APK count.
- Optional **Advertising ID permission removal** — the clone no longer declares `com.google.android.gms.permission.AD_ID`, so it reads a zeroed advertising ID instead of the device-wide one.

### Process

- Merge split APKs from App Bundle installs into a single output APK before rewriting.
- Rewrite the package name in the manifest and resource table, along with permission declarations, provider `authorities`, component permission references, and package-derived process names.
- Replace the app label and optionally the launcher icon.
- Strip old signing metadata while keeping unrelated `META-INF` payloads such as service descriptors.
- Sign the result with the persistent OpenClone key using APK Signature Scheme v1, v2, and v3.
- Run the whole pipeline off the main thread, with staged progress and a cancellable operation.

### Inspect the result

The result screen reports what may affect the clone instead of leaving you to guess:

- **Compatibility notes** — `sharedUserId`, signature-level permissions that stop applying after re-signing, apps that require extracted native libraries, and merged split APKs.
- **Identity notes** — Android ID changes automatically because the signing key changed, whether the advertising ID is zeroed or shared with the original, and the fact that device model and network identity stay the same for every app on the phone.

### Install, share, revisit

- Install the generated APK through Android's package installer.
- Share the generated APK with any app through the system share sheet.
- Clone history keeps every finished clone, so an APK can be re-installed, shared, or deleted later without cloning again.
- Search inside the history and see the stored size and creation time.

### Settings and signing key

- Key status row with subject, algorithm, expiry date, and SHA-256 fingerprint.
- **Export** the active signing key to a password-protected `.p12` file.
- **Import** a backup to restore the key on a new device, with a confirmation step before the active key is replaced.
- Informational rows for storage location and app version.

## Download

Get the latest build from the [Releases page](https://github.com/bicknickiticky/openclone/releases/latest).

| Option | Details |
|---|---|
| Latest release | [Download OpenClone-0.4.1-debug.apk](https://raw.githubusercontent.com/bicknickiticky/openclone/main/apk/OpenClone-0.4.1-debug.apk) |
| APK included in the repository | [`apk/OpenClone-0.4.1-debug.apk`](apk/OpenClone-0.4.1-debug.apk) |
| Build from source | Follow [Build from source](#build-from-source) |

The current `v0.4.1` release ships `OpenClone-0.4.1-debug.apk`, a **debug build intended for testing and personal use**, not a Play Store production release. Sizes and checksums are recorded in [`apk/README.md`](apk/README.md).

### Requirements on the phone

- Android 8.0 (API 26) or newer.
- **Install unknown apps** allowed for OpenClone, because clones are installed as ordinary APKs.
- `QUERY_ALL_PACKAGES` to list installed apps and `REQUEST_INSTALL_PACKAGES` to start installation.

## How it works

Android has no public API for running the same installed app twice. OpenClone therefore produces another installable package:

```text
Installed app
     │
     ▼
Copy the base APK and any split APKs
     │
     ▼
Load the APK bundle and merge splits
     │
     ▼
Rewrite package name and related manifest identifiers
     │
     ▼
Update app label, optional icon, optional AD_ID permission removal
     │
     ▼
Remove obsolete signature entries
     │
     ▼
Write and sign the new APK (v1 + v2 + v3)
     │
     ▼
Report compatibility and identity notes
     │
     ▼
Install or share the clone
```

The engine uses [REAndroid ARSCLib](https://github.com/REAndroid/ARSCLib) to read and rewrite resources and manifests, then [AOSP apksig](https://android.googlesource.com/platform/tools/apksig/) to sign the output.

### Implementation details

- **Package identity** is rewritten in the manifest and in the resource table, not just in one place.
- **Manifest references** — permission declarations, provider `authorities`, component permission references, and package-derived process names are adjusted where applicable.
- **Split APKs** are loaded as an APK bundle and merged before rewriting, so App Bundle installs produce one output file.
- **Signature cleanup** removes the previous signing block while retaining unrelated `META-INF` files.
- **`resources.arsc`** is written uncompressed to satisfy Android's APK installation rules.
- **Atomic output** — the APK is written and signed through temporary files, so an interrupted run never replaces a finished clone.
- **Keystore safety** — importing a backup writes to a temporary file first and only then swaps it in, so a failed import cannot destroy the active key.

## Compatibility and limitations

OpenClone only changes package metadata and re-signs the APK. It cannot make every app accept the result.

- Apps that verify their signing certificate, use Play Integrity or similar attestation, or implement anti-tampering checks may refuse to run. Banking apps and some games are the usual cases.
- **Google Sign-In does not work inside a clone.** The clone's package name and signing certificate are not registered in the original developer's Google Cloud / Firebase project, so Google rejects the request. OpenClone shows this warning on the result screen instead of pretending otherwise.
- A clone installs as a **fresh app**. OpenClone copies APK files, not the original app's private data or logged-in session.
- Clones cannot receive Play Store updates, because both the package name and the signing certificate differ.
- Apps that look up their own package name in custom resource paths or external integrations may misbehave.
- `sharedUserId` apps get a separate Linux user for the clone, so data from the original is not visible.
- Signature-level permissions declared by the app stop being granted to the clone after re-signing.
- Apps that require extracted native libraries may fail if the merged APK cannot load them on that device.

### What OpenClone deliberately does not do

- No DEX or native-code patching.
- No signature-check, SafetyNet, or Play Integrity bypass.
- No device identifier spoofing. `Build.BRAND`, `Build.MODEL`, `Build.MANUFACTURER`, the network identity, and the IP address are properties of the phone and stay identical for every app on it. Only the advertising ID can be affected, and only by removing the `AD_ID` permission from the clone.
- No root, no Xposed, no hooking framework.

OpenClone is meant for personal use with apps you are entitled to use. Do not use it to pirate apps, bypass paid features, commit fraud, or violate an app's terms or licence.

## Privacy and security

- All APK processing happens on the device.
- No account, no remote cloning server, no `INTERNET` permission in the manifest.
- No advertising or analytics SDK in the dependencies.
- The optional custom icon is read through Android's Storage Access Framework for a single picked image; no broad media or storage permission is requested.
- Generated APKs, clone metadata, and the signing keystore live in OpenClone's private app storage.

### Signing key: back it up

OpenClone creates one PKCS#12 keystore on first use and reuses that key for every clone. It is stored as `openclone-keystore.p12` in private app storage, with its internal password kept in private `SharedPreferences`.

**If the key is lost, existing clones cannot be updated in place.** They must be uninstalled and reinstalled, which deletes their app data.

Use **Settings → Kunci Penandatangan → Ekspor kunci** to save a password-protected backup before clearing app data, uninstalling OpenClone, or moving to another phone. On the new device, use **Impor kunci** to restore it. The exported file is protected only by the password you choose, so store it somewhere safe.

## Tech stack

| Area | Technology |
|---|---|
| Language | Kotlin 1.9.24 |
| Android build | Android Gradle Plugin 8.4.2, Gradle 8.7 |
| UI | Android Views, XML layouts, Material Components 1.12.0, RecyclerView, View Binding |
| Async work | Kotlin Coroutines 1.8.1 |
| APK and resource processing | [REAndroid ARSCLib](https://github.com/REAndroid/ARSCLib) 1.4.0 |
| APK signing | [AOSP apksig](https://android.googlesource.com/platform/tools/apksig/) 8.5.2 |
| Certificate and keystore handling | [Bouncy Castle](https://www.bouncycastle.org/) 1.78.1 |
| Unit tests | JUnit 4.13.2 |

## Build from source

### Requirements

- JDK 17 (AGP 8.4.2 does not support JDK 21+).
- Android SDK Platform 34.
- Android SDK Build-Tools 34.0.0.
- Internet access on the first build so Gradle can download dependencies.

### Build a debug APK

```bash
git clone https://github.com/bicknickiticky/openclone.git
cd openclone
./gradlew assembleDebug
```

The APK is written to:

```text
app/build/outputs/apk/debug/OpenClone-0.4.1-debug.apk
```

On Windows:

```powershell
gradlew.bat assembleDebug
```

If Gradle cannot find the Android SDK, set `ANDROID_HOME` or `ANDROID_SDK_ROOT`, or create a `local.properties` file with your SDK path. Do not commit `local.properties`.

### Run checks

```bash
./gradlew testDebugUnitTest
./gradlew lintDebug
./gradlew assembleDebug
```

GitHub Actions runs unit tests, Android lint, and the debug build on pushes to `main` and on pull requests. Tags matching `v*.*.*` publish a GitHub Release with the generated APK attached.

## Project structure

```text
openclone/
├── .github/workflows/build.yml
├── apk/
│   ├── OpenClone-0.4.1-debug.apk
│   └── README.md
├── app/src/main/
│   ├── AndroidManifest.xml
│   ├── java/dev/bill/openclone/
│   │   ├── MainActivity.kt          # App list tab, search, entry point
│   │   ├── AppPickerActivity.kt     # Full-screen app picker with search
│   │   ├── CloneConfigActivity.kt   # Clone name, package, icon, advanced options
│   │   ├── CloneProgressActivity.kt # Progress steps, result, notes, install/share
│   │   ├── HistoryActivity.kt       # Saved clones, search, reinstall, share, delete
│   │   ├── SettingsActivity.kt      # Preferences, keystore export/import, about link
│   │   ├── AboutActivity.kt         # Version, licence, credits
│   │   ├── CloneEngine.kt           # APK merge, rewrite, notes, signing pipeline
│   │   ├── CloneConfig.kt           # Clone models, validation, suggestions
│   │   ├── CloneStore.kt            # Local clone metadata storage
│   │   ├── KeystoreManager.kt       # Persistent key, export and import
│   │   ├── ManifestNaming.kt        # Manifest identifier rewrite rules
│   │   ├── AppRepository.kt         # Installed-app discovery and cache
│   │   ├── AppIcons.kt              # Lazy icon loading and LRU cache
│   │   ├── AppListAdapter.kt        # Main list adapter
│   │   ├── AppPickerAdapter.kt      # Grid picker adapter
│   │   ├── HistoryAdapter.kt        # History list adapter
│   │   ├── BottomNavView.kt         # Shared bottom navigation
│   │   ├── RingProgressView.kt      # Circular progress indicator
│   │   └── InstalledApp.kt          # Installed-app model
│   └── res/
│       ├── layout/                  # One layout per screen, plus dialogs and items
│       ├── drawable/                # Icons, backgrounds, progress rings
│       ├── values/                  # Strings (Indonesian UI), colours, themes
│       ├── xml/                     # FileProvider paths
│       └── mipmap-*/                # Launcher icons
├── app/src/test/java/dev/bill/openclone/
│   ├── CloneConfigTest.kt
│   └── ManifestNamingTest.kt
├── docs/images/                     # README artwork and screenshots
├── gradle/wrapper/
├── build.gradle.kts
├── settings.gradle.kts
└── README.md
```

## Tests

The JVM unit tests cover the pure configuration and manifest-naming rules:

- `CloneConfigTest` — package-name validation, suggested clone label, and package suffix selection.
- `ManifestNamingTest` — permission names, provider authorities, process names, signature-file detection, and idempotent renaming.

They do not replace device testing. Installation behaviour and runtime compatibility still have to be checked on real phones, against the apps being cloned.

## Contributing

Bug reports, documentation fixes, translations, tests, and code changes are welcome.

When reporting a problem, include where possible:

- Android version and device model.
- Source app name and version.
- The clone package name used.
- Steps to reproduce.
- Relevant `adb logcat` output with personal information removed.

For code changes, run the [checks](#run-checks) and describe how the change was tested. Keep changes focused and avoid adding Android permissions or large dependencies without a clear reason.

### Project boundaries

The project intentionally does not implement DEX or native-code patching, signature-check bypasses, Play Integrity bypasses, device-identifier spoofing, or root/Xposed hooks. Features that would require any of these are surfaced as honest warnings instead.

## Credits

Created by **Nabil Deja**.

Built with open-source Android libraries and tools:

- [REAndroid ARSCLib](https://github.com/REAndroid/ARSCLib) — APK resource and manifest processing.
- [AOSP apksig](https://android.googlesource.com/platform/tools/apksig/) — APK signing.
- [Bouncy Castle](https://www.bouncycastle.org/) — certificate and keystore utilities.
- AndroidX, Material Components, Kotlin Coroutines, and JUnit.

See [`app/build.gradle.kts`](app/build.gradle.kts) for exact versions.

## License

OpenClone is released under the [MIT License](LICENSE).

Copyright (c) 2026 Nabil Deja.
