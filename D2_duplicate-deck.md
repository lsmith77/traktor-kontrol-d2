# D2 Duplicate Deck

**Version:** v0.1.0
**Traktor:** 4.4.2

## Description

Binds the **Edit** button to duplicate the focused deck to the sister deck in its pair, then automatically splits the stems so the two decks can play in sync as an instrumental+vocal split.

- **AC assignment:** focused on A → copies A into C; focused on C → copies C into A.
- **BD assignment:** focused on B → copies B into D; focused on D → copies D into B.

`duplicate_deck` loads the same track at the exact same playback position (Traktor native behaviour). Both decks are in perfect sync as long as sync is enabled on the target deck.

### Stem split on duplicate

| | Stems 1–3 (Drums / Bass / Melody) | Stem 4 (Vocals) |
|---|---|---|
| **Source deck** (duplicated from) | Muted | Audible |
| **Target deck** (duplicated into) | Audible | Muted |

If the source deck was already playing when Edit was pressed, the target deck starts playing automatically once loaded.

## Features

- **Instant stem split:** No manual pad-press required — instruments and vocals separate automatically.
- **Auto-play:** If source is running, target starts playing immediately after load.
- **LED feedback:** Edit button is bright while the opposing deck is playing, dim when idle.
- **Stop on second press:** If the opposing deck is already playing, pressing Edit stops it and re-enables all stem slots on both decks.
- **Pair-aware:** Automatically resolves source/target from the current deck assignment and pad focus.
- **No conflict with beatgrid Edit mode:** The standard beatgrid Edit wire is guarded with `padsMode.value != stemMode`, so in stem mode the Edit button exclusively duplicates; outside stem mode it still opens beatgrid edit as usual.
- **No conflict with Capture Freeze:** Capture Freeze uses the Capture button; this feature uses Edit.

## Modified Files

- `qml/CSI/Common/Deck_S8Style.qml` — Adds state properties, AppProperty instances for duplicate triggers and dynamic target deck bindings, extends `onDeckLoaded` for post-load setup, inserts the Edit duplicate WiresGroup, and adds `&& padsMode.value != stemMode` to the beatgrid Edit wire.

---

## Configuration

### Scope (Stem Mode Only vs Always)

```qml
// In Deck_S8Style.qml, find this line near the top:
property bool duplicateDeckOnlyInStemMode: true
```

- **`true` (default)**: Edit duplicates the deck only when the pads are focused on a Stem deck (stemMode). Outside stem mode the Edit button continues to trigger beatgrid edit.
- **`false`**: Edit always duplicates the deck regardless of pad mode. **Note:** when `false`, beatgrid Edit mode is unavailable in stem mode (it was already guarded by `padsMode != stemMode`), and is unavailable in all other pad modes too since Edit duplicates instead.

---

## Changes in `qml/CSI/Common/Deck_S8Style.qml`

### New Properties

```qml
// After sfxCaptureFreezeOnlyInStemMode
property bool duplicateDeckOnlyInStemMode: true

// State: target deck awaiting post-load mute+play setup (0 = none pending)
property int  duplicateDeckPendingTargetId:   0
property bool duplicateDeckSourceWasRunning:  false

// Live opposing deck ID — the sister deck of whichever deck currently has pad focus.
// Uses padsFocus (not deckFocus) since stem mode is pad-focused; the two can diverge.
readonly property int  duplicateDeckTargetId: padsFocus.value ? topDeckId : bottomDeckId

// Whether the opposing deck is currently running, derived from the static deckXRunning AppProperties.
// Avoids the async stale-value race that AppProperties with dynamic path bindings can have on rebind.
readonly property bool duplicateDeckOpposingRunning:
  duplicateDeckTargetId == 1 ? deckARunning.value :
  duplicateDeckTargetId == 2 ? deckBRunning.value :
  duplicateDeckTargetId == 3 ? deckCRunning.value : deckDRunning.value
```

### New AppProperties (after sfxStem4FxSendOn)

