# Traktor Kontrol D2 - Stem Mode Customization

**Production-ready Stem Mode implementation for Traktor Kontrol D2 (4.4.2).**

This repository contains a fully-featured D2 customization with Serato-inspired Stem Mode controls, designed as a working example of the hybrid layering approach documented in the main [traktor-kontrol-qml](https://github.com/lsmith77/traktor-kontrol-qml) ecosystem.

---

## What This Does

Adds professional Stem Mode support to the D2 by extending `qml/CSI/Common/Deck_S8Style.qml` with three coordinated patches:

### Patch 01: Stem Mute (S5-Style)

- Performance pads auto-switch to Stem Mode when stems are loaded
- Press remix button to activate pad controls

### Patch 02: Serato-Style Stem FX

- Pads 5-8 control two FX units when in Stem Mode
- Pads 5, 7, 8: Delay + Freeze on FX Unit 3 (single mode)
- Pad 6: Turntable FX brake on FX Unit 4 (group mode: Beatmasher + Gater + Turntable FX)
- Familiar workflow for Serato users

### Patch 03: Advanced Controls (Shift+Pads)

- Shift + Pads 1-4: Toggle FX Send on/off per stem
- Shift + Pads 5-8: Toggle Filter on/off per stem
- LED feedback shows current state

### Patch 04: Stem Capture Freeze

- Capture button toggles a persistent Delay+Freeze lock on all four stems
- First press: freeze locked, button lights up
- Second press: freeze released, button goes dark
- Pressing any stem FX pad cancels the freeze instantly
- **Configurable scope**: Works in stem mode by default; set `sfxCaptureFreezeOnlyInStemMode = false` to enable on all decks
- Requires Patch 02 (Serato-Style Stem FX)

---

## Installation

### Option A: All at Once (Simplest)

```bash
# Copy entire customization to Traktor
cp -r qml/ ~/Library/Application\ Support/Native\ Instruments/Traktor\ Pro\ 4.4.2/

# Restart Traktor
```

**Result**: All three patches applied. Complete Stem Mode ready.

### Option B: Patches Only (Most Control)

Use patches/ YAML files with traktor-mod tool:

```bash
traktor-mod apply \
  --base /path/to/traktor-kontrol-qml/qml_4_4_1/ \
  --patch patches/01-stem-mute-pads.yaml \
  --patch patches/02-stem-fx-serato-style.yaml \
  --patch patches/03-fx-send-filter-toggles.yaml
```

**Result**: Same features, installed step-by-step with full visibility.

---

## Key Files

| File                                     | Purpose                                      |
| ---------------------------------------- | -------------------------------------------- |
| `qml/CSI/Common/Deck_S8Style.qml`        | Core modification (all patches applied here) |
| `patches/01-stem-mute-pads.yaml`         | Feature 1: Stem mode support                 |
| `patches/02-stem-fx-serato-style.yaml`   | Feature 2: FX Unit 3+4 routing               |
| `patches/03-fx-send-filter-toggles.yaml` | Feature 3: Shift controls                    |
| `config.yaml`                            | Hybrid mode guide & precedence               |
| `DESIGN_PHILOSOPHY.md`                   | Why this approach                            |

---

## Configuration

**FX Unit 3** — Single Mode, select: Echo

**FX Unit 4** — Group Mode, three slots (in order):

1. Beatmasher
2. Gater
3. Turntable FX

**Set in Traktor**: Preferences > Effects > Effect Units Configuration

To reassign effects to different FX units, change `sfxDelayUnit` and `sfxTurntableUnit` in `qml/CSI/Common/Deck_S8Style.qml`.

---

## Features in Detail

### When Stems Are Loaded

- **Pads 1-4**: Toggle stem mute on/off (shows in mixer)
- **Pads 5, 7, 8**: Delay + Freeze via FX Unit 3 (single mode)
- **Pad 6**: Turntable FX brake via FX Unit 4 (group mode)
- **Capture button**: Toggle persistent Delay+Freeze lock on all four stems
- **All decks**: Auto-detect stems and adapt accordingly

### With Shift Key

- **Shift + Pads 1-4**: Toggle FX Send on/off per stem
- **Shift + Pads 5-8**: Toggle Filter on/off per stem
- **LED feedback**: Shows on/off state

---

## Design & Architecture

This D2 setup is **young and modular**:

- Three feature patches that layer cleanly
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
- Check FX Unit 4 is configured
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

YAML files in `patches/` describe:

- What was changed
- Where in the file
- Why it's needed
- Dependencies between patches

---

## For Musicians (Non-Developers)

You don't need to understand code to use this:

1. **Copy the QML folder** to Traktor
2. **Restart Traktor**
3. **Load a track with stems**
4. **Press the remix button** — pads switch to stem mode
5. **Use pads 5-8** for effects, **pads 1-4** for mute

Hold shift for advanced controls. That's it.

---

## For Developers

See `patches/` YAML files for:

- Exact line numbers affected
- Before/after code comparisons
- Dependencies and requirements
- How patches compose together

The git history shows the evolution:

- Commit 1: Stock Traktor 4.4.2 baseline
- Commits 2-5: Features added incrementally
- Current: All features combined

Fork and extend as you like (MIT licensed).

---

## Integration with Broader Ecosystem

This D2 mod is an example of the **hybrid approach** documented in [traktor-kontrol-qml](https://github.com/lsmith77/traktor-kontrol-qml):

- **Young, modular mods** (like D2): Extract patches, compose features
- **Mature, integrated mods** (like X1): Keep as complete systems

The D2 shows how small-to-medium mods can evolve iteratively. As it matures, core features stay together while new options become patches.

### Learning & Documentation

For understanding how to work with versioned mods, combine them safely, and track updates:

- **[Chapter 09: Mod Documentation Guide](https://github.com/lsmith77/traktor-kontrol-qml/blob/main/09_MOD_DOCUMENTATION_GUIDE.md)** — How to document mods with metadata lock files
- **[Chapter 10: Mod Combination Prompt](https://github.com/lsmith77/traktor-kontrol-qml/blob/main/10_MOD_COMBINATION_PROMPT.md)** — How to combine multiple mods and track versions

These chapters explain the **metadata lock file pattern** (like `package-lock.json` for QML) that lets musicians track which versions of which mods they're using and update incrementally.

---

## Support & Questions

- See [DESIGN_PHILOSOPHY.md](DESIGN_PHILOSOPHY.md) for architecture details
- See [CHANGELOG.md](CHANGELOG.md) for what's planned next
- Check [config.yaml](config.yaml) for hybrid mode explanation
- Review [traktor-kontrol-qml](https://github.com/lsmith77/traktor-kontrol-qml) for broader context
- See [Chapter 09](https://github.com/lsmith77/traktor-kontrol-qml/blob/main/09_MOD_DOCUMENTATION_GUIDE.md) for versioning and metadata lock files

---

**Status**: Stable for Traktor 4.4.2  
**License**: MIT (fork, modify, use freely)  
**Author**: Lukas Kahwe Smith
