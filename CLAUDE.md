# Frequency Scope: project context

## Origin

Mike (Kotlin/Android developer) looked at the Play Store app "Sound Spectrum Analyzer" by PC Mehanik (`com.pcmehanik.splspectrumanalyzer`) and wanted his own version.

- That app has 1M+ downloads and a 4.3★ rating. It is ad-supported, and its data-safety label says it shares location and app activity with third parties.
- Common complaints in its reviews: non-standard band scale (not 1/3-octave), a dB readout that updates too fast to read, no export, no pink noise, and no mic calibration-file import.

## Decisions so far

- **Detection only.** No tone or noise generator. Mike explicitly doesn't want generation.
- **Web, not native Android.** The goal is the fastest path to a working tool. `AnalyserNode` gives the FFT for free. Native Android is reserved for later, if raw/unprocessed mic input (`AudioSource.UNPROCESSED`) or accurate sub-50 Hz readings become important.
- **Self-contained HTML pages.** No framework and no build step, so they can be served by GitHub Pages directly. Each page inlines its own CSS and JS.
- **Must be served over HTTPS or localhost** for mic access. It can't run as a claude.ai artifact, because artifacts block the microphone.

## Spectrum analyzer (spectrum.html; was index.html until the PWA)

- Log-frequency spectrum on a canvas (20 Hz to min(20 kHz, Nyquist)), y axis −110 to −10 dB, with a peak-hold trace that decays 0.25 dB per frame.
- Readout panel: dominant frequency, note and cents, RMS level in dBFS, peak dB.
- Top-5 peaks table: local maxima above −90 dB, parabolic interpolation, 60-cent separation.
- "Number updates" selector (Live / 4 per sec / 2 per sec, the default / 1 per sec), remembered in localStorage. In the slower modes, power is averaged per bin and RMS over the interval, and peaks are found on the averaged spectrum. The graph always stays live.
- Detection range ("Detect from X to Y Hz"): peaks, the readout and the table only consider that range, and the graph dims what's outside it. Saved in localStorage.
- Sticky readout (on by default): the dominant readout only switches to a different peak (more than 60 cents away) when it is at least 3 dB louder for 2 or more updates and 600 ms or more. Added because a ~75 Hz hum and ~500 Hz sound of similar level made the readout flip between them. The graph dot follows the same source.
- Peaks table always has five fixed rows (dashes when empty) so the layout never jumps.
- Tap/drag cursor readout, Freeze/Resume, FFT size selector, smoothing selector, peak-hold toggle and Clear peaks.
- Screen wake lock while running. Light and dark themes via CSS tokens.

## PWA: "Prana Tools" (index.html launcher)

Mike's wife uses an iPhone, so the app must work on both iOS and Android. It's an installable PWA rather than a native app.

- **Files:** `index.html` is the launcher, with Box, 1:4:2:1 and Custom (linking to `pacer.html#...`), Bhramari and Spectrum. `manifest.webmanifest` (name "Prana Tools", short "Prana", standalone, portrait, app shortcuts), `sw.js`, and `icons/` (192, 512, maskable 512, apple-touch 180). Icons were generated with PIL.
- **Every page** has the same `<!-- pwa --> … <!-- /pwa -->` head block (manifest, theme-color, apple-touch-icon, apple-mobile-web-app-* tags), a Home link in its header, and a one-line service worker registration at the end of `<body>`. Standalone mode has no browser back button, so the Home links are required.
- **Service worker:** network-first for page navigations, so pushes show up as soon as you're online, with the cached copy used offline. Cache-first with background refresh for icons and Google Fonts. **Bump `CACHE` in sw.js (currently prana-v2) when adding or renaming files.** Other PWAs share the `maniacalk.github.io` origin (e.g. mortgage-mate), so they share Cache Storage and localStorage. The activate step only deletes caches prefixed `prana-`, and localStorage keys are prefixed (`fs-`, `bh-`, `bp-`). The manifest `id` `./` resolves to `/frequency-scope/`, which keeps the install separate from other apps on the domain.
- **Share button** at the bottom of the launcher. It uses `navigator.share` (the native share sheet on Android and iOS) with the app's home URL, computed as `new URL('./', location.href)`, plus a title and a one-line description. If sharing isn't available it copies the link to the clipboard, or shows the link if the clipboard is blocked. Cancelling the share sheet does nothing.
- **Launcher install help:** uses Android's `beforeinstallprompt` button when available; otherwise Chrome menu instructions, or Safari Share → Add to Home Screen on iOS. Hidden when already running standalone.
- **iOS fixes:**
  - The silent switch mutes Web Audio, so the pacer sets `navigator.audioSession.type = 'playback'` and, on iOS, loops a tiny silent `<audio>` (data-URI WAV) while running.
  - Safari has no Vibration API, so vibrate options are hidden when `!('vibrate' in navigator)`.
  - Wake lock only works in home-screen apps from iOS 18.4.
  - No platform can keep running with the screen off; that needs native.
- **Tested:** local server and headless Chrome. The service worker caches the shell, all pages load offline, and the only installability error was "in-incognito" (from the test harness). iPhone UA emulation hides vibrate and shows Safari steps. Not yet tested on a real iPhone, and the silent-switch fix can only be verified on a real device.

## Status

