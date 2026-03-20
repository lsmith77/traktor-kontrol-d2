# D2 Serato-Style Stem FX — Dual FX Unit

**Version:** v0.5.0
**Traktor:** 4.4.2

## Description

This feature maps pads 5-8 to two FX units when in stem mode:

- **FX Unit 3 (Single Mode):** Delay + Freeze for delay pads (5, 7, 8)
- **FX Unit 4 (Group Mode):** Beatmasher (slot 1) + Gater (slot 2) + Turntable FX (slot 3) for the turntable pad (6)

Both units are initialized once when entering stem mode and reset when hitting the Remix button in stem mode.

**To move effects to different FX units:** Change `sfxDelayUnit` (default: 3) or `sfxTurntableUnit` (default: 4) in `Deck_S8Style.qml` — all AppProperty paths update automatically.

## Features

- **Pads 5-8 → Dual FX Units:** Serato-style hold-to-apply effect control per stem.
- **Two Effect Types:**
  - **Pads 5, 7, 8 (Delay):** FX Unit 3 single mode — Delay + Freeze (button2)
  - **Pad 6 (Turntable FX):** FX Unit 4 group mode slot 3 — Turntable FX BRK
- **Per-Stem Routing:** Each pad controls different stem combinations:
  - Pad 5: Stem 1 only (Drums)
  - Pad 6: Stems 1+2+3 (Drums, Bass, Melody)
  - Pad 7: Stems 1+2+3 (Drums, Bass, Melody)
  - Pad 8: Stem 4 only (Vocals)
- **Hold-to-Apply Semantics:** Press retains unmuted stems, mute is applied on release; FX teardown resets state.
- **Shift+Release Escape:** Press shift while holding a pad, then release — FX tears down but stems are not muted.

## Modified Files

- `qml/CSI/Common/Deck_S8Style.qml` — Adds dual FX unit properties, init/teardown functions, and per-pad stem FX wiring

---

## Configuration

### FX Unit Assignment

Change the unit numbers at the top of the stem FX block in `Deck_S8Style.qml`:

```qml
readonly property int sfxDelayUnit:     3  // FX unit for single-mode Delay+Freeze (pads 5, 7, 8)
readonly property int sfxTurntableUnit: 4  // FX unit for group-mode Turntable FX (pad 6)
```

Changing either value updates all AppProperty paths automatically — no other edits needed.

### Effect Indices

Verify your Traktor effect indices and update these constants if needed:

```qml
readonly property int sfxDelayEffectIndex: 6   // Delay (single mode, verify in Traktor)
readonly property int sfxBeatmasherIndex:  1   // Beatmasher (group mode slot 1)
readonly property int sfxGaterIndex:       5   // Gater (group mode slot 2)
readonly property int sfxTurntableFxIndex: 18  // Turntable FX (group mode slot 3)
```

---

## Changes in `qml/CSI/Common/Deck_S8Style.qml`

### FX Unit Constants & AppProperties

Two sets of AppProperties — one for the delay unit (single mode) and one for the turntable unit (group mode). Both use `sfxDelayUnit` / `sfxTurntableUnit` constants in their paths for easy reassignment.

### Init / Teardown Functions

- **`sfxInit()`** — Called on stem mode entry. Selects Delay on the delay unit (enabled, DryWet=0). Configures turntable unit to Group Mode with Beatmasher/Gater/Turntable FX (once, guarded by `sfxGroupModeInitialized`).
- **`sfxDelayStart(config)`** — Routes stems through the delay unit, sets DryWet=1, activates Freeze (button2).
- **`sfxTurntableStart(config)`** — Routes stems through the turntable unit, sets DryWet=1, triggers BRK (button3), sets B.SPD (knob3).
- **`sfxTeardown()`** — Resets stem routing and both FX units to dry/idle (units stay enabled for fast re-use).
- **`sfxShutdown()`** — Full teardown + clears all static channel assigns + disables both units. Called on stem mode exit or Remix button reset.

### Remix Button Reset

Pressing the Remix button while in stem mode calls `sfxShutdown()` then `sfxInit()` (with `sfxGroupModeInitialized = false` to force a full turntable unit reinit).

---

## Pad Implementation Details

### Pad 5: Drums Delay (Delay+Freeze, Stem 1 only)

- **Press (stem 1 unmuted):** Routes stem 1 through Delay+Freeze on FX unit `sfxDelayUnit`
- **Press (stem 1 muted):** Unmutes stem 1; no FX
- **Release:** Mutes stem 1, tears down FX state
- **Shift+release:** Tears down FX state; stem 1 stays unmuted

---

### Pad 6: Instrumental Turntable FX (Group Mode Slot 3, Stems 1+2+3)

- **Press (stems 1+2+3 unmuted):** Routes stems 1, 2, 3 through Turntable FX on FX unit `sfxTurntableUnit`; B.SPD ≈ 0.55
- **Press (all three muted):** Unmutes stems 1, 2, 3; no FX
- **Release:** Mutes stems 1, 2, 3, tears down FX state
- **Shift+release:** Tears down FX state; stems 1, 2, 3 stay unmuted

---

### Pad 7: Instrumental Delay (Delay+Freeze, Stems 1+2+3)

Same as Pad 5, routes stems 1+2+3 (all but vocals).

- **Shift+release:** Tears down FX state; stems 1, 2, 3 stay unmuted

---

### Pad 8: Vocal Delay (Delay+Freeze, Stem 4 only)

Same as Pad 5, routes stem 4 only.

- **Shift+release:** Tears down FX state; stem 4 stays unmuted

---

## Implementation Steps

1. **Install the mod** — copy `qml/CSI/Common/Deck_S8Style.qml` to your Traktor folder.

