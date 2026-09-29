# Soft takeover

DJ and console faders are not motorized. The hardware position and the on-screen position diverge whenever the song changes or a preset loads.
Applying the first CC jumps the gain. Soft takeover waits until the hand meets the screen.

Mixxx’s controller layer is the reference behavior for a DJ desk. The rules below are the ones Warrior Lab uses.

## State per control

Remember three things, keyed by device + kind + channel + number:

- whether any byte has arrived
- the previous hardware position (0…1)
- the time the control was last **accepted**

## Decision

1. No byte yet: ignore, remember the value, do not stamp the accept time.
2. Accepted less than 50 ms ago: accept again and stamp now. A fast throw keeps moving after the catch.
3. Else compute `screen - incoming` and `screen - previous`.
   - Signs differ and neither difference is zero: the hand crossed the screen. Accept.
   - Either distance is at most 3/128: the hand is on it. Accept.
   - Otherwise ignore.

Always store the new hardware position, even when the tick is ignored.
Only stamp the accept time when the tick is used. That is what keeps the 50 ms whip attached to a real catch instead of to a parked fader.

## What not to do

- Do not ease the parameter toward the hardware while it is still ignored. The ear hears a fade the hand did not play.
- Do not accept the first tick “if it is close enough”. The first tick is how a recalled mix gets destroyed.
- Do not use pickup mode on an endless encoder. An encoder has no position to meet. Use relative.
- Do not use pickup on a button. Use the button edge.
