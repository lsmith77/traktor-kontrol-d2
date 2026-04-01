---
feature: browser-layout
version: 1.1.0
controller: D2
file: qml/CSI/Common/Deck_S8Style.qml
---

# Browser Layout

Opens Traktor's browser layout on the laptop screen when the Browse knob is touched while browse mode is active on the D2. Releasing the knob restores the previous state.

## Behaviour

Touching the **Browse knob** while the D2 screen is in browse mode (i.e. the Browse knob has been pushed) sets `app.traktor.browser.full_screen` to `true`, bringing up the full browser panel in Traktor's GUI on the laptop screen.

When the finger lifts, the browser reverts to whatever state it was in before the touch (open or closed).

- Has no effect when an overlay (e.g. browser warnings) is active
- Has no effect while Shift is held

## Controls summary

| Control        | Condition                            | Action                                              |
| -------------- | ------------------------------------ | --------------------------------------------------- |
| Browse touch   | Browse mode active, no overlay       | Open browser layout on laptop screen                |
| Browse release | Browser was opened by touch          | Restore laptop browser to state before touch        |

## Implementation notes

- `browse.touch` is a **one-shot pulse** signal — there is no corresponding release event. `ButtonScriptAdapter.onRelease` does not fire for it.
- The correct pattern is a `SwitchTimer`: it stays set while touch pulses arrive and resets after a short timeout (200 ms) when the finger lifts.
- `browserFullScreen` (`AppProperty`) must be declared **before** `screenViewProp` in the QML object tree, because `screenViewProp.onValueChanged` references it. QML forward references in signal handlers may not resolve reliably.
- `browserFullScreenActive` (plain `property bool`) tracks whether this feature opened the browser, to avoid restoring when it was already open before the touch.

## AppProperty paths used

| Path                               | Purpose                                             |
| ---------------------------------- | --------------------------------------------------- |
| `app.traktor.browser.full_screen`  | Opens / closes the browser panel in Traktor's GUI   |