2. **Restart Traktor** — the mod configures both FX units automatically when you enter stem mode (FX unit modes, effect slot assignments, and effect selection are all set by `sfxInit()`; no manual Traktor configuration needed).

3. **If effects don't trigger correctly** — the hardcoded effect indices may not match your Traktor installation. See the [Effect Index Verification Guide](#effect-index-verification-guide) below to check and update the constants in the QML.

---

## Testing Checklist

**Hold-to-apply (existing behaviour)**
- [ ] Load a stem track and activate stem mode (Remix button)
- [ ] Pad 5: Stem 1 audible with Delay+Freeze; mutes on release
- [ ] Pad 6: Stems 1+2+3 audible with Turntable FX (BRK brake); mutes on release
- [ ] Pad 7: Stems 1+2+3 audible with Delay+Freeze; mutes on release
- [ ] Pad 8: Stem 4 audible with Delay+Freeze; mutes on release
- [ ] Remix button in stem mode: FX units fully reinitialize
- [ ] Rapid pad presses: No stutter or double-triggering
- [ ] Test on all four decks

**Shift+release escape (new behaviour)**
- [ ] Hold pad 5 → press shift → release pad: FX stops, stem 1 stays unmuted
- [ ] Hold pad 6 → press shift → release pad: FX stops, stems 1+2+3 stay unmuted
- [ ] Hold pad 7 → press shift → release pad: FX stops, stems 1+2+3 stay unmuted
- [ ] Hold pad 8 → press shift → release pad: FX stops, stem 4 stays unmuted
- [ ] After shift+release: subsequent pad press behaves normally (normal mute on release)

**Filter toggle conflict (verify no spurious toggle)**
- [ ] Hold pad 5 → press shift → release pad: stem 1 filter state unchanged
- [ ] Hold pad 6 → press shift → release pad: stem 2 filter state unchanged
- [ ] Hold pad 7 → press shift → release pad: stem 3 filter state unchanged
- [ ] Hold pad 8 → press shift → release pad: stem 4 filter state unchanged

---

## Compatibility

### Requires (Prerequisites)

- **D2 Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md))
  - Provides stem mode detection and state management
  - Must be installed first

- **Traktor Pro 4.4.2+**
- **D2 Controller** — CSI/Common/Deck_S8Style.qml architecture
- **FX Unit 3 in Single Mode** — Delay effect
- **FX Unit 4 in Group Mode** — Beatmasher + Gater + Turntable FX

### Works Alongside

- **Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md))
  - Stem Mute uses pads 1-4; Serato FX uses pads 5-8 — no conflict

- **Stem FX Send & Filter** ([D2_stem-fx-send-filter-toggles.md](D2_stem-fx-send-filter-toggles.md))
  - Shift+Pad interaction documented — no conflict

### Conflicts With

- **Other workflows using FX Unit 3 or FX Unit 4** — These units are managed exclusively during stem mode

### Tested On

- D2 with Traktor Pro 4.4.2

---

## Effect Index Verification Guide

Your Traktor installation may have effects in a different order. Verify by selecting each effect manually and reading the value from the debug overlay:

- **Delay (single mode):** Set FX Unit 3 to Single Mode, select Delay → read `app.traktor.fx.3.select.1`
- **Beatmasher:** Set FX Unit 4 to Group Mode, assign to slot 1 → read `app.traktor.fx.4.select.1`
- **Gater:** Assign to slot 2 → read `app.traktor.fx.4.select.2`
- **Turntable FX:** Assign to slot 3 → read `app.traktor.fx.4.select.3`

**Known defaults:** Delay=6, Beatmasher=1, Gater=5, Turntable FX=18 — verify your own.

---

## Control Layout Reference

| Pad | FX Unit | Mode   | Effect       | Stems | Control         | Value      | Purpose         |
| --- | ------- | ------ | ------------ | ----- | --------------- | ---------- | --------------- |
| 5   | 3       | Single | Delay+Freeze | 1     | dryWet          | 0.3        | FX mix level    |
| 5   | 3       | Single | Delay+Freeze | 1     | knob1           | 0.7        | TIME            |
| 5   | 3       | Single | Delay+Freeze | 1     | knob2           | 0.0        | FEEDBACK (none) |
| 5   | 3       | Single | Delay+Freeze | 1     | knob3           | 0.6        | DEPTH           |
| 5   | 3       | Single | Delay+Freeze | 1     | button2         | true       | Freeze          |
| 6   | 4       | Group  | Turntable FX | 1+2+3 | knob3, button3  | 0.55, true | B.SPD, BRK      |
| 7   | 3       | Single | Delay+Freeze | 1+2+3 | (same as pad 5) | —          | —               |
| 8   | 3       | Single | Delay+Freeze | 4     | (same as pad 5) | —          | —               |

---

## Notes

- **FX Unit 3** stays in single mode with Delay selected for the entire stem mode session; DryWet is 0 when idle, 0.3 on pad hold (TIME=0.7, FEEDBACK=0, DEPTH=0.6). Adjust knob values in the `sfxDelayStart` calls per pad.
- **FX Unit 4** is configured once in group mode: Beatmasher (slot 1), Gater (slot 2), Turntable FX (slot 3). Pad 6 activates only slot 3 (BRK).
- **LED feedback:** Bright when held (FX active), dim when ready.
- **Turntable B.SPD:** 0.9–1.0 = glitch; 0.6–0.75 = 1-beat; 0.3–0.4 = 1–2 bar classic; 0.55 ≈ 2–4 beats.
- **Shift mode:** Pads 5-8 access filter toggles when Shift is held (see [D2_stem-fx-send-filter-toggles.md](D2_stem-fx-send-filter-toggles.md)).
- **Stem mode only:** This feature is disabled in other pad modes.
