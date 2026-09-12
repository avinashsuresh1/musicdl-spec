# 📄 MusicDL Formal Specification (v1.1.9)

This document defines the formal YAML schema, validation rules, data types, and default values for **MusicDL (Music Definition Language)** projects.

---

## 1. Composition Metadata (`composition.yaml`)

The `composition.yaml` file defines the root parameters of a song. Every MusicDL project folder **MUST** contain a valid `composition.yaml` file.

```yaml
title: "Composition Title"
tempo: 120
root_frequency: 261.63
interval: 100
```

### Schema & Validation Rules
| Field | Type | Required? | Default | Validation / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `title` | `string` | **Yes** | N/A | Must be a non-empty string. |
| `tempo` | `number` | **Yes** | N/A | Must be a positive number ($> 0$). Specified in Beats Per Minute (BPM). |
| `root_frequency` | `number` | **Yes** | N/A | Must be a positive number ($> 0$). Base frequency in Hertz (Hz) corresponding to pitch $0$. |
| `interval` | `number` | No | `100` | Step size in cents ($> 0$). $100\text{ cents} = 1\text{ semitone}$ in 12-Tone Equal Temperament (12-TET). |

---

## 2. Instrument Definitions (`instruments/*.yaml`)

Instrument files live under the `instruments/` directory. Each `.yaml` file defines a unique instrument named after its filename (without extension).

```yaml
octave_shift: -1
harmonics:
  - { z: 1, amplitude: 1.0 }
  - { z: 2, amplitude: 0.5 }
adsr:
  attack: 100
  decay: 200
  sustain: 0.7
  release: 400
```

### Schema & Validation Rules
| Field | Type | Required? | Default | Validation / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `harmonics` | `array` | **Yes** | N/A | Array of harmonic objects `{ z: number, amplitude: number }`. Must contain at least 1 harmonic. |
| `harmonics[].z` | `number` | **Yes** | N/A | Harmonic frequency multiplier ($z > 0$). $1.0$ is fundamental, $2.0$ is 1 octave up, $2.76$ is inharmonic bell partial. |
| `harmonics[].amplitude` | `number` | **Yes** | N/A | Relative amplitude ($0.0 \le \text{amplitude} \le 1.0$). |
| `octave_shift` | `integer` | No | `0` | Integer octave register shift ($\dots, -2, -1, 0, 1, 2, \dots$). Shifts all pitches for this instrument by $12 \times \text{octaveShift}$ semitones. |
| `adsr` | `object` | No | Default envelope | ADSR envelope specification containing `attack`, `decay`, `sustain`, `release`. |
| `adsr.attack` | `number` | No | `10` | Attack time in milliseconds ($\ge 0$). |
| `adsr.decay` | `number` | No | `50` | Decay time in milliseconds ($\ge 0$). |
| `adsr.sustain` | `number` | No | `0.8` | Sustain amplitude level ($0.0 \le \text{sustain} \le 1.0$). |
| `adsr.release` | `number` | No | `100` | Release ring-out time in milliseconds ($\ge 0$). |

---

## 3. Melody Definitions (`melodies/*.yaml`)

Melody files live under the `melodies/` directory. Melodies define strictly sequential note sequences.

```yaml
instrument: flute
loop: true
loop_start: 1.0
loop_end: 3.0
notes:
  - { pitch: 0, duration: 1.0 }
  - { pitch: 4, duration: 0.5 }
  - { pitch: rest, duration: 0.5 }
```

### Schema & Validation Rules
| Field | Type | Required? | Default | Validation / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `instrument` | `string` | **Yes** | N/A | Must match the name of an existing instrument in `instruments/`. |
| `notes` | `array` | **Yes** | N/A | Non-empty array of note objects `{ pitch, duration }`. |
| `notes[].pitch` | `integer` \| `'rest'` | **Yes** | N/A | Integer interval relative to root frequency, or string `'rest'` for silence. |
| `notes[].duration` | `number` | **Yes** | N/A | Duration in beats ($> 0$). |
| `loop` | `boolean` | No | `false` | Whether to repeat the melody continuously to fill song duration. |
| `loop_start` | `number` | No | `0.0` | Beat offset where looping section begins. |
| `loop_end` | `number` | No | Total duration | Beat offset where looping section ends. |

> [!NOTE]
> **Sequential Contract**: Notes inside melodies are strictly sequential and auto-accumulate beat start times. Melody notes **do not contain `offset` keys**.

---

## 4. Chord Definitions (`chords/*.yaml`)

Chord files live under the `chords/` directory. Chords define simultaneous pitch combinations.

```yaml
instrument: piano
pitches: [0, 4, 7]
```

### Schema & Validation Rules
| Field | Type | Required? | Default | Validation / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `instrument` | `string` | **Yes** | N/A | Must match the name of an existing instrument in `instruments/`. |
| `pitches` | `array<integer>` | **Yes** | N/A | Non-empty array of integer pitch intervals played simultaneously. |

---

## 5. Track Definitions (`tracks/*.yaml`)

Track files live under the `tracks/` directory. Tracks place melodies and chords onto the global composition timeline.

```yaml
volume: 0.85
melodies:
  - { name: lead, offset: 0.0 }
chords:
  - { name: c_major, offset: 0.0, duration: 4.0 }
```

### Schema & Validation Rules
| Field | Type | Required? | Default | Validation / Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `volume` | `number` | No | `1.0` | Master track volume level ($0.0 \le \text{volume} \le 1.0$). |
| `melodies` | `array` | No | `[]` | List of melody references (`string` name or `{ name, offset }`). |
| `chords` | `array` | No | `[]` | List of chord placement objects `{ name, offset, duration }`. |

---

## 6. Pitch & Frequency Formulas

Given:
- Pitch interval $p \in \mathbb{Z}$ (plus $12 \times \text{octaveShift}$)
- Root frequency $f_0 \in \mathbb{R}^+$
- Step interval in cents $I \in \mathbb{R}^+$

The output frequency $f$ in Hz is calculated as:
$$f = f_0 \times 2^{\frac{(p + 12 \times \text{octaveShift}) \times I}{1200}}$$
