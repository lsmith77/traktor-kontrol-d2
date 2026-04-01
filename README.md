# Traktor Kontrol D2 - Stem Mode Customization

**Production-ready Stem Mode implementation for Traktor Kontrol D2 (4.4.2).**

This repository contains a fully-featured D2 customization with lots of stem specific features, like Serato-inspired Stem Mode controls.

This D2 mod is an example of a **overlay mod** documented in [traktor-kontrol-qml](https://github.com/lsmith77/traktor-kontrol-qml):

---

## What This Does

Adds professional Stem Mode support to the D2 by extending `qml/CSI/Common/Deck_S8Style.qml` with several coordinated patches:

### Patch 01: Stem Mute (S5-Style)

- Performance pads auto-switch to Stem Mode when stems are loaded
- Press remix button to activate pad controls

### Patch 02: Serato-Style Stem FX

- **Pads 5-8** control two FX units when in Stem Mode
- **Pads 5, 7, 8**: Delay + Freeze on FX Unit 3 (single mode)
- **Pad 6**: Turntable FX brake on FX Unit 4 (group mode: Beatmasher + Gater + Turntable FX)
- Familiar workflow for Serato users

### Patch 03: Advanced Controls (Shift+Pads)

- **Shift + Pads 1-4**: Toggle FX Send on/off per stem
- **Shift + Pads 5-8**: Toggle Filter on/off per stem
- LED feedback shows current state

### Patch 04: Stem Capture Freeze

- **Capture button** toggles a persistent Delay+Freeze lock on all four stems
- First press: freeze locked, button lights up
- Second press: freeze released, button goes dark
- Pressing any stem FX pad cancels the freeze instantly
- **Configurable scope**: Works in stem mode by default; set `sfxCaptureFreezeOnlyInStemMode = false` to enable on all deck types
- **Requires** Patch 02 (Serato-Style Stem FX)

### Patch 05: Duplicate Deck

- **Edit button** duplicates the focused deck to the sister deck (A↔C or B↔D)
- Automatically splits stems: source keeps vocals only, target gets instrumentals only
- If source was playing, target auto-plays at the same position for instant in-sync layering
- Edit button LED is bright while the opposing deck is playing, dim when idle
- Second press (while opposing deck is playing): stops the opposing deck instead of duplicating
- **Configurable scope**: Works in stem mode by default; set `duplicateDeckOnlyInStemMode = false` to enable on all deck types
- **Requires** Patch 02 (Serato-Style Stem FX)

### Patch 06: Stem Super Separation

- **FX knob 1**: vocal/instrumental crossfade on **focused deck only** (standard formula)
- **FX knob 2**: same on **sibling deck only** (A↔C or B↔D) — independent of focused
- **FX knob 3**: same on **other-side deck only** (A↔B, C↔D) — **reversed direction**
- **FX knob 4**: all 4 decks simultaneously — focused uses standard, the other three use reversed
- **Standard formula** — turn left: isolates vocal; turn right: isolates instrumental
- **Reversed formula** — turn left: isolates instrumental; turn right: isolates vocal
- Center position (0.5) = all stems at 100% on all affected decks
- **Per-stem soft-takeover**: each stem only responds once the knob reaches its current volume (no sudden jumps when stems are below 100% or knob is off-center)
- All four knobs follow `sssRestoreMode` on shift release: `"snapshot"` (default), `"fader"`, or `"latch"`
- **StemSuperSeparationMode**: Shift+Flux enters a persistent mode; FX knobs perform SSS without holding shift; FLUX LED pulsates; Shift+Flux exits and applies latch/restore; Shift + FX knobs temporarily disables StemSuperSeparationMode to be able to pre-position knobs
- **Configurable scope**: Stem decks only by default; set `sssOnlyInStemMode = false` to enable on all deck types

### Patch 07: Preview Player

While in browse mode (ie. Browse knob has been pushed):

- **Shift + Browse knob push**: cycles `load_or_play` — loads and plays if nothing loaded, stops without unloading if playing, resumes if stopped
- **Browse knob turn** (while preview is playing): seeks through the preview track instead of scrolling the browser list

### Patch 08: Browser Layout

While in browse mode (ie. Browse knob has been pushed):

- **Browse knob touch**: opens Traktor's browser layout on the laptop screen
- **Browse knob release**: restores the laptop browser to the state it was in before the touch

---

## Installation

For complete setup guide, backup/restore options, and detailed instructions, see:
**https://github.com/lsmith77/traktor-kontrol-qml/blob/main/08_SHARING_CHANGES.md**

Quick install using the `traktor-mod` script:

```bash
git clone https://github.com/lsmith77/traktor-kontrol-d2.git
cd traktor-kontrol-d2
traktor-mod
```

---

## Configuration

Make sure to set the effects in the right order in your settings in Traktor: Preferences > Effects > Effect Units Configuration
Or adapt the code as needed to adjust the index values inside `Deck_S8Style.qml` or swap intentionally to use other effects.

**FX Unit 3** — Single Mode: Delay, ie. Delay should be in 6th position

```
  readonly property int sfxDelayEffectIndex:  6   // Delay (single mode on sfxDelayUnit, verify in Traktor)
```

**FX Unit 4** — Group Mode, three slots (in order):

1. Beatmasher (should be in 1st position)
2. Gater (should be in 5th position)
3. Turntable FX (should be in 18th position)

```
  readonly property int sfxBeatmasherIndex:  1   // Beatmasher (group mode slot 1)
  readonly property int sfxGaterIndex:       5   // Gater (group mode slot 2)
  readonly property int sfxTurntableFxIndex: 18  // Turntable FX (group mode slot 3)
```

---

## Design & Architecture

This D2 setup is **young and modular**:

- Several mostly independent feature patches that layer cleanly
- Easy to understand each piece
- Simple to add more features
- Designed for evolution

All modifications are in **qml/CSI/Common/Deck_S8Style.qml** — a single file containing the pad mode logic. See patches/ YAML for exactly what changed and where.

**Git history**: `git log --oneline` shows the development of each feature (commits bd304ba through 47b413c).

---

## Troubleshooting

**Pads not responding to stems?**

- Confirm track has stems (gear icon in Traktor)
- Verify D2 recognized (Preferences > Control Surfaces)
- Check FX Units are configured
- Restart Traktor

**Shift+pads not working?**

- Shift key must be mapped on D2 control surface
- Test in Stem Mode first (normal mode doesn't have shift controls)
- Check Traktor error log for QML syntax issues

**FX Unit pads not triggering effects?**

- Verify FX Unit 3 is in Single Mode with Delay selected
- Verify FX Unit 4 is in Group Mode with Beatmasher/Gater/Turntable FX in slots 1-3
- Check effect indices match (`sfxDelayEffectIndex`, `sfxBeatmasherIndex`, etc.) — verify via debug overlay
- Try assigning effects manually in Traktor, then restart

---

## Development

Each feature is documented in its patch YAML file. To understand exactly what changed:

```bash
# See development history
git log --oneline

# See first feature (stem mute)
git diff bd304ba~1..bd304ba

# See second feature (FX routing)
git diff 34be413~1..34be413

# etc...
```

YAML files each describe:

- What was changed
- Where in the file
- Why it's needed
- Dependencies between patches

---

### Learning & Documentation

For understanding how to work with versioned mods, combine them safely, and track updates:

- **[Chapter 09: Mod Documentation Guide](https://github.com/lsmith77/traktor-kontrol-qml/blob/main/09_MOD_DOCUMENTATION_GUIDE.md)** — How to document mods with metadata lock files
- **[Chapter 10: Mod Combination Prompt](https://github.com/lsmith77/traktor-kontrol-qml/blob/main/10_MOD_COMBINATION_PROMPT.md)** — How to combine multiple mods and track versions

These chapters explain the **metadata lock file pattern** (like `package-lock.json` for QML) that lets musicians track which versions of which mods they're using and update incrementally.

---

**Status**: Stable for Traktor 4.4.2  
**License**: CC0 1.0 Universal
**Author**: Lukas Kahwe Smith
