# Custom D2 QML setup

This is a custom QML configuration for the D2 based on Traktor Pro 4.4.1.

## It includes the following changes:

Stem Mode — Performance pads:

- pads 1-4: toggle mute on/off per stem (native stem colors, handled per-deck above, inspired by the S5)
- pads: 5-8: Serato style stem FX (see below)
- Shift + pads 1-4: toggle FX send on/off per stem (native stem colors, handled per-deck above, inspired by the S5).
- Shift + pads 5-8: toggle filter on/off per stem slot (blue LEDs, reflect filter_on state).

Serato style stem FX in Stem Mode:

- pad 5: FX Drums Echo pad button to mute and apply a delay+freeze out effect to the drums
- pad 6: FX Instrumental Echo pad to mute and apply a delay+freeze out effect to the instruments (everything but vocals)
- pad 7: Instrumental Breaker FX pad button to mute and apply a turntable FX effect to the instruments (everything but vocals)
- pad 8: FX Vocal Echo pad to mute and apply an delay+freeze out effect to the vocals

Effect will hold as the button will is held. After releasing, pressing the button again sets will unmute the given stem slot again.

Note:

- Stem FX uses FX Unit 4 in Single Mode (Delay T3/Reverb/Turntable FX) set when entering Stem Mode for the first time
- If manual changes were made, pressing the "Remix" button while already in Stem Mode will force a reset of the FX unit
- Configuration of this FX unit will be overridden and FX Unit 4 deck assignment will be cleared as well
- For documentation on how Stem FX work inside Serato see https://support.serato.com/hc/en-us/articles/5700921209615-Using-Stems

Required configuration:

- Delay T3 must be set as the 6th effect in the effect configuration
- Reverb must be set as the 20th effect in the effect configuration
- Turntable FX as the 18th effect in the effect configuration
