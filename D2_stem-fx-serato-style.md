# D2 Serato-Style Stem FX — Group Mode

**Version:** v1.2.3  
**Traktor:** 4.4.1

## Description

This feature maps pads 5-8 to FX Unit 4 when in stem mode, using **Group Mode** with three effect slots: Delay T3 (slot 1) + Reverb (slot 2) + Turntable FX (slot 3).

**Critical:** Pads 5, 7, and 8 both trigger Delay **and** Reverb together (slots 1+2). Pad 6 triggers only Turntable FX (slot 3). Users must verify their effect indices in Traktor match the code variables or adjust the variables accordingly.

## Features

- **Pads 5-8 → FX Unit 4 Group Mode:** Provides Serato-style hold-to-apply effect control per stem.
- **Two Effect Types:**
  - **Pads 5, 7, 8 (Echo):** Route stems through Delay T3 + Reverb simultaneously
  - **Pad 6 (Turntable FX):** Routes stems through Turntable FX only
- **Per-Stem Routing:** Each pad controls different stem combinations:
  - Pad 5: Stem 1 only (Drums)
  - Pad 6: Stems 1+2+3 (Drums, Bass, Melody)
  - Pad 7: Stems 1+2+3 (Drums, Bass, Melody)
  - Pad 8: Stem 4 only (Vocals)
- **Hold-to-Apply Semantics:** Press retains unmuted stems, mute is applied on release; FX teardown resets state.

## Modified Files

- `CSI/Common/Deck_S8Style.qml` (lines 645–3010) - Adds effect index variables, Group Mode init, and per-pad stem FX wiring

---

## Configuration

### MOD Settings

Add this to your main D2.qml controller file:

```qml
// --- MOD SETTINGS: Serato-Style Stem FX ---
MappingPropertyDescriptor {
    id: seratoStemFxEnabledProp
    path: "app.traktor.settings.serato_stem_fx_enabled"
    initialValue: true  // Set to false to disable Serato-style FX
}
// --- END MOD SETTINGS ---
```

Configure effect indices in Deck_S8Style.qml (verify your own):

```qml
readonly property int delayEffectIndex: 7      // Delay T3 (verify in Traktor)
readonly property int reverbEffectIndex: 20    // Reverb (verify in Traktor)
readonly property int turntableFxIndex: 18     // Turntable FX (verify in Traktor)
```

---

## Changes in `CSI/Common/Deck_S8Style.qml`

### Effect Index Variables & Group Mode Init

**Location:** Lines 645–675 in `CSI/Common/Deck_S8Style.qml`

Verify your effect indices in Traktor and update these variables accordingly:

- `sfxDelayEffectIndexGroup: 7` — Delay T3 effect index
- `sfxReverbEffectIndexGroup: 20` — Reverb effect index
- `sfxTurntableFxEffectIndexGroup: 18` — Turntable FX effect index

Use `sfxGroupModeInit()` to configure FX Unit 4 once per deck, setting slots to Group Mode with all three effects.

Use `sfxFxUnitStartGroup(effectIndex, config)` to apply an effect combination to specific stems on pad press.

Use `sfxFxUnitTeardown()` on pad release to reset stem routing.

---

## Pad Implementation Details

### Pad 5: Drums Echo (Delay T3 + Reverb, Stem 1 only)

**Location:** Lines 2885–2910 in `CSI/Common/Deck_S8Style.qml`

- **Press (stem 1 unmuted):** Routes stem 1 through Delay T3 + Reverb; RATE ≈ 0.15, DECAY ≈ 0.35
- **Press (stem 1 muted):** Unmutes stem 1; no FX
- **Release:** Mutes stem 1, tears down FX routing

---

### Pad 6: Instrumental Turntable FX (Turntable only, Stems 1+2+3)

**Location:** Lines 2920–2945 in `CSI/Common/Deck_S8Style.qml`

- **Press (stems 1+2+3 unmuted):** Routes stems 1, 2, 3 through Turntable FX; B.SPD ≈ 0.55
- **Press (all three muted):** Unmutes stems 1, 2, 3; no FX
- **Release:** Mutes stems 1, 2, 3, tears down FX routing

---

### Pad 7: Instrumental Echo (Delay T3 + Reverb, Stems 1+2+3)

**Location:** Lines 2950–2980 in `CSI/Common/Deck_S8Style.qml`

Same as Pad 5, routes stems 1+2+3 (all but vocals).

---

### Pad 8: Vocal Echo (Delay T3 + Reverb, Stem 4 only)

**Location:** Lines 2990–3010 in `CSI/Common/Deck_S8Style.qml`

Same as Pad 5, routes stem 4 only, with higher RATE (0.45 vs 0.15).

