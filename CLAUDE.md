# Frequency Scope: project context

## Origin

Mike (Kotlin/Android developer) looked at the Play Store app "Sound Spectrum Analyzer" by PC Mehanik (`com.pcmehanik.splspectrumanalyzer`) and wanted his own version.

- That app has 1M+ downloads and a 4.3★ rating. It is ad-supported, and its data-safety label says it shares location and app activity with third parties.
- Common complaints in its reviews: non-standard band scale (not 1/3-octave), a dB readout that updates too fast to read, no export, no pink noise, and no mic calibration-file import.

## Decisions so far

- **Detection only.** No tone or noise generator. Mike explicitly doesn't want generation.
- **Web, not native Android.** The goal is the fastest path to a working tool. `AnalyserNode` gives the FFT for free. Native Android is reserved for later, if raw/unprocessed mic input (`AudioSource.UNPROCESSED`) or accurate sub-50 Hz readings become important.
- **Single self-contained `index.html`.** No framework and no build step, so it can be served by GitHub Pages directly.
- **Must be served over HTTPS or localhost** for mic access. It can't run as a claude.ai artifact, because artifacts block the microphone.

## Current features (index.html)

- Log-frequency spectrum on a canvas (20 Hz to min(20 kHz, Nyquist)), y axis −110 to −10 dB, with a peak-hold trace that decays 0.25 dB per frame.
- Readout panel: dominant frequency, note and cents, RMS level in dBFS, peak dB.
- Top-5 peaks table: local maxima above −90 dB, parabolic interpolation, 60-cent separation.
- "Number updates" selector (Live / 4 per sec / 2 per sec, the default / 1 per sec), remembered in localStorage. In the slower modes, power is averaged per bin and RMS over the interval, and peaks are found on the averaged spectrum. The graph always stays live.
- Detection range ("Detect from X to Y Hz"): peaks, the readout and the table only consider that range, and the graph dims what's outside it. Saved in localStorage.
- Sticky readout (on by default): the dominant readout only switches to a different peak (more than 60 cents away) when it is at least 3 dB louder for 2 or more updates and 600 ms or more. Added because a ~75 Hz hum and ~500 Hz sound of similar level made the readout flip between them. The graph dot follows the same source.
- Peaks table always has five fixed rows (dashes when empty) so the layout never jumps.
- Tap/drag cursor readout, Freeze/Resume, FFT size selector, smoothing selector, peak-hold toggle and Clear peaks.
- Screen wake lock while running. Light and dark themes via CSS tokens.

## Status

Live on GitHub Pages at https://maniacalk.github.io/frequency-scope/ (public repo) and in use on Mike's Android phone in Chrome. Before pushing layout changes, check at phone width (390px, dark mode) in headless Chrome with `--use-fake-device-for-media-stream --use-fake-ui-for-media-stream`.

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

## Possible next steps (not yet requested)

- 1/3-octave band view (bar mode)
- A/C weighting
- Per-band calibration offsets, or importing a mic calibration file (e.g. Dayton iMM-6)
- Snapshot/export (PNG or CSV)
- Spectrogram/waterfall view

## Preferences

Mike wants direct, honest answers ("don't be a yes man") and is happy to be asked clarifying questions.
