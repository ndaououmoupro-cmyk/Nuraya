# Nûraya — offline spiritual companion (PWA)

> 🇫🇷 *Nûraya est une application web progressive (PWA) en français : Coran, tafsir, horaires de prière, adhkar, Qibla et suivi quotidien, utilisable hors-ligne. Le code est en JavaScript « vanilla », sans framework ni étape de build.*

A mobile-first Progressive Web App I designed and built end to end: product, UX, data modelling, offline strategy and front-end code. It runs entirely in the browser, with no backend, and stays usable without a connection.

**Live demo:** ``

## Features

- **Quran:** all 114 surahs in Arabic with the French translation (Hamidullah), tafsir Ibn Kathir (EN/AR) and several reciters. Selected verses of every surah are built into the app, and the full text can be downloaded once for complete offline reading.
- **Prayer times:** fetched from the AlAdhan API by city or geolocation, with selectable calculation methods (MWL, UOIF, ISNA, Umm al-Qura, Egypt), plus a Qibla compass.
- **Daily practice:** morning/evening adhkar, a dhikr counter with a synthesised sound (Web Audio, no audio files), daily intention and evening review, custom routines, streak tracking.
- **Learning:** structured courses (aqida, tawhid, sira, prophets, the 99 names, hajj, family education), a zakat calculator and a Quran memorisation planner.
- **Privacy:** data stays in `localStorage`. The app is PIN-locked at startup (PIN stored as a SHA-256 hash, progressive lockout after 5 failed attempts) and asks for consent before storing anything.
- **Personalisation:** several selectable themes, light and dark modes.

## Technical choices

| Concern | Approach |
| --- | --- |
| Framework | None. Plain HTML/CSS/JS, rendered through a single `render()` function driven by a global state object `S`. |
| Offline | `sw.js` precaches the app shell. Quran text and tafsir use *stale-while-revalidate*; prayer times use *network-first* with a cache fallback; audio streams straight from the network. |
| Persistence | `localStorage`, saved periodically and on change. |
| Data | Large static datasets (Quran text, courses, education content) live in `js/data/`, separate from UI logic. |
| Installable | Web app manifest and SVG icon; the app also handles the `beforeinstallprompt` event. |

## What this project demonstrates

I started Nûraya as a single 600 KB HTML file and turned it into a structured, tested project. Along the way I worked on:

- **External data sources.** Combining three public APIs (Quran text, tafsir, prayer times) with different formats and reliability, and choosing a cache strategy for each: stale-while-revalidate for stable content, network-first for time-sensitive prayer times.
- **Offline-first design.** Keeping the app usable without a connection, through a service worker and a downloadable local copy of the Quran text.
- **Data quality.** Validating the datasets automatically (114 surahs, 6,236 verses, 99 names, required fields), so that a content error is caught by a test rather than by a user.
- **Privacy by design.** Keeping user data on the device, asking for consent first, and protecting access with a hashed PIN and a lockout after repeated failures.
- **Safe refactoring.** Splitting a large single-file codebase into separate files and checking, with scripts, that the app behaves exactly as before.

These habits (reliable data ingestion, validation, testing and reproducibility) carry over directly to data and machine learning work.

## Project structure

```
index.html              app shell (markup, splash, bottom navigation)
manifest.webmanifest    PWA manifest
sw.js                   service worker (offline strategy)
css/styles.css          global styles
icons/icon.svg          app icon
js/
  data/                 static datasets (Quran, ILM courses, tarbiya, adhkar + state)
  render.js, home.js    rendering core and home screen
  quran-*.js            reader, surah list, offline download
  prayer-times.js       AlAdhan integration and Qibla
  profil.js, security-and-features.js   settings, PIN, consent
  main.js               startup (must load last)
```

The scripts are classic (non-module) scripts that share one global scope, so **the order of the `<script>` tags in `index.html` matters**.

## Tests

```bash
npm install
npm test
```

The suite uses Node's built-in test runner and loads the real app in [jsdom](https://github.com/jsdom/jsdom). It checks that every script parses, that `index.html` and the service worker reference exactly the files that exist, that the app starts without runtime errors and opens locked, and that the data is consistent (114 surahs totalling 6,236 verses, the 99 names, adhkar fields, HTML escaping). A GitHub Actions workflow runs it on every push.

## Run locally

Service workers need `http://` or `https://`, so opening the file directly will not enable offline mode.

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploy on GitHub Pages

Push the repository, then go to **Settings → Pages**, choose the `main` branch and the `/ (root)` folder. All paths are relative, so the app works from a `/<repo-name>/` sub-path.

## Data sources

Quran text and translation: [AlQuran Cloud](https://alquran.cloud/api). Tafsir: [spa5k/tafsir_api](https://github.com/spa5k/tafsir_api). Prayer times: [AlAdhan](https://aladhan.com/prayer-times-api). Recitation audio: [EveryAyah](https://everyayah.com).

## Limitations and roadmap

- The PIN lock is a client-side privacy feature, not strong security: anyone with access to the device's browser storage can bypass it.
- The "Nûr" AI assistant screen is a placeholder describing a planned feature; it is not implemented yet.
- State and rendering are global and string-template based. Next steps: migrate to ES modules (the inline `onclick` handlers will need to move to event listeners first) and move state handling to a small store.
- The religious content (including the women's fiqh section) was written for personal use and should be reviewed by a qualified source before wider distribution.
- 
Copyright (c) 2026 Oumou N’Daou. All rights reserved.

This source code and its design are published for viewing and evaluation only,
for example by recruiters, employers and collaborators reviewing the author's work.

No permission is granted to copy, modify, redistribute, sublicense, sell or use
this software, in whole or in part, for any other purpose without the prior
written permission of the author.

Third-party content (Quran text and translation, tafsir, recitation audio and
prayer times) is provided by external services (AlQuran Cloud, spa5k/tafsir_api,
EveryAyah, AlAdhan) and remains subject to their own terms. It is not covered
by this notice.

## Author

Oumou N’Daou
