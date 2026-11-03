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
- Tap/drag cursor readout, Freeze/Resume, FFT size selector, smoothing selector, peak-hold toggle and Clear peaks.
- Screen wake lock while running. Light and dark themes via CSS tokens.

## Status

The code passes a syntax check. It has **not yet been tested on a real device** with a real microphone. First job: test it on Mike's phone via GitHub Pages and fix whatever breaks.

## Possible next steps (not yet requested)

- 1/3-octave band view (bar mode)
- Slower or averaged level readout (fast/slow response times)
- A/C weighting
- Per-band calibration offsets, or importing a mic calibration file (e.g. Dayton iMM-6)
- Snapshot/export (PNG or CSV)
- Spectrogram/waterfall view

## Preferences

Mike wants direct, honest answers ("don't be a yes man") and is happy to be asked clarifying questions.
