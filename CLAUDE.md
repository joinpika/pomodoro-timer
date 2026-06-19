# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Desktop Pomodoro timer — a single, zero-dependency HTML file (`pomodoro.html`) that opens directly in a browser. No build step, no package manager, no server needed.

## Opening and testing

- **Open the app**: Double-click `pomodoro.html`, or `start pomodoro.html` on Windows.
- **Quick test cycle**: Open in browser → set work/short/long durations to short values (e.g. 0.1 minutes) via the settings panel → run through start/pause/reset and phase transitions.
- No automated tests exist for this project.

## Architecture

`pomodoro.html` contains everything inline: CSS in a `<style>` block, JS in a `<script>` block at the end of `<body>`.

### State machine

Timer modes: `IDLE → RUNNING → PAUSED` (reset from any state back to IDLE).

Phases cycle: `WORK → SHORT_BREAK` (every 1 work session) or `WORK → LONG_BREAK` (every 4th work session), then back to `WORK`.

### Key JS objects

- `TIMER` — global state object holding `mode`, `phase`, `remainingSeconds`, `pomodoroCount`, `intervalId`, and settings durations.
- `PHASES` — enum: `{ WORK, SHORT_BREAK, LONG_BREAK }`.
- `MODES` — enum: `{ IDLE, RUNNING, PAUSED }`.

### SVG ring progress

Circumference = `2π × 110 ≈ 691.15`. Progress is driven by `stroke-dashoffset`, which decreases from full circumference (time remaining = total) down to 0 (time remaining = 0). The ring is rotated `-90deg` so progress starts at 12 o'clock.

### Key functions

| Function | Role |
|---|---|
| `tick()` | Decrements timer every 1s, updates DOM and SVG ring |
| `startTimer()` / `pauseTimer()` / `resetTimer()` / `resetAll()` | Timer lifecycle |
| `onPhaseComplete()` | Phase transition logic; saves `completedPhase` before switching to preserve correct overlay message |
| `updateRing(seconds, total)` | Maps remaining time to `stroke-dashoffset` |
| `playSound()` | Web Audio API oscillator triad (A5-C#6-E6) |
| `saveToLocal()` / `loadFromLocal()` | localStorage persistence with try/catch guards |

### Persistence

`localStorage` key `pomodoro-state` stores `{ pomodoroCount, settings: { work, shortBreak, longBreak } }`. Read on load, written on every phase completion and settings change.

### Notifications

Uses the browser `Notification` API with `Permission` check. Permission is requested on first user interaction (start button click).

## Git remote

- **Remote**: `https://github.com/joinpika/pomodoro-timer.git`
- **Branch**: `main`
- Push with `git push origin main`. The remote URL is clean (no embedded token).