---

## Implementation Steps

1. **Verify effect indices in Traktor:**
   - Open Traktor Preferences > Effects > Effect Units
   - Set FX Unit 4 to **Group Mode**
   - Assign: Slot 1 = Delay T3, Slot 2 = Reverb, Slot 3 = Turntable FX
   - Manually select each effect and read values from debug overlay
   - Update variables if different from defaults (7, 20, 18)

2. **Update Deck_S8Style.qml** with the correct effect indices and pad implementations (lines 645–3010).

3. **Restart Traktor** to apply changes.

---

## Testing Checklist

- [ ] Load a stem track and activate stem mode (Remix button)
- [ ] Pad 5: Stem 1 audible with Delay T3 + Reverb echo
- [ ] Pad 6: Stems 1+2+3 audible with Turntable FX breaker
- [ ] Pad 7: Stems 1+2+3 audible with Delay T3 + Reverb echo
- [ ] Pad 8: Stem 4 audible with Delay T3 + Reverb (slower RATE)
- [ ] Release pads: Stems mute immediately; FX state resets
- [ ] Rapid pad presses: No stutter or double-triggering
- [ ] Test on all four decks

---

## Compatibility

### Requires (Prerequisites)

- **D2 Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md))
  - Provides stem mode detection and state management
  - Must be installed first

- **Traktor Pro 4.4.1+** — Group Mode FX Unit 4 required
- **D2 Controller** — CSI/Common/Deck_S8Style.qml architecture
- **FX Unit 4 in Group Mode** — Must be configured with three effect slots

### Works Alongside

- **Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md))
  - Stem Mute uses pads 1-4
  - Serato FX uses pads 5-8
  - No conflict

- **Stem FX Send & Filter** ([D2_stem-fx-send-filter-toggles.md](D2_stem-fx-send-filter-toggles.md))
  - Shift+Pad interaction documented
  - No conflict

### Conflicts With

- **Other FX Unit 4 Workflows** — This feature requires exclusive control of FX Unit 4

### Tested On

- D2 with Traktor Pro 4.4.1
- D2 with Group Mode effects (Delay T3, Reverb, Turntable FX)

### NOT Tested On

- S4 MK3, S8, X1 MK3 (different FX architecture)
- FX Unit 4 in non-Group Mode configurations

---

## Effect Index Verification Guide

Your Traktor installation may have effects in different order. The variables **must** match your actual effect indices:

1. Open Traktor Preferences > Effects > Effect Units Configuration
2. Set FX Unit 4 to **Group Mode** with three slots
3. Assign your chosen effects to each slot
4. In Traktor Mixer, manually select the first effect in Group Mode and read `app.traktor.fx.4.select.1` from debug overlay
5. Repeat for `select.2` and `select.3`
6. Update code variables to match

**Common defaults:** 7 (Delay T3), 20 (Reverb), 18 (Turntable) — verify your own.

---

## Control Layout Reference

| Pad | Effect       | Stems | Button/Knob   | Value     | Purpose            |
| --- | ------------ | ----- | ------------- | --------- | ------------------ |
| 5   | Delay+Reverb | 1     | button1,knob1 | true,0.15 | Delay RATE         |
| 5   | Delay+Reverb | 1     | button2,knob2 | true,0.35 | Reverb DECAY       |
| 6   | Turntable    | 1+2+3 | button3,knob3 | true,0.55 | BRK trigger, B.SPD |
| 7   | Delay+Reverb | 1+2+3 | button1,knob1 | true,0.15 | Delay RATE         |
| 7   | Delay+Reverb | 1+2+3 | button2,knob2 | true,0.35 | Reverb DECAY       |
| 8   | Delay+Reverb | 4     | button1,knob1 | true,0.45 | Delay RATE (warm)  |
| 8   | Delay+Reverb | 4     | button2,knob2 | true,0.35 | Reverb DECAY       |

---

## Notes

- **Critical difference:** Pads 5, 7, 8 simultaneously activate both Delay T3 and Reverb (slots 1+2). Pad 6 activates only Turntable FX (slot 3).
- **LED feedback:** Bright when held (FX active), dim when ready.
- **Effect persistence:** 3-slot config remains until you change Traktor settings; FX state resets per pad release.
- **Turntable B.SPD:** 0.9–1.0 = glitch; 0.6–0.75 = 1-beat; 0.3–0.4 = 1–2 bar classic; 0.45 ≈ 2–4 beats.
- **Shift mode:** Pads 5-8 access filter toggles when Shift is held (see [D2_stem-fx-send-filter-toggles.md](D2_stem-fx-send-filter-toggles.md)).
- **Stem mode only:** This feature is disabled in other pad modes.
