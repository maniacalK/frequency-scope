# Frequency Scope

Three pages:

- `index.html`: real-time spectrum analyzer
- `bhramari.html`: Bhramari (bee breath) coach that keeps your humming pitch inside a target range
- `pacer.html`: timed breathing pacer (box 1:1:1:1, 1:4:2:1, custom) with visual guide and sound cues. Links: `#box`, `#1-4-2-1`, `#custom`

A real-time audio frequency analyzer that runs in the browser. It listens to the microphone and shows:

- A live spectrum graph (log frequency scale, 20 Hz–20 kHz) with a peak-hold trace
- The dominant frequency, its nearest musical note and cents offset
- Overall input level (dBFS)
- A live table of the top 5 peaks
- A tap/drag cursor to read the frequency and level at any point, plus Freeze

It is a single `index.html` with no build step and no dependencies (Google Fonts only, with system fallbacks).

## Running it

The microphone API only works in a secure context (HTTPS or `localhost`). Opening the file from disk usually won't work on a phone.

- **GitHub Pages:** Settings → Pages → Deploy from branch → `main` / root. Open the `https://<user>.github.io/frequency-scope/` link on your phone.
- **Netlify Drop:** drag `index.html` onto app.netlify.com/drop.
- **Local:** `npx serve` and open `http://localhost:3000`. For a phone, use Chrome's `chrome://inspect` port forwarding so it loads as `localhost`.

Tap **Start mic** and allow microphone access.

## How it works

- `getUserMedia` with `echoCancellation`, `noiseSuppression` and `autoGainControl` disabled, mono.
- Web Audio `AnalyserNode`. FFT size is selectable (4096–32768), default 16384 (~2.9 Hz bins at 48 kHz).
- Peaks are local maxima above −90 dB, refined with parabolic interpolation. Peaks within 60 cents of a stronger one are dropped.
- The level is the RMS of the time-domain buffer, in dBFS.

## Limitations

- Levels are dBFS (relative to the mic's full scale), not calibrated SPL.
- Phone mics roll off below ~50 Hz and above ~16 kHz.
- Some browsers, especially on mobile, still apply processing even when asked not to.