Live on GitHub Pages at https://maniacalk.github.io/frequency-scope/ (public repo; launcher at root, spectrum at spectrum.html) and in use on Mike's Android phone in Chrome. Before pushing layout changes, check at phone width (390px, dark mode) in headless Chrome with `--use-fake-device-for-media-stream --use-fake-ui-for-media-stream`.

Lesson learned: body is a flex column, so its children have `flex-shrink:0`. Without that, panels with `overflow:hidden` get squashed and clipped when the page grows taller than the screen.

Graph height is `min(34vh, 300px)`, kept compact at Mike's request.

## Bhramari Coach (bhramari.html)

Mike's actual goal: a coach for Bhramari pranayama (bee breath) that keeps his humming pitch inside a target range he sets.

- **Pitch detection** uses the McLeod normalised square difference function on 2048-sample time-domain frames, 55–600 Hz, at 20 per second. The FFT-peak approach was deliberately avoided: with a hum, the 2nd or 3rd harmonic is often louder than the fundamental, especially on phone mics. A frame counts as voiced only if it is above −50 dBFS with clarity ≥ 0.8. Output is the median of the last 5 estimates.
- **UI:** a big Hz readout with the note, and a status line ("Too low · raise by N Hz", "In range", "Too high"). A log gauge with the target zone, and a 30 s pitch trace coloured by in/out of range over the shaded target band.
- **Session stats:** time, breaths, current hum length and % in range. A breath ends after 400 ms of silence and is only counted if it lasted at least 1 s. The breath table shows the last 8.
- **"Set from my hum"** records 3 s and sets the range to the median ±5%. An optional "Vibrate when out of range" setting triggers after 1.5 s out of range, at most every 2 s. Settings are saved in localStorage under `bh-set`.
- **Known tension:** Bhramari is traditionally done with eyes closed and ears closed with the thumbs, so visual feedback only helps with eyes open. Vibration may be picked up by the mic.
- **Tested** with Node unit tests on synthetic hums whose harmonics are louder than the fundamental, and end to end in headless Chrome using a WAV as the fake mic (`--use-file-for-fake-audio-capture`). Not yet tested by Mike on a real phone.

## Breath Pacer (pacer.html)

A timed pranayama pacer: Box (1:1:1:1), 1:4:2:1, 4-7-8, 1:2 and Custom ratios, multiplied by a "time unit" in seconds. Phases with zero length are skipped. Patterns: Box, 1:4:2:1, 4-7-8 (`p478`, ratio [4,7,8,0], default unit 1 s), 1:2 extended exhale (`ext`, [1,0,2,0], default unit 4 s) and Custom. Each is bookmarkable as `#box`, `#1-4-2-1`, `#4-7-8`, `#1-2` or `#custom` (see the `HASH` map). The segmented control shows 3+2 buttons on phones and 5 across on wide screens. The 1:4:2:1 pattern has a "Skip the final hold" option (saved as `skipFinal`) that makes it 1:4:2. The button label changes to match, and the link becomes `#1-4-2`.

- **No breath detection, deliberately.** A phone mic can't reliably hear quiet nasal breathing or tell an inhale from an exhale, so Mike chose the timed pacer.
- **Timing runs on the AudioContext clock.** Cues are scheduled 3 s ahead every 250 ms, so they stay sample-accurate even when timers are throttled. Pause suspends the context, which freezes its clock. Reset closes the context to drop any cues already scheduled.
- **Sound cues:** one high bell (E5) to inhale, two soft mid bells (D5) for either hold, one low bell (B4) to exhale, and three descending bells at the end. Sound and generation are fine on this page; the "detection only" rule applies to the spectrum analyzer.
- **Visual:** a ring split into phase arcs with a moving marker, plus a "lung" circle that grows on inhale and shrinks on exhale. The centre shows the phase name, a seconds countdown and the next phase.
- **Session:** a length in minutes (0 = open-ended) that always finishes its last full round, with elapsed, rounds and remaining stats, plus optional vibration on phase change. Settings are saved under `bp-set`, with the time unit stored per pattern.
- **Tested:** Node simulation of the scheduler (exact boundaries, no duplicate cues, correct end time for box, 1:4:2:1 and no-hold patterns) and a headless Chrome render at 390px. Not yet tried by Mike on a phone.

## Info overlays (pacer.html, bhramari.html)

Each page has an "i" button that opens a `<dialog>` with bullet points per practice: benefits, oxygen, CO₂, nitric oxide, a caution (on 1:4:2:1), and sources. On the pacer the content follows the selected pattern (box, ratio or custom), stored in an `INFO` object in each page's script. The content deliberately states evidence quality honestly:
- Box breathing didn't beat mindfulness in Balban 2023.
- Bhramari's systematic review covered 6 non-randomised, very-low-quality studies.
- There is no evidence for an "optimal" humming pitch.
- Slow breathing doesn't raise SpO₂ in healthy people.
- Humming raises nasal NO about 15× (Weitzberg & Lundberg 2002).
Keep that standard when editing.

## Possible next steps (not yet requested)

- 1/3-octave band view (bar mode)
- A/C weighting
- Per-band calibration offsets, or importing a mic calibration file (e.g. Dayton iMM-6)
- Snapshot/export (PNG or CSV)
- Spectrogram/waterfall view

## Preferences

Mike wants direct, honest answers ("don't be a yes man") and is happy to be asked clarifying questions.
