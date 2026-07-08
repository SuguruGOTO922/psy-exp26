# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository is a single-page jsPsych behavioral experiment (`index.html`) — a shape-judgment reaction-time task ("図形判断実験", Japanese-language UI). There is no build system, package manager, or test suite: the entire experiment (config, stimuli, timeline, styling) lives in one HTML file with an inline `<script>` block. jsPsych and its plugins are loaded from the `unpkg.com` CDN at runtime; there are no local dependencies to install.

## Running / developing

- Open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`) and visit it — a plain `file://` open works too since there's no bundler.
- There is no lint, build, or test command. Verify changes by running the experiment in a browser and clicking/keying through a full pass (entry form → fullscreen → instructions → practice → main blocks → end screen).
- Data upload uses DataPipe (https://pipe.jspsych.org) when `CONFIG.DATAPIPE_EXPERIMENT_ID` is set; if it's empty, results save locally as a CSV via `jsPsych.data.get().localSave(...)` on finish instead.

## Architecture

Everything is in `index.html`, structured as:

1. **`CONFIG` object** — the single place to tune experiment parameters: DataPipe experiment ID, number of blocks/repeats, fixation timing range, feedback duration, and stimulus size/offset. Change experiment design here first.
2. **`STIMULI` array** — defines the shape → correct-response mapping (`circle` → `up`, `square` → `down`). Stimuli are rendered as inline SVG via `makeStimulusSVG`/`stimulusHTML`, sized from `CONFIG.STIM_SIZE` and offset via `CONFIG.STIM_OFFSET_X/Y`.
3. **Response mode branching** — the experiment auto-detects touch vs. keyboard (`use_keyboard`, based on `ontouchstart`/`maxTouchPoints`) but lets the participant override it on the entry form. Two parallel trial definitions exist for every response: `trial_keyboard` (arrow keys) and `trial_button` (on-screen arrow-key-styled buttons positioned bottom-left/bottom-right per handedness via `button_position`). Both write the same `data.chosen`/`data.correct` fields so downstream analysis doesn't need to know which input mode was used.
4. **`response_unit`** — a reusable timeline fragment (fixation → conditional keyboard trial → conditional button trial) used identically in both the practice block and each main block, keeping the two in sync.
5. **Timeline assembly** — built as a flat `timeline` array in execution order: entry/demographic form → fullscreen → multi-page instructions → practice block (with feedback) → "start main experiment" screen → `N_BLOCKS` main blocks (with rest screens between them) → optional DataPipe save step → fullscreen exit → end screen. `sample: { type: "fixed-repetitions", size: N }` on each block controls how many times each `STIMULI` entry repeats per block.
6. **Data properties** — participant/session metadata (participant ID, demographics, input mode, handedness, screen/viewport dimensions, device pixel ratio, user agent) is attached globally via `jsPsych.data.addProperties(...)` so every row in the exported data carries it, rather than being stored separately.

When modifying the experiment design (new stimuli, different timing, additional conditions), prefer editing `CONFIG`/`STIMULI` and the shared `response_unit` over duplicating trial logic between practice and main blocks.
