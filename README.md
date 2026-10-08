<div align="center">
<img src="fastlane/metadata/android/en-US/images/icon.png" width="160" height="160" style="display: block; margin: 0 auto"/>
<h1>Spotty</h1>
<p>An Android music client that fuses Spotify and YouTube Music</p>

[![Latest release](https://img.shields.io/github/v/release/prakha194/Spotty?style=for-the-badge)](https://github.com/prakha194/Spotty/releases/latest)
[![License](https://img.shields.io/github/license/prakha194/Spotty?style=for-the-badge)](https://github.com/prakha194/Spotty/blob/main/LICENSE)
[![Downloads](https://img.shields.io/github/downloads/prakha194/Spotty/total?style=for-the-badge)](https://github.com/prakha194/Spotty/releases)

</div>

---

Spotty is a Kotlin/Jetpack Compose Android app that uses your Spotify account for
discovery (search, home, recommendations) and streams audio through YouTube Music.
This README is written for people building, running, and contributing to the code.
If you just want the app, grab the APK from the [releases page](https://github.com/prakha194/Spotty/releases).

## Building from source

### Requirements

- **JDK 21** (the build targets Java 21)
- **Android SDK** — `compileSdk 37`; install via Android Studio or `sdkmanager`
- **Git**

### Clone and configure

```bash
git clone https://github.com/prakha194/Spotty.git
cd Spotty
cp local.properties.sample local.properties   # then fill in what you need
```

`local.properties` is git-ignored. Any key you leave blank simply disables that feature
(for example, no `LASTFM_API_KEY` means scrobbling is off).

### Build

```bash
./gradlew assembleFossDebug      # default flavour, debug build
./gradlew assembleGmsRelease     # release, with Google Cast
./gradlew assembleIzzyRelease    # F-Droid-compliant (no Cast, no updater)
```

APKs are written to `app/build/outputs/apk/<flavour>/<type>/`. Release builds are minified
(R8) and resource-shrunk.

### Flavours

| Flavour | Google Cast | In-app updater | Notes |
|---------|-------------|----------------|-------|
| `foss`  | no          | yes            | default |
| `gms`   | yes         | yes            | Cast + MediaRouter |
| `izzy`  | no          | no             | the only F-Droid-compliant build |

### Signing a release

The release build reads its signing config from the environment:
`STORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`, with the keystore at
`app/keystore/release.keystore`. Locally you can also drop a keystore there and set the
same variables.

### Configuration (build-time)

These environment variables (or the matching `local.properties` keys) are read at build time:

| Variable | Purpose | Default |
|----------|---------|---------|
| `METROLIST_APP_NAME` | app display name | `Spotty` |
| `METROLIST_APPLICATION_ID` | `applicationId` | `com.spotty.app` |
| `METROLIST_BUILD_COMMIT` | commit SHA appended to the version name | — |
| `LASTFM_API_KEY` / `LASTFM_SECRET` | Last.fm scrobbling | off |
| `CRASH_REPORT_REPO` | `owner/repo` that receives crash reports | `prakha194/Spotty` |
| `CRASH_REPORT_TOKEN` | token with `issues:write` on that repo | off |

In CI these come from repository **secrets** and **variables** (see `.github/workflows/`).
Without `CRASH_REPORT_TOKEN` the in-app crash reporter disables itself silently.

### Tests and lint

```bash
./gradlew :app:testFossDebugUnitTest :innertube:test   # unit tests
./gradlew :app:lintGmsRelease                          # lint on demand
```

Lint is deliberately not run on every CI build (it never gated a merge); run it locally
before a release.

## Project layout

```
app/                  the Android app — Compose UI, Media3 playback, Room, Hilt, Ktor
innertube/            the YouTube Music (InnerTube) API client
fastlane/             store metadata, icon and screenshots
.github/workflows/    CI — PR builds, nightly, tagged releases
docs/                 project site assets
```

### Architecture in brief

- **Spotify** is used for data only (never audio): GraphQL for playlists, liked songs,
  artist/album details, new releases and search, with REST fallbacks for top tracks and
  artists. Session tokens are derived in-app from a WebView login.
- **Playback** goes through YouTube Music via the InnerTube client. Spotify tracks are
  matched to YouTube equivalents with fuzzy title/artist/duration matching and cached.
- **Lossless (optional)** routes audio through Qobuz's FLAC catalogue, matched by ISRC,
  with a silent fallback to YouTube Music when a track is missing or a resolver is down.
- **Crash reporting** posts sanitized reports to GitHub Issues — device/app metadata and
  stack trace only, no account, playlist or personal data.

## Contributing

1. Fork the repo and branch off `main`.
2. Make your change; keep it focused and follow the existing style.
3. Run the unit tests and lint above.
4. Open a pull request against `main`. CI (`build_pr.yml`) builds the APK and runs tests.

Keep `main` compiling — the nightly workflow builds from it. Releases are cut by bumping
`versionName`/`versionCode` in `app/build.gradle.kts`, which triggers `release.yml`.

## Features

**Spotify integration** — Spotify as search and/or home source, a Spotify-only mode, a
recommendation engine that builds radio-like queues from your taste profile, library sync,
fuzzy Spotify→YouTube matching with manual override, album browsing, and a 3-tier profile
cache (GraphQL → REST → local DB) for instant home-screen loads.

**Lossless audio (experimental)** — optional Qobuz FLAC/Hi-Res streaming with ISRC-matched
tracks, persistent match caching, three independent community resolvers, quality tiers
(AAC 320 / CD / Hi-Res) and automatic YouTube fallback.

**Core player** — background playback, offline downloads and caching, live and synced
lyrics (multiple providers, romanization, AI translation), Listen Together, playlists
(local, YouTube-synced, import/export), home-screen widgets, sleep timer and alarm,
Discord Rich Presence, SponsorBlock, skip-silence, crossfade, equalizer (incl. AutoEQ),
Material 3 theming with dynamic colours, and more.

## FAQ

**Do I need Spotify Premium?** No. Spotty uses Spotify for data only; audio streams from
YouTube Music. A free Spotify account is enough.

**How do I install it?** Download `Spotty.apk` from the [latest release](https://github.com/prakha194/Spotty/releases/latest)
and open it on your device (allow install from unknown sources). GitHub is the only
supported source — APKs elsewhere are not ours.

**My playlists aren't showing after Spotify login.** Enable "Use Spotify for Home" and/or
"Use Spotify for Search" in Settings → Integrations → Spotify, then pull down to refresh.

**Playback is slow to start.** Disable battery optimization for Spotty
(Settings → Apps → Spotty → Battery → Unrestricted) — this is the usual fix.

**Can my account get banned?** Spotify is used read-only; no streaming, no artificial
plays. Using unofficial clients sits outside Spotify's ToS, so the risk is low but
non-zero. YouTube streaming uses the InnerTube API; avoid logging in with Google unless
you need age-restricted content. Use at your own risk.

**I have a bug or feature request.** Open an issue on the
[GitHub repository](https://github.com/prakha194/Spotty/issues) so it can be tracked.

## Attribution & License

Spotty is a rebranded fork of **[Meld](https://github.com/FrancescoGrazioso/Meld)** by
[Francesco Grazioso](https://github.com/FrancescoGrazioso), which is itself a fork of
**[Metrolist](https://github.com/MetrolistGroup/Metrolist)** by
[Mo Agamy](https://github.com/mostafaalagamy). Spotty is maintained by
[prakha194](https://github.com/prakha194).

All three are licensed under the **GNU General Public License v3.0**, and Spotty is
distributed under the same license. The original copyright notices and full license text
are retained in [LICENSE](LICENSE). As the GPL requires, the complete source for Spotty is
available in this repository.

### Credits

- **InnerTune** — [Zion Huang](https://github.com/z-huang) · [Malopieds](https://github.com/Malopieds)
- **OuterTune** — [Davide Garberi](https://github.com/DD3Boh) · [Michael Zh](https://github.com/mikooomich)
- [**Kizzy**](https://github.com/dead8309/Kizzy) — Discord Rich Presence
- [**Better Lyrics**](https://better-lyrics.boidu.dev) — time-synced lyrics
- [**SimpMusic Lyrics**](https://github.com/maxrave-dev/SimpMusic) — lyrics API
- [**metroserver**](https://github.com/MetrolistGroup/metroserver) — Listen Together
- [**MusicRecognizer**](https://github.com/aleksey-saenko/MusicRecognizer) — music recognition

## Disclaimer

This project is not affiliated with, funded, authorized, endorsed by, or in any way
associated with YouTube, Google LLC, Spotify AB, or any of their affiliates. Any
trademarks used belong to their respective owners.
