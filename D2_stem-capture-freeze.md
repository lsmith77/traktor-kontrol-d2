# D2 Stem Capture Freeze

**Version:** v0.5.0
**Traktor:** 4.4.2

## Description

Overrides the **Capture** button in stem mode to toggle a persistent all-stems Delay+Freeze on the focused deck.

- **First press:** Routes all four stem slots through FX Unit 3 (Delay+Freeze). The freeze locks and the Capture button lights up.
- **Second press:** Releases the freeze and resets the FX unit. Button goes dark.
- Any **stem FX pad** press (pads 5–8) cancels the freeze immediately on press (via `sfxDelayStart`/`sfxTurntableStart`); the button goes dark instantly.

## Features

- **Toggle semantics:** Unlike the hold-to-apply stem FX pads, Capture Freeze stays locked until explicitly released.
- **All stems:** All four stem slots (Drums, Bass, Melody, Vocals) are routed through the delay unit simultaneously.
- **LED feedback:** Capture button is bright (on) while frozen, dark when idle.
- **Safe exit:** Leaving stem mode (or pressing Remix to reset) calls `sfxShutdown()` → `sfxTeardown()`, which resets `sfxCaptureFreezeActive`.

## Modified Files

- `qml/CSI/Common/Deck_S8Style.qml` — Adds `sfxCaptureFreezeActive` state, resets it in `sfxTeardown`, adds Capture Wire inside the stem FX WiresGroup, and guards the Remix-mode capture wire to disable it in stemMode.

---

## Configuration

### Scope (Stem Mode Only vs All Decks)

Control where the Capture Freeze feature is available:

```qml
// In Deck_S8Style.qml, find this line near the top:
property bool sfxCaptureFreezeOnlyInStemMode: true
```

- **`true` (default)**: Capture Freeze only works when in Stem Mode (pads focused on a stem deck)
- **`false`**: Capture Freeze works on any deck (non-stems, Remix decks, etc.)

### Effect Settings

Same Delay+Freeze settings as the stem FX pads:

```qml
sfxDelayStart({ stems: [true, true, true, true], dryWet: 0.3, knob1: 0.7, knob2: 0.0, knob3: 0.6, button2: true })
```

Adjust `dryWet`, `knob1` (TIME), `knob2` (FEEDBACK), `knob3` (DEPTH) to taste.

---

## Changes in `qml/CSI/Common/Deck_S8Style.qml`

### New State Property

```qml
// Near sfxPad8Held
property bool sfxCaptureFreezeActive: false
```

### `sfxDelayStart()` / `sfxTurntableStart()` Reset

```qml
sfxCaptureFreezeActive = false
```

Added at the top of both start functions so the Capture LED goes dark **immediately on pad press**, not deferred to release. `sfxTeardown()` also resets it as a safety net (e.g. direct teardown without a prior start call).

### Capture Wire (inside stem FX WiresGroup)

```qml
Wire
{
  from: "%surface%.capture"
  to: ButtonScriptAdapter
  {
    brightness: sfxCaptureFreezeActive ? onBrightness : dimmedBrightness
    onPress:
    {
      if (sfxCaptureFreezeActive)
      {
        sfxCaptureFreezeActive = false
        sfxTeardown()
      }
      else
      {
        sfxCaptureFreezeActive = true
        sfxDelayStart({ stems: [true, true, true, true], dryWet: 0.3, knob1: 0.7, knob2: 0.0, knob3: 0.6, button2: true })
      }
    }
  }
}
```

### Remix-Mode Capture Guard

The existing Remix-deck capture wire was moved to its own `WiresGroup` with an added `&& padsMode.value != stemMode` guard, preventing the two capture wires from conflicting:

```qml
WiresGroup
{
  enabled: ((topDeckType == DeckType.Remix) || (bottomDeckType == DeckType.Remix)) && padsMode.value != stemMode
  Wire { from: "%surface%.capture"; to: DirectPropertyAdapter { path: propertiesPath + ".capture" } }
}
```

---

## Interaction with Stem FX Pads

| Action                             | Result                                                                                                     |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Capture press (idle)               | Freeze locked on all stems; button lights up                                                               |
| Capture press (frozen)             | Freeze released; button goes dark                                                                          |
| Any stem FX pad press while frozen | `sfxDelayStart`/`sfxTurntableStart` clears `sfxCaptureFreezeActive`; button goes dark immediately on press |
| Stem mode exit                     | `sfxShutdown()` → `sfxTeardown()` resets everything                                                        |

---

## Compatibility

### Requires (Prerequisites)

- **D2 Serato-Style Stem FX** ([D2_stem-fx-serato-style.md](D2_stem-fx-serato-style.md))
  - Provides `sfxDelayStart`, `sfxTeardown`, `sfxShutdown`, the delay AppProperties, and stem mode detection
  - Must be installed first

- **Traktor Pro 4.4.2+**
- **D2 Controller** — CSI/Common/Deck_S8Style.qml architecture
- **FX Unit 3 in Single Mode** — Delay effect (shared with stem FX pads)

### Works Alongside

- **Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md)) — pads 1–4, no conflict
- **Stem FX Send & Filter** ([D2_stem-fx-send-filter-toggles.md](D2_stem-fx-send-filter-toggles.md)) — Shift+Pad, no conflict
- **Stem FX Pads 5–8** — Hold-to-apply; cancels capture freeze on release (by design)

### Conflicts With

- **Remix-deck encoder-capture mode** — ⚠️ _Only when `sfxCaptureFreezeOnlyInStemMode = true` (default)_

  When the pads are focused on a Stem deck (stemMode), the Capture button can no longer trigger encoder-capture mode for a Remix deck on the other slot. The guard only disables the wire that feeds `captureState` (used to switch the encoder into "capture source" mode for capturing audio into Remix slots) — it has **no effect on the Remix button** itself or on entering/exiting stemMode. If you need encoder-capture for the Remix deck, switch pad focus away from the Stem deck first.

  **Workaround**: Set `sfxCaptureFreezeOnlyInStemMode = false` to allow Capture Freeze on all decks, which frees encoder-capture to work independently.

---

## Testing Checklist

- [ ] Stem mode active: press Capture → button lights up, all stems freeze
- [ ] Press Capture again → button goes dark, freeze released
- [ ] Press Capture to freeze, then press pad 5 and release → freeze cancelled, pad LED returns to normal
- [ ] Leave stem mode while frozen → everything resets cleanly
- [ ] Remix-mode Capture still works on non-stem decks
- [ ] Test on all four decks
