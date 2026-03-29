---
feature: preview-player
version: 1.0.0
controller: D2
file: qml/CSI/Common/Deck_S8Style.qml
---

# Preview Player

Adds browse-knob controls for the Traktor preview player.

## Behaviour

### Load / play / stop toggle

**Shift + Browse knob** controls the preview player via `load_or_play`:

- **Browser open, nothing loaded** → loads the selected track and starts playback
- **Browser open, playing** → stops playback, track stays loaded
- **Browser open, stopped but loaded** → resumes playback

### Browse knob seek (browser view only)

While a preview is playing, turning the **Browse knob** seeks through the preview track instead of scrolling the browser list.

- Each encoder step seeks by 0.01 (1% of track length)
- Resumes normal browser scrolling as soon as the preview is stopped

## Controls summary

| Control             | Condition                        | Action                                           |
| ------------------- | -------------------------------- | ------------------------------------------------ |
| Shift + Browse push | Browser open, preview not loaded | Load selected track into preview player and play |
| Shift + Browse push | Preview loaded (any view)        | Toggle play/stop, track stays loaded             |
| Browse turn         | Browser open, preview loaded     | Seek through preview track                       |
| Browse turn         | Browser open, no preview         | Scroll browser rows / pages (unchanged)          |

## AppProperty paths used

| Path                                               | Purpose                                                        |
| -------------------------------------------------- | -------------------------------------------------------------- |
| `app.traktor.browser.preview_player.is_loaded`     | Gate condition for seek override                               |
| `app.traktor.browser.preview_player.load_or_play`  | Load + play trigger                                            |
| `app.traktor.browser.preview_player.seek`          | Relative seek via browse knob                                  |
| `app.traktor.browser.preview_player.elapsed_time`  | Current playhead position (declared, available for future use) |
| `app.traktor.browser.preview_content.track_length` | Total track duration (declared, available for future use)      |
