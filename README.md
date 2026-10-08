<div align="center">
<img src="fastlane/metadata/android/en-US/images/icon.png" width="140" />

<h1>Spotty</h1>
<p><b>One player for Spotify and YouTube Music.</b></p>

[![Release](https://img.shields.io/github/v/release/prakha194/Spotty?style=for-the-badge)](https://github.com/prakha194/Spotty/releases/latest) [![License](https://img.shields.io/github/license/prakha194/Spotty?style=for-the-badge)](https://github.com/prakha194/Spotty/blob/main/LICENSE) [![Downloads](https://img.shields.io/github/downloads/prakha194/Spotty/total?style=for-the-badge)](https://github.com/prakha194/Spotty/releases)

</div>

Spotty is an Android music client that uses **Spotify for discovery** — search, home and
recommendations — and **streams through YouTube Music**. Kotlin + Jetpack Compose, GPL-3.0.

## Install

Grab **`Spotty.apk`** from [Releases](https://github.com/prakha194/Spotty/releases/latest) and open it on your device (allow install from unknown sources). GitHub is the only supported source.

## Build

Needs **JDK 21** and the Android SDK (`compileSdk 37`).

```bash
git clone https://github.com/prakha194/Spotty.git && cd Spotty
cp local.properties.sample local.properties
./gradlew assembleFossDebug        # default flavour
```

APKs land in `app/build/outputs/apk/<flavour>/<type>/`.

| Flavour | Google Cast | In-app updater |
|---------|-------------|----------------|
| `foss`  | no          | yes |
| `gms`   | yes         | yes |
| `izzy`  | no          | no  |

Build-time config (env or `local.properties`): `LASTFM_API_KEY`, `LASTFM_SECRET`,
`CRASH_REPORT_REPO`, `CRASH_REPORT_TOKEN`, `METROLIST_APP_NAME`, `METROLIST_APPLICATION_ID`.

```bash
./gradlew :app:testFossDebugUnitTest :innertube:test   # tests
./gradlew :app:lintGmsRelease                          # lint
```

## Contribute

Fork → branch → PR against `main`. Keep `main` green — the nightly workflow builds it. Releases are cut by bumping `versionName`/`versionCode` in `app/build.gradle.kts`.

## What's inside

- **Spotify for data** (search, home, recommendations), **YouTube Music for playback** — tracks are matched and cached.
- Optional **Qobuz lossless** (FLAC / Hi-Res) with a silent YouTube fallback.
- Synced lyrics, offline downloads, Listen Together, widgets, Discord presence, SponsorBlock, equalizer, Material 3 theming.
- `app/` — the app · `innertube/` — the YouTube Music API client

## License

A rebranded fork of **[Meld](https://github.com/FrancescoGrazioso/Meld)** (by Francesco Grazioso), itself based on **[Metrolist](https://github.com/MetrolistGroup/Metrolist)** (by Mo Agamy). All three are **GPL-3.0**; attribution is retained in [LICENSE](LICENSE).
