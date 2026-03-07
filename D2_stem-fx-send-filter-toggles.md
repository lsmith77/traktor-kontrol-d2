# D2 Stem FX Send & Filter Toggles (Shift+Pads)

**Version:** v0.5.0  
**Traktor:** 4.4.1

## Description

This feature adds shift+pad controls for fine-grained stem manipulation in stem mode. Hold shift and press pads 1-8 to toggle FX send routing and filter on/off for individual stems, providing quick access to per-stem effect chain controls without entering remix mode.

## Features

- **Shift+Pads 1-4: FX Send Toggle:** Hold shift and press pads 1-4 to toggle the FX send on/off for each stem independently.
- **Shift+Pads 5-8: Filter Toggle:** Hold shift and press pads 5-8 to toggle the filter on/off for each stem independently.
- **LED Feedback:** Pads show current state with LED brightness (both stem and shift+pad modes).
- **Per-Stem Control:** Each of the four stems can be routed through effects and filters independently.

## Modified Files

- `qml/CSI/Common/Deck_S8Style.qml` - Adds stem filter properties and shift+pad stem control wiring

---

## Configuration

### MOD Settings

Add this to your main D2.qml controller file:

```qml
// --- MOD SETTINGS: Stem FX Send & Filter Toggles ---
MappingPropertyDescriptor {
    id: stemFxSendFilterEnabledProp
    path: "app.traktor.settings.stem_fx_send_filter_enabled"
    initialValue: true  // Set to false to disable shift+pad FX/filter controls
}
// --- END MOD SETTINGS ---
```

---

## Changes in `qml/CSI/Common/Deck_S8Style.qml`

### Stem Filter Properties

Add these AppProperty declarations after the `stemMode` property definition. These provide read/write access to each stem's filter on/off state:

```qml
// Stem filter properties for pads 5-8 (shift mode)
AppProperty { id: sfxStem1FilterOn; path: "app.traktor.decks." + padsFocusedDeckId + ".stems.1.filter_on" }
AppProperty { id: sfxStem2FilterOn; path: "app.traktor.decks." + padsFocusedDeckId + ".stems.2.filter_on" }
AppProperty { id: sfxStem3FilterOn; path: "app.traktor.decks." + padsFocusedDeckId + ".stems.3.filter_on" }
AppProperty { id: sfxStem4FilterOn; path: "app.traktor.decks." + padsFocusedDeckId + ".stems.4.filter_on" }
```

---

### Restrict Normal Stem Wiring When Shift Is Active

Modify the existing stem mute wiring blocks (pads 1-4 mute, pads 5-8 FX) to disable when shift is pressed. In each deck's `WiresGroup`, update the `enabled:` condition:

```qml
// Before
enabled: padsMode.value == stemMode

// After
enabled: padsMode.value == stemMode && !module.shift
```

This ensures stem mute pads (1-4) and FX pads (5-8) only function when shift is **not** pressed, allowing shift+pads to override with FX send and filter controls.

---

### Add Shift+Pad Stem Controls

Add this wiring block for each deck. Repeat for all four decks, changing `decks.1` to `decks.2`, `decks.3`, and `decks.4`:

```qml
// Stem shift pads: FX send on/off (pads 1-4) + filter on/off (pads 5-8)
WiresGroup
{
  enabled: padsMode.value == stemMode && module.shift

  Wire { from: "%surface%.pads.1"; to: "decks.1.stems.1.fx_send_on" }
  Wire { from: "%surface%.pads.2"; to: "decks.1.stems.2.fx_send_on" }
  Wire { from: "%surface%.pads.3"; to: "decks.1.stems.3.fx_send_on" }
  Wire { from: "%surface%.pads.4"; to: "decks.1.stems.4.fx_send_on" }

  Wire { from: "%surface%.pads.5"; to: ButtonScriptAdapter { color: Color.Blue; brightness: sfxStem1FilterOn.value ? onBrightness : dimmedBrightness; onPress: { sfxStem1FilterOn.value = !sfxStem1FilterOn.value } } }
  Wire { from: "%surface%.pads.6"; to: ButtonScriptAdapter { color: Color.Blue; brightness: sfxStem2FilterOn.value ? onBrightness : dimmedBrightness; onPress: { sfxStem2FilterOn.value = !sfxStem2FilterOn.value } } }
  Wire { from: "%surface%.pads.7"; to: ButtonScriptAdapter { color: Color.Blue; brightness: sfxStem3FilterOn.value ? onBrightness : dimmedBrightness; onPress: { sfxStem3FilterOn.value = !sfxStem3FilterOn.value } } }
  Wire { from: "%surface%.pads.8"; to: ButtonScriptAdapter { color: Color.Blue; brightness: sfxStem4FilterOn.value ? onBrightness : dimmedBrightness; onPress: { sfxStem4FilterOn.value = !sfxStem4FilterOn.value } } }
}
```

