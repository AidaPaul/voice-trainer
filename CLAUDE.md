# Voice Training App — Development Guide

## Architecture

This is a **single-file HTML app** (`index.html`). No build step, no framework, no dependencies. Everything — CSS, HTML, and JS — lives in one file. Keep it that way.

## Key Sections (in order of appearance in the file)

1. **CSS** — custom properties in `:root`, dark theme, responsive breakpoints at 600px
2. **HTML** — splash screen, main UI (header, stage nav pills, pitch panel with canvas, stage content area, footer nav)
3. **Note data** — `ALL_NOTES` array (C2–E5), `noteFreq()`, `freqToNote()` with cent offset
4. **Harvard sentences** — `HARVARD` array, 72 lists of 10 sentences each, used in the Sentences stage
5. **Conversation starters** — `STARTERS` array, used in Free Practice stage
6. **Audio engine** — `initAudio()` sets up `AnalyserNode` (fftSize 4096), `detectPitch()` does autocorrelation, `detectPitchFFT()` does FFT peak detection (used in spectrogram mode for performance)
7. **Reference tone** — `startReference()`/`stopReference()`, triangle wave oscillator with volume slider
8. **Pitch graph** — `renderGraph()`, rolling line chart with note grid, target lines/zones, 10-second window
9. **Spectrogram** — `renderSpectrogram()`, Friture-style waterfall with double-width ring buffer, log frequency scale, LUT colormap, time-based column advancement (10-second window)
10. **Custom reference lines** — `customLines` array, right-click to pin, per-line colors, chip list in header
11. **Stage definitions** — `STAGES` array and per-stage constants (`WARMUP_NOTES`, `COUNT_NOTES`, `TARGET_ZONE`, `VOWELS`)
12. **Stage rendering** — `renderCurrentStage()` with a switch on stage ID, each stage has its own timer/substep logic
13. **Main loop** — `mainLoop()` runs via `requestAnimationFrame`, handles pitch detection, UI updates, and graph/spectrogram rendering

## How to Customize the Training Routine

### Change target notes and ranges

Edit these constants near the `STAGE DEFINITIONS` section:

- `WARMUP_NOTES` — array of note names for the warm-up stage (e.g., `['C3','D3','E3']`)
- `COUNT_NOTES` — notes used in the counting exercise
- `TARGET_ZONE` — `{low, high}` note range for blend/sentences/free practice stages
- `VOWELS` — array of `{vowel, word}` objects for the resonance stage

### Change the stages themselves

The `STAGES` array defines the stage list. Each entry has `{id, name, short}`. The `id` is used in the `switch` statement inside `renderCurrentStage()` to produce each stage's UI and logic. To add/remove/reorder stages, update both `STAGES` and the corresponding `case` in `renderCurrentStage()`.

### Change timing

Each stage uses `startStageTimer(seconds)` for its countdown. Search for `startStageTimer` calls to adjust durations. The warm-up uses per-note timers (15 seconds each by default).

### Change sentences

Replace the `HARVARD` array with your own. Structure: array of arrays, where each inner array is a list of sentence strings. The app picks a random list and cycles through sentences.

### Change conversation starters

Replace the `STARTERS` array with your own prompts.

## Performance Notes

- **Pitch detection** — autocorrelation is O(n^2) on ~4096 samples. In spectrogram mode, the app switches to `detectPitchFFT()` which is O(n) using the FFT data already computed for the spectrogram. Do not run autocorrelation in spectrogram mode.
- **Spectrogram rendering** — uses a double-width offscreen canvas as a ring buffer. New columns are written at two positions (Friture trick) so the visible region is always a single contiguous `drawImage` blit. Column advancement is time-based, not frame-based. A pre-computed `spectroBinMap` avoids log/pow math per pixel per frame.
- **LUT** — the spectrogram colormap is a flat `Uint8Array(256*3)`, indexed with bitwise ops in the hot loop. Do not replace with object arrays.

## Style

- Dark theme with CSS custom properties
- Monospace fonts for data display
- No external fonts or assets
- Responsive down to 600px width
