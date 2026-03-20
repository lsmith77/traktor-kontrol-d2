# D2 Stem Super Separation

**Version:** v1.0.0
**File:** `qml/CSI/Common/Deck_S8Style.qml`

---

## Overview

Shift + FX knob becomes a vocal/instrumental crossfader (superknob) for the current stem track. One knob controls all four stem volumes simultaneously with per-stem soft-takeover.

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

| Knob | Deck | On Shift Release / Mode Exit |
|------|------|------------------------------|
| FX knob 1 | Focused deck | **Latch** — volumes stay |
| FX knob 2 | Sibling deck | **Latch** — volumes stay |
| FX knob 3 | Sibling deck | **Restore** — see `sssRestoreMode` |
| FX knob 4 | Focused deck | **Restore** — see `sssRestoreMode` |

The sibling deck is the other deck in this controller's pair (A↔C or B↔D).

---

## Crossfader Formula

Center (0.5) = all stems at 100%.

| Direction | Effect |
|-----------|--------|
| Turn left  | Fades out **vocal** (stem 4) → isolates instrumental |
| Turn right | Fades out **instrumental** (stems 1–3) → isolates vocal |

Volume targets:
- **Stems 1–3** (drums, bass, other): `min(1.0, (1 − knob) × 2)`
- **Stem 4** (vocal): `min(1.0, knob × 2)`

---

## Soft-Takeover

Each stem has independent soft-takeover. A stem is only affected once the knob's crossfader target reaches (from above) the stem's current volume. This prevents jumps when stems are below 100% or when the FX knob is not at center when shift is pressed.

**Knob displaced from center at shift press:** The first Wire callback after shift press is absorbed (no volume change), so the user must actively move the knob before anything happens.

**Example:** Drums at 10%, knob at center (0.5). As you turn right, `instTarget = (1−knob)×2` decreases from 1.0. Drums only start moving when `instTarget` reaches 0.1 (at knob ≈ 0.95). Before that, drums stay at 10%.

---

## FX Knob Routing

When shift is held, FX unit parameter control (dry_wet / knob 1–3) is suspended for Effect Units 1 and 2. SSS takes exclusive control of the FX knobs. FX routing resumes when shift is released; the existing `SoftTakeoverIndicator` instances handle the re-sync of knob positions to FX parameters.

---

## Configuration

```qml
// Restrict SSS to Stem decks only (true) or allow on any deck type (false).
property bool sssOnlyInStemMode: true

// Restore behavior for knobs 3 (sibling) and 4 (focused) on shift release or mode exit.
//   "snapshot": revert stems to volumes captured at shift press / mode entry (default).
//   "fader":    leave stems at their current positions (volumes stay as SSS set them).
property string sssRestoreMode: "snapshot"
```

`sssOnlyInStemMode: false` allows the knobs on any deck type (no audible effect on non-stem tracks, but FX parameters are still suspended while shift is held or SSS mode is active).

`sssRestoreMode: "fader"` makes the restore knobs (3 and 4) behave like latch knobs — the stem volumes stay exactly where SSS or the hardware faders left them when you release shift or exit SSS mode.

---

## StemSuperSeparationMode

Press **Shift+Flux** to enter a persistent SSS mode. While active:

- FX knobs 1–4 perform the vocal/instrumental crossfade **without holding shift**.
- The **FLUX button LED pulsates** to indicate the mode is on.
- All four knobs still follow the latch/restore rules (knobs 1+2 latch; knobs 3+4 respect `sssRestoreMode`).

Press **Shift+Flux again** to exit the mode. The configured latch/restore behavior fires on exit, exactly as it would on a normal shift release.

**Note:** While SSS mode is active, the FLUX button's transport function is suspended for shift-presses only. Regular (non-shift) FLUX still works normally.

---

## Interaction with Other Features

- **Stem FX pads (shift+pad escape):** SSS and the sfx pad escape use shift independently. Since you can't physically hold a pad AND turn an FX knob simultaneously, no conflict handling is needed.
- **Stem faders:** SSS writes directly to `app.traktor.decks.N.stems.M.volume`, the same properties the faders control. If you move a fader while SSS is active, it wins (last writer wins). SSS does not restore fader-driven changes.
- **Remix decks:** SSS checks deck type and skips non-stem decks when `sssOnlyInStemMode: true`.