**Repeat this block for all four deck sections**, changing `decks.1` to `decks.2`, `decks.3`, and `decks.4` respectively.

**Note:** Pads 1-4 use direct wiring to `stems.X.fx_send_on`, while pads 5-8 use `ButtonScriptAdapter` to provide LED feedback showing current filter state.

---

## Implementation Steps

1. **Add stem filter properties** around line 600 (global, one time).
2. **Modify normal stem wiring** in all four deck sections to add the `&& !module.shift` condition.
3. **Add shift+pad control blocks** after each deck's normal stem wiring (one per deck).
4. **Ensure shift key is mapped** in Traktor CSI configuration (D2 typically has this by default).
5. **Restart Traktor** and test.

---

## Testing Checklist

- [ ] Load a track with stems and activate stem mode
- [ ] **Shift+Pads (shift IS pressed):**
  - [ ] Shift+Pad 1 toggles FX send for stem 1 (check Traktor mixer FX send state)
  - [ ] Shift+Pad 2 toggles FX send for stem 2
  - [ ] Shift+Pad 3 toggles FX send for stem 3
  - [ ] Shift+Pad 4 toggles FX send for stem 4
  - [ ] Shift+Pad 5 toggles filter for stem 1 (LED shows state)
  - [ ] Shift+Pad 6 toggles filter for stem 2 (LED shows state)
  - [ ] Shift+Pad 7 toggles filter for stem 3 (LED shows state)
  - [ ] Shift+Pad 8 toggles filter for stem 4 (LED shows state)
- [ ] Test on all four decks
- [ ] Confirm audio is affected when filters are toggled (filter should remove highs when on)

---

## Compatibility

### Requires (Prerequisites)

- **D2 Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md))
  - Provides stem mode detection
  - Must be installed first

- **Traktor Pro 4.4.1+** — Stem API required
- **D2 Controller with Shift Key** — Mapped by default

### Works Alongside

- **Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md))
  - Uses pads 1-4 for muting
  - Shift+Pads 1-4 toggle FX send (override)
  - No conflict when properly scoped

- **Serato-Style Stem FX** ([D2_stem-fx-serato-style.md](D2_stem-fx-serato-style.md))
  - Uses pads 5-8 for FX effects
  - Shift+Pads 5-8 toggle filters (override)
  - No conflict when properly scoped

### Conflicts With

- **Custom Shift Key Mappings** — If shift key is remapped elsewhere, conflicts may occur

### Tested On

- D2 with Traktor Pro 4.4.1 (default shift key mapping)
- D2 with Traktor Pro 4.5+

### NOT Tested On

- D2 with custom shift key remappings
- S4 MK3, S8, X1 MK3 (different shift architecture)

---

## Shift Key Configuration

The D2 controller has a shift key mapped by default. Verify in Traktor:

1. Open **Preferences > Control Surfaces**
2. Select **Traktor Kontrol D2**
3. Look for the shift key mapping (typically labeled "Shift" or "Mod")
4. Ensure it's mapped to `module.shift` (default configuration)

If the shift key isn't working, check the Traktor error log for QML syntax issues.

---

## LED Feedback Guide

| Pads | Shift NOT Pressed | Shift IS Pressed                                       |
| ---- | ----------------- | ------------------------------------------------------ |
| 1-4  | Stem mute state   | FX send state (no LED)                                 |
| 5-8  | FX effect state   | Filter on/off state (Blue LED, brightness shows state) |

When shift+pads 5-8 are used:

- **Bright Blue LED** = Filter is ON
- **Dim Blue LED** = Filter is OFF

Pads 1-4 in shift mode toggle FX send (direct wiring, minimal LED feedback).

---

## Practical Use Cases

1. **Creative Filter Sweeps:** Toggle filters on/off while mixing stems to create filter sweeps.
2. **Per-Stem FX Routing:** Send some stems through effects and bypass others to create custom FX chains.
3. **Live Remixing:** Quickly toggle effects and filters during live performance without touching the mouse.
4. **Stem Isolation:** Filter individual stems to hear them in isolation or to clean up specific frequency ranges.

---

## Notes

- Shift+Pads only function in stem mode (see [02_API_REFERENCE.md](https://github.com/lsmith77/traktor-kontrol-qml/blob/main/02_API_REFERENCE.md#stem-mode)); in other modes, the shift key is passed through to normal shift behaviors.
- When shift is pressed in stem mode, normal stem mute pads (1-4) and FX pads (5-8) are disabled, replaced by shift+pad controls for FX send and filter toggle.
- Shift+Pad controls for FX send (pads 1-4) use direct wiring with minimal LED feedback, while filter toggles (pads 5-8) use ButtonScriptAdapter for blue LED state indication.
- Filter on/off is a global stem property in Traktor; toggling it affects the stem fully across all operations.
- FX send on/off controls whether the stem goes to the selected FX unit; turning it off bypasses effects for that stem.
- LED feedback on pads 5-8 (filter toggles) provides visual confirmation of current state.