```qml
// Duplicate deck triggers
AppProperty { id: dupDeck3From1; path: "app.traktor.decks.3.track.duplicate_deck.1" }  // AC: A→C
AppProperty { id: dupDeck1From3; path: "app.traktor.decks.1.track.duplicate_deck.3" }  // AC: C→A
AppProperty { id: dupDeck4From2; path: "app.traktor.decks.4.track.duplicate_deck.2" }  // BD: B→D
AppProperty { id: dupDeck2From4; path: "app.traktor.decks.2.track.duplicate_deck.4" }  // BD: D→B

// Dynamic bindings for the target deck — paths update when duplicateDeckPendingTargetId changes.
// Used by onDeckLoaded to apply mutes and start playback after the load completes.
AppProperty { id: dupTargetStem1Muted; path: "app.traktor.decks." + duplicateDeckPendingTargetId + ".stems.1.muted" }
AppProperty { id: dupTargetStem2Muted; path: "app.traktor.decks." + duplicateDeckPendingTargetId + ".stems.2.muted" }
AppProperty { id: dupTargetStem3Muted; path: "app.traktor.decks." + duplicateDeckPendingTargetId + ".stems.3.muted" }
AppProperty { id: dupTargetStem4Muted; path: "app.traktor.decks." + duplicateDeckPendingTargetId + ".stems.4.muted" }
AppProperty { id: dupTargetPlay;       path: "app.traktor.decks." + duplicateDeckPendingTargetId + ".play" }

// Per-deck play/pause — used to stop the opposing deck on second Edit press.
// Static paths (no dynamic binding) to avoid stale-value issues with path rebinding.
// Path: app.traktor.decks.N.play (bool) — true = playing, false = stopped.
AppProperty { id: deckAPlay; path: "app.traktor.decks.1.play" }
AppProperty { id: deckBPlay; path: "app.traktor.decks.2.play" }
AppProperty { id: deckCPlay; path: "app.traktor.decks.3.play" }
AppProperty { id: deckDPlay; path: "app.traktor.decks.4.play" }

// Target-deck stem mutes bound to duplicateDeckTargetId — stable before onPress fires so no
// rebinding race.  Used to re-enable all stem slots on the target deck on the second Edit press.
AppProperty { id: dupStopTargetStem1Muted; path: "app.traktor.decks." + duplicateDeckTargetId + ".stems.1.muted" }
AppProperty { id: dupStopTargetStem2Muted; path: "app.traktor.decks." + duplicateDeckTargetId + ".stems.2.muted" }
AppProperty { id: dupStopTargetStem3Muted; path: "app.traktor.decks." + duplicateDeckTargetId + ".stems.3.muted" }
AppProperty { id: dupStopTargetStem4Muted; path: "app.traktor.decks." + duplicateDeckTargetId + ".stems.4.muted" }
```

### `onDeckLoaded` Extension

```qml
// Post-duplicate setup: apply stem split and auto-play when the target deck finishes loading.
if (deckId > 0 && deckId == duplicateDeckPendingTargetId)
{
  dupTargetStem1Muted.value = false
  dupTargetStem2Muted.value = false
  dupTargetStem3Muted.value = false
  dupTargetStem4Muted.value = true
  if (duplicateDeckSourceWasRunning)
  {
    dupTargetPlay.value = true
  }
  duplicateDeckPendingTargetId = 0
}
```

### Beatgrid Edit Wire Guard

Added `&& padsMode.value != stemMode` so the Edit button is freed for duplicate in stem mode:

```qml
Wire {
  from: "%surface%.edit"
  to: ButtonScriptAdapter { ... }
  enabled: hasEditMode(focusedDeckType) && module.screenView.value == ScreenView.deck && padsMode.value != stemMode
}
```

### Duplicate Deck WiresGroup

Placed as a sibling of the outer stem FX WiresGroup (not nested inside it):

