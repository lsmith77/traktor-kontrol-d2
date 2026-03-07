# D2 Stem Mute Pads

**Version:** v0.5.0  
**Traktor:** 4.4.2

## Description

This feature adds S5-style stem mute support to the Traktor Kontrol D2. When a track with stems is loaded, pressing the remix button activates stem mode on the performance pads. In stem mode, pads 1-4 directly toggle the mute state of stems 1-4 using simple wire mappings. This mirrors the workflow on the S5 controller.

## Features

- **Remix Button Activation:** Press the remix button to activate stem mode (mode 5) when stems are loaded on a Stem deck.
- **Direct Stem Muting:** In stem mode, pads 1-4 directly toggle the mute state of stems 1-4 (no menu navigation needed).
- **Stem Mode Detection:** When a Stem deck is detected, the remix button becomes active and shows dim LED (50% brightness) when available, bright when active.
- **S5-Compatible Workflow:** Uses the same stem mode logic as the S5 controller, making the experience consistent.
- **All Four Decks Supported:** Stem muting works on all four D2 decks (A, B, C, D).

## Modified Files

- `qml/CSI/Common/Deck_S8Style.qml` - Adds stem mode property (mode 5), Stem deck type support, remix button behavior, and direct pad-to-stem wiring

---

## Configuration

### MOD Settings

Add this to your main D2.qml controller file:

```qml
// --- MOD SETTINGS: Stem Mute Pads ---
MappingPropertyDescriptor {
    id: stemMutePadsEnabledProp
    path: "app.traktor.settings.stem_mute_pads_enabled"
    initialValue: true  // Set to false to disable stem muting
}
// --- END MOD SETTINGS ---
```

Pass to Deck module:

```qml
Deck {
    id: deck1
    stemMutePadsEnabled: stemMutePadsEnabledProp.value
}
```

---

## Changes in `qml/CSI/Common/Deck_S8Style.qml`

### Stem Mode Property Definition

Add this property after the `remixMode` property definition:

```qml
readonly property int stemMode:     5
```

This defines `stemMode` as mode 5, following the S5's stem mode numbering (modes 0–4 are: disabled, hotcue, loop, freeze, remix).

---

### Stem Deck Type Support

In the `updateDeckPadsMode()` function, add this case after the remix deck type check:

```qml
else if (focusedDeckType == DeckType.Stem)
{
  padsMode.value = stemMode;
  padsFocus.value = deckFocus;
}
```

This handles when the remix button is pressed on a Stem deck: it switches pads to stem mode (mode 5) and focuses the deck for stem controls.

---

### Direct Stem Mute Wiring

Add this wiring block for each deck. Repeat the block for all four decks, changing `decks.1` to `decks.2`, `decks.3`, and `decks.4` respectively:

```qml
// Stem Mute (S5-style: pads 1-4 toggle stem slot mute)
WiresGroup
{
  enabled: padsMode.value == stemMode

  Wire { from: "%surface%.pads.1"; to: "decks.1.stems.1.muted" }
  Wire { from: "%surface%.pads.2"; to: "decks.1.stems.2.muted" }
  Wire { from: "%surface%.pads.3"; to: "decks.1.stems.3.muted" }
  Wire { from: "%surface%.pads.4"; to: "decks.1.stems.4.muted" }
}
```

This is the **core feature**: When pads are in stem mode, each pad directly toggles the mute state of its corresponding stem. The wiring:

- Only activates when `padsMode.value == stemMode`
- Provides direct toggle of `decks.N.stems.N.muted` property (no button script needed)
- Is duplicated for all four decks so any deck can be controlled

---

## Implementation Steps

1. **Add the stem mode property:** After the `remixMode` property, add `readonly property int stemMode: 5`

2. **Add stem deck type check:** In the `updateDeckPadsMode()` function, add a case to detect when the remix button is pressed on a Stem deck.

3. **Add stem mute wiring blocks:** For each of the four decks, add the `WiresGroup` block that maps pads 1-4 to stem mute toggles.

4. **Update remix button wire:** Modify the remix button wire to use `ButtonScriptAdapter` and support both `DeckType.Remix` and `DeckType.Stem`.

5. **Restart Traktor** and test with a stem-enabled track.

---

## Testing Checklist

- [ ] Load a track with stems into a deck
- [ ] Confirm remix button shows available state (dim LED if stem track, bright if remix deck currently active)
- [ ] Press remix button to activate stem mode
- [ ] Pad 1 toggles stem 1 (Drums) mute on/off
- [ ] Pad 2 toggles stem 2 (Bass) mute on/off
- [ ] Pad 3 toggles stem 3 (Melody) mute on/off
- [ ] Pad 4 toggles stem 4 (Vocals) mute on/off
- [ ] Press remix button again to exit stem mode and return to previous mode
- [ ] Switch between top and bottom deck pairs; stem controls work for all decks
- [ ] Load a standard (non-stem) track; remix button shows full brightness, pressing it activates remix mode
- [ ] Mix stem and non-stem tracks on different decks; controls adapt appropriately

---

## Compatibility

### Requires (Prerequisites)

- **Traktor Pro 4.4.2+** — Stem deck type native to Traktor 4.4.2+
- **D2 Controller** — Hardware recognized by Traktor
- **Stem Tracks** — One or more audio tracks with stems loaded

### Works Alongside

- **Serato-Style Stem FX** ([D2_stem-fx-serato-style.md](D2_stem-fx-serato-style.md))
  - Stem Mute uses pads 1-4
  - Serato FX uses pads 5-8
  - No conflict

- **Stem FX Send & Filter** ([D2_stem-fx-send-filter-toggles.md](D2_stem-fx-send-filter-toggles.md))
  - Shift+Pad overrides normal wiring
  - No conflict

### Conflicts With

- **Remix Deck Only Workflows** — This feature only activates in Stem mode; Remix mode unaffected
- **Non-Stem Tracks** — Feature disabled when no stems detected; remix mode functions normally

### Tested On

- D2 with Traktor Pro 4.4.2
- D2 with Traktor Pro 4.5+

### NOT Tested On

- S4 MK3, S8, X1 MK3 (different pad architecture)
- CDJ3000 (no stem support)

---

## Notes

- **Mode Behavior:** In stem mode, only pads 1-4 are used (stem muting). Pads 5-8 are available for other features (like [D2 Serato-Style Stem FX](D2_stem-fx-serato-style.md)) that may depend on this feature.
- **LED Feedback:** Remix button shows full brightness when active, 50% when available but inactive, dim when not available.
- **Native Stem Support:** The `DeckType.Stem` type is a native Traktor feature (since 4.4.2); this feature simply wires the D2 pads to use it.
- **Wiring Philosophy:** Direct toggle wiring (`Wire { from: pads.1; to: stems.1.muted }`) keeps the implementation simple and responsive.
