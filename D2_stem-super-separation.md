# D2 Stem Super Separation

**Version:** v1.0.0
**File:** `qml/CSI/Common/Deck_S8Style.qml`

---

## Overview

Shift + FX knob becomes a vocal/instrumental isolation control for stem tracks. Each of the four knobs operates on a different set of decks, and all four follow the configured restore behavior on shift release.

---

## Stem Slot Mapping

| Slot | Stem | Group |
|------|------|-------|
| 1 | Drums | Instrumental |
| 2 | Bass | Instrumental |
| 3 | Other / Melody | Instrumental |
| 4 | Vocal | Vocal |

---

## Knob Assignment

| Knob | Target deck(s) | Formula | On Shift Release / Mode Exit |
|------|---------------|---------|------------------------------|
| FX knob 1 | Focused only | Standard | See `sssRestoreMode` |
| FX knob 2 | Sibling only | Standard | See `sssRestoreMode` |
| FX knob 3 | Other-side only | Reversed | See `sssRestoreMode` |
| FX knob 4 | All 4 decks | Focused standard + others reversed | See `sssRestoreMode` |

**Sibling deck**: the other deck managed by this same controller (A↔C or B↔D).

**Other-side deck**: the deck at the same position on the opposing controller pair — A↔B, C↔D (or B↔A, D↔C).

**Other-sib deck**: the sibling of the other-side deck (the 4th deck). AC focused A→D; AC focused C→B; BD focused B→C; BD focused D→A.

---

## Crossfader Formula

Center (0.5) = all stems at 100%.

### Standard formula (FX knobs 1 and 2, and the focused deck in FX knob 4)

| Direction | Inst (stems 1–3) | Vocal (stem 4) |
|-----------|-----------------|----------------|
| Full left (0.0) | MIN | MAX |
| Center (0.5) | 100% | 100% |
| Full right (1.0) | MAX | MIN |

`inst = min(1.0, knob×2)`, `voc = min(1.0, (1−knob)×2)`

### Reversed formula (FX knob 3, and all secondary decks in FX knob 4)

The opposite direction: where the standard formula fades out instrumentals, the reversed formula fades out vocals instead.

| Direction | Inst (stems 1–3) | Vocal (stem 4) |
|-----------|-----------------|----------------|
| Full left (0.0) | MAX | MIN |
| Center (0.5) | 100% | 100% |
| Full right (1.0) | MIN | MAX |

`inst = min(1.0, (1−knob)×2)`, `voc = min(1.0, knob×2)`

### FX knob 4 combined behavior

| Direction | Focused | Sibling / Other-side / Other-sib |
|-----------|---------|----------------------------------|
| Full left (0.0) | Vocal isolated | Instrumental isolated |
| Center (0.5) | All 100% | All 100% |
| Full right (1.0) | Instrumental isolated | Vocal isolated |

---

## Soft-Takeover

Each stem has independent soft-takeover. A stem is only affected once the knob's target reaches (from above) the stem's current volume. This prevents jumps when stems are below 100% or when the FX knob is not at center when shift is pressed.

**Knob displaced from center at shift press:** The first Wire callback after shift press is absorbed (no volume change), so the user must actively move the knob before anything happens.

**Example:** Drums at 10%, knob at center (0.5). As you turn right (standard formula), `inst = knob×2` increases. Drums are already above 10% at center, so they start moving immediately. But if you turn left, `inst` decreases from 1.0 — drums only start responding once the target drops below 0.1 (at knob ≈ 0.05).

---

## FX Knob Routing

When shift is held, FX unit parameter control (dry_wet / knob 1–3) is suspended for Effect Units 1 and 2. SSS takes exclusive control of the FX knobs. FX routing resumes when shift is released; the existing `SoftTakeoverIndicator` instances handle the re-sync of knob positions to FX parameters.

---

## Configuration

```qml
// Restrict SSS to Stem decks only (true) or allow on any deck type (false).
property bool sssOnlyInStemMode: true

// Restore behavior for all four knobs on shift release / mode exit.
//   "snapshot": restore all modified deck(s) to pre-engagement volumes (default).
//   "fader":    restore secondary deck(s) only; focused deck latches (keeps SSS position).
//               For knob 1 (focused only): no secondary → same as "latch".
//               For knobs 2 and 3 (single secondary): secondary is restored → same as "snapshot".
//               For knob 4 (all 4 decks): sibling, other-side, other-sib restored; focused latches.
//   "latch":    all modified decks latch — no restoration at all.
property string sssRestoreMode: "snapshot"
```

`sssOnlyInStemMode: false` allows the knobs on any deck type (no audible effect on non-stem tracks, but FX parameters are still suspended while shift is held or SSS mode is active).

- `"snapshot"` (default): everything reverts to the state at shift press / mode entry.
- `"fader"`: your focused deck keeps its SSS-adjusted state when using knob 4; secondary decks are cleaned up. For knobs 2 and 3, the single secondary deck is always restored (same as snapshot). For knob 1, no secondary exists so focused latches.
- `"latch"`: all changes on all decks are kept.

---

## StemSuperSeparationMode

Press **Shift+Flux** to enter a persistent SSS mode. While active:

- FX knobs 1–4 perform the vocal/instrumental crossfade **without holding shift**.
- The **FLUX button LED pulsates** to indicate the mode is on.
- All four knobs follow the configured `sssRestoreMode` on exit.

Press **Shift+Flux again** to exit the mode. The configured latch/restore behavior fires on exit, exactly as it would on a normal shift release.

**Note:** While SSS mode is active, the FLUX button's transport function is suspended for shift-presses only. Regular (non-shift) FLUX still works normally.

---

## Interaction with Other Features

- **Stem FX pads (shift+pad escape):** SSS and the sfx pad escape use shift independently. Since you can't physically hold a pad AND turn an FX knob simultaneously, no conflict handling is needed.
- **Stem faders:** SSS writes directly to `app.traktor.decks.N.stems.M.volume`, the same properties the faders control. If you move a fader while SSS is active, it wins (last writer wins). SSS does not restore fader-driven changes.
- **Remix decks:** SSS checks deck type and skips non-stem decks when `sssOnlyInStemMode: true`.