```qml
WiresGroup
{
  enabled: (duplicateDeckOnlyInStemMode ? padsMode.value == stemMode : true)

  Wire
  {
    from: "%surface%.edit"
    to: ButtonScriptAdapter
    {
      brightness: duplicateDeckOpposingRunning ? onBrightness : dimmedBrightness
      onPress:
      {
        if (duplicateDeckOpposingRunning)
        {
          // Opposing deck is playing — stop it by setting app.traktor.decks.N.play to false.
          switch (duplicateDeckTargetId) {
            case 1: deckAPlay.value = false; break
            case 2: deckBPlay.value = false; break
            case 3: deckCPlay.value = false; break
            case 4: deckDPlay.value = false; break
          }

          // Re-enable all stem slots on both decks (undo the vocal/instrumental split).
          dupStopTargetStem1Muted.value = false
          dupStopTargetStem2Muted.value = false
          dupStopTargetStem3Muted.value = false
          dupStopTargetStem4Muted.value = false
          sfxStem1Muted.value = false
          sfxStem2Muted.value = false
          sfxStem3Muted.value = false
          sfxStem4Muted.value = false
        }
        else
        {
          // Record running state before anything changes.
          switch (padsFocusedDeckId) {
            case 1: duplicateDeckSourceWasRunning = deckARunning.value; break
            case 2: duplicateDeckSourceWasRunning = deckBRunning.value; break
            case 3: duplicateDeckSourceWasRunning = deckCRunning.value; break
            case 4: duplicateDeckSourceWasRunning = deckDRunning.value; break
          }
          // Set pending target so onDeckLoaded knows which deck to finish setting up.
          duplicateDeckPendingTargetId = duplicateDeckTargetId
          // Source: mute instrumentals, keep vocals
          sfxStem1Muted.value = true
          sfxStem2Muted.value = true
          sfxStem3Muted.value = true
          sfxStem4Muted.value = false
          // Trigger the duplicate
          if (decksAssignment == DecksAssignment.AC)
          {
            if (!padsFocus.value) dupDeck3From1.value = true  // A→C
            else                  dupDeck1From3.value = true  // C→A
          }
          else
          {
            if (!padsFocus.value) dupDeck4From2.value = true  // B→D
            else                  dupDeck2From4.value = true  // D→B
          }
        }
      }
    }
  }
}
```

---

## Interaction Summary

| Action                                              | Result                                                          |
| --------------------------------------------------- | --------------------------------------------------------------- |
| Edit (stem mode, opposing deck idle)                | Duplicate + stem split + auto-play; button lights up            |
| Edit (stem mode, opposing deck **playing**)         | Stop opposing deck; re-enable all stems on both decks; button goes dim |
| Edit (outside stem mode)                            | Beatgrid edit mode (unchanged)                                  |
| Capture (stem mode)                                 | Capture Freeze toggle (unchanged)                               |

---

## Compatibility

### Requires (Prerequisites)

- **D2 Serato-Style Stem FX** ([D2_stem-fx-serato-style.md](D2_stem-fx-serato-style.md)) — provides `sfxStem*Muted` AppProperties, `stemMode` constant, and the outer WiresGroup that gates this feature

### Works Alongside

- **Stem Capture Freeze** — no conflict (different buttons)
- **Stem Mute Pads** ([D2_stem-mute-pads.md](D2_stem-mute-pads.md)) — pads 1–4, no conflict
- **Stem FX Pads 5–8** — unaffected
- **Beatgrid Edit mode** — available in all non-stem pad modes as usual

### Conflicts With

- **Beatgrid Edit in stem mode** — ⚠️ By design: in stem mode the Edit button duplicates the deck instead of opening beatgrid edit. Switch pad focus away from the stem deck to use beatgrid edit.
- **Beatgrid Edit (all modes) when `duplicateDeckOnlyInStemMode = false`** — Edit always duplicates, so beatgrid edit is unavailable.

---

## Testing Checklist

- [ ] AC assignment, top deck (A) focused, stem mode, **playing**: Edit → button lights up; C loads at same position; A plays only vocals; C plays only instrumentals; C auto-plays
- [ ] AC assignment, top deck (A) focused, stem mode, **stopped**: Edit → C loads; stems split; C does NOT auto-play; button lights up once C starts playing
- [ ] AC assignment, **C is playing**, Edit pressed again → C stops; button goes dim
- [ ] AC assignment, bottom deck (C) focused, stem mode: Edit → reverse direction (C→A); same LED behaviour
- [ ] BD assignment: same tests on B and D
- [ ] Stem mute pads (1–4) can still override mutes manually after the split
- [ ] Outside stem mode: Edit still opens beatgrid edit as usual; LED is unaffected
- [ ] Capture in stem mode still triggers Capture Freeze (unchanged)
- [ ] Set `duplicateDeckOnlyInStemMode = false`: Edit duplicates outside stem mode too
