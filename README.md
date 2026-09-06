# 🎼 MusicDL Specification (`musicdl-spec`)

A language-agnostic YAML format specification, schema guidelines, usage documentation, and reference composition examples for **MusicDL (Music Definition Language)**.

MusicDL allows you to compose music declaratively using plain YAML files defining instruments, melodies, chords, and tracks.

> [!NOTE]
> musicdl-engine is one implementation of this spec.

---

## 📚 Table of Contents
1. [Overview](#-overview)
2. [Specification Structure](#-specification-structure)
3. [Language Components](#-language-components)
   - [Global Setup (`composition.yaml`)](#1-global-setup-compositionyaml)
   - [Code-Defined Instruments (`instruments/`)](#2-code-defined-instruments-instruments)
   - [Melodies (`melodies/`)](#3-melodies-melodies)
   - [Chords (`chords/`)](#4-chords-chords)
   - [Tracks (`tracks/`)](#5-tracks-tracks)
4. [Acoustic Register Shifting & Musical Clefs](#-acoustic-register-shifting--musical-clefs)
5. [Included Sample Compositions](#-included-sample-compositions)
6. [Engine Implementations & Ecosystem](#-engine-implementations--ecosystem)

---

## 📖 Overview

A MusicDL composition folder represents a complete piece of music. The folder is structured into modular YAML files:

```
my-composition/
├── composition.yaml          # Global composition settings (title, tempo, root frequency)
├── instruments/              # Additive synthesis instrument definitions
│   ├── piano.yaml
│   └── flute.yaml
├── melodies/                 # Sequential note definitions
│   └── lead.yaml
├── chords/                   # Simultaneous pitch definitions
│   └── pad_chords.yaml
└── tracks/                   # Composition timeline arrangement & volume mixing
    ├── melody_track.yaml
    └── harmony_track.yaml
```

---

## 🎼 Specification Structure

For a full formal breakdown of all fields, constraints, types, and defaults, see [SPECIFICATION.md](file:///d:/MusicDL/musicdl-spec/SPECIFICATION.md).

### Quick Summary

| Component | Directory / File | Description | Key Fields |
| :--- | :--- | :--- | :--- |
| **Composition** | `composition.yaml` | Song title, BPM tempo, root frequency in Hz, pitch interval in cents | `title`, `tempo`, `root_frequency`, `interval` |
| **Instrument** | `instruments/*.yaml` | Additive synthesis spectrum (harmonics $z$ & amp), ADSR envelope, register shift | `harmonics`, `adsr`, `octave_shift` |
| **Melody** | `melodies/*.yaml` | Sequential note sequences (no offsets in melody definition), optional looping | `instrument`, `notes`, `loop`, `loop_start`, `loop_end` |
| **Chord** | `chords/*.yaml` | Simultaneous pitch blocks | `instrument`, `pitches` |
| **Track** | `tracks/*.yaml` | Timeline positioning (`offset`) & volume mixing of melodies & chords | `volume`, `melodies`, `chords` |

---

## 🎹 Language Components

### 1. Global Setup (`composition.yaml`)
```yaml
title: "My Song"
tempo: 80              # Playback speed in Beats Per Minute (BPM)
root_frequency: 261.63 # Starting root note frequency in Hz (261.63 = Middle C4)
interval: 100          # Step size in cents (100 cents = 1 semitone in 12-TET)
```

### 2. Code-Defined Instruments (`instruments/`)
```yaml
octave_shift: -1       # Shift instrument register by octaves (-1 = 1 octave down)
harmonics:
  - { z: 1, amplitude: 1.0 }   # Fundamental frequency
  - { z: 2, amplitude: 0.5 }   # 1 octave higher
  - { z: 3, amplitude: 0.2 }   # Perfect 5th higher
adsr:
  attack: 150    # Attack fade-in (ms)
  decay: 200     # Decay time (ms)
  sustain: 0.6   # Sustain volume level (0.0 to 1.0)
  release: 600   # Release ring-out (ms)
```

### 3. Melodies (`melodies/`)
```yaml
instrument: flute
loop: true
loop_start: 1.0
loop_end: 3.0
notes:
  - { pitch: 0, duration: 1.0 }    # Pitch 0 = root frequency
  - { pitch: 2, duration: 1.0 }    # Pitch 2 = +2 semitones
  - { pitch: rest, duration: 1.0 } # Rest / silence
```

### 4. Chords (`chords/`)
```yaml
instrument: piano
pitches: [0, 4, 7] # Simultaneous pitches played together (e.g. Major Triad)
```

### 5. Tracks (`tracks/`)
```yaml
volume: 0.8
melodies:
  - { name: lead, offset: 0 }
chords:
  - { name: pad_chords, offset: 0, duration: 3.0 }
```

---

## 🎺 Acoustic Register Shifting & Musical Clefs

In traditional sheet music notation, instruments use **Clefs** (Bass Clef, Treble Clef, $8^{va}$, $8^{vb}$) to write notes in their natural acoustic register. In MusicDL, **`octave_shift`** is the general mathematical primitive:
* `octave_shift: -1` or `-2` $\iff$ **Bass Clef / $8^{vb}$** (Sub Bass, Cello, Tuba)
* `octave_shift: 0` $\iff$ **Alto / Tenor / Treble Clef** (Piano, Guitar, Viola)
* `octave_shift: 1` or `2` $\iff$ **Treble $8^{va}$** (Piccolo, Glockenspiel, High Bells)

---

## 📁 Included Sample Compositions

This repository contains ready-to-play sample compositions in `examples/`:
- `examples/simple-melody`: Multi-instrument test-bench with 10 code-defined instruments.
- `examples/chord-progression`: Warm ambient chord layers with sub-bass synth.
- `examples/grandfather-clock`: Relative chord sequencing with automated clock ticks.
- `examples/silent-night`: Full traditional composition with reusable melody phrases.

---

## 🛠 Engine Implementation

musicdl-engine is one implementation of this spec.
* **[`musicdl-engine`](file:///d:/MusicDL/musicdl-engine)**: Interactive desktop editor, timeline scheduler, real-time Web Audio player, and offline PCM audio synthesizer renderer.
