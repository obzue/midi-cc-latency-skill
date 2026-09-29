---
name: midi-cc-latency
description: >-
  MIDI CC mapping and audio-driver latency for a DAW, DJ deck, or controller.
  Use when the user says MIDI, CC, encoder, jog wheel, soft takeover, pickup,
  MIDI learn, Mackie, relative CC, ASIO, WASAPI, Core Audio, buffer size,
  driver latency, dropouts, or look-ahead. Covers absolute, relative, pickup,
  and button maps, the three relative curves, and how to read a real interface
  buffer without pretending look-ahead is the driver.
license: MIT
metadata:
  author: obzue (synthesized)
  version: "1.0.0"
  category: music
  sources:
    - tttapa/Control-Surface
    - mixxxdj/mixxx
    - djipco/webmidi
    - dechamps/FlexASIO
    - mackron/miniaudio
    - microsoft/low-latency-audio
---

# MIDI CC mapping and driver latency

Load this before changing MIDI learn, jog wheels, or the latency readout.
The studio already speaks this dialect in `src/studio/midi-cc.ts`.
Do not vendor the GPL trees this was studied from. See `UPSTREAM.md`.

## 1. One message, four behaviors

A control-change is status `0xBn`, then CC number, then a 7-bit value.
Channel is the low nibble. Learn stores **device + channel + CC**, not CC alone.
Two controllers often share CC 7. A map that ignores the device will fight itself.

| Mode | Hardware | What the byte means |
| --- | --- | --- |
| Absolute | Fader, pot | `value / 127` is the position |
| Relative | Endless encoder, jog | A **signed step**, not a position |
| Pickup | Fader that can disagree with the screen | Position, but ignored until it meets the screen |
| Button | Pad, transport | Edge. Fire on press, not on every repeat |

Notes are the same table with `kind: "note"`. Note-on velocity 0 is note-off.

Factory CCs stay live until a learned map replaces that exact key.
Do not reschedule the transport from a master-volume CC. Gain only.

## 2. Relative curves

Endless encoders do not send 0–127. They send a step.
The same byte means opposite directions under different curves.
Pick the curve per map. Do not guess from one tick: 65 is +1 on binary offset and −1 on sign-magnitude.

| Curve | Also called | 0 | +1 | −1 |
| --- | --- | --- | --- | --- |
| `twos` | Reaper relative 1, Tracktion | 0 (and treat 64 as 0) | 1 | 127 |
| `offset` | Reaper relative 2 | 64 | 65 | 63 |
| `sign` | Mackie Control, Reaper relative 3 | 0 | 1 | 65 (bit 6 set, magnitude in the low 6 bits) |

Korg inc/dec mode uses **two** CC numbers: positive steps on N, negative steps on N+1. That is not a fourth curve. Map both CCs.

Cap a single tick (the studio uses ±8 on a learned encoder, ±6 on a factory jog) so a wrong curve cannot throw the fader across the mixer.
Sensitivity for a 0–1 target is `step / 32`.

Decode lives in `decodeRelative`. Do not reintroduce a decoder that treats every value under 64 as positive and every value over 64 as two’s complement. That is only the `twos` curve.

## 3. Soft takeover

A fader left at the top will smash the channel if the first CC is applied.
Rules, in order:

1. Drop the first value. Store it. Do not move the parameter.
2. If this control was caught in the last 50 ms, take the new value. That is a whip, not a jump from a parked knob.
3. Otherwise take it only if the new value **crossed** the on-screen value, or either the new or previous value is within **3/128** of it.
4. Same side and both far: ignore. Update the remembered hardware value anyway.

Threshold 3/128 is about 2.3%. Wider than that and the catch is late. Narrower and a coarse 7-bit fader never catches.

## 4. Jog wheels

Factory jog CCs on this desk are 33 / 21 / 48 (deck A) and 34 / 22 / 49 (deck B), decoded as `twos` and capped at ±6.
Pitch bend on the DJ view is a second jog path: 14-bit bend, ignore under ±80, scale by 900, cap ±6.
Scratch and backspin are audio-rate position changes, not a second MIDI mode.
If a wheel feels backwards, change the curve on a **learned** jog map. Do not flip the platter direction and the decoder.

## 5. Driver latency is not look-ahead

Heard delay ≈ scheduler look-ahead + the buffer the device actually opened.

```
periodMs = bufferSamples / sampleRate * 1000
heardMs  = lookAheadMs + max(outputLatency, baseLatency)
```

`outputLatency` and `baseLatency` come from the audio context after it is running.
A requested period is a hint. The driver rounds to its own quantum and clamps to a min/max. Read back the buffer. Do not display the number the user asked for.

| What you measured | Band | What to tell the player |
| --- | --- | --- |
| Context not running | closed | Start the engine before quoting a buffer |
| Under 10 ms | exclusive | ASIO, WASAPI exclusive, or Core Audio. Track at 64–128 samples |
| 10 ms up to 30 ms | shared | OS mixer floor. Look-ahead cannot break it |
| 30 ms and up | legacy | DirectSound / MME class. DirectSound input sits near 31 ms |

Windows shared mode (WASAPI shared, and MME or DirectSound on top of it) keeps about a **10 ms** engine buffer. Only an exclusive open goes under that.
WASAPI exclusive and kernel streaming do. They also take the device away from other apps.
A universal ASIO wrapper is only as low-latency as the backend it opens. Exclusive WASAPI or the interface’s own ASIO driver. Not DirectSound.

128 samples at 48 kHz is 2.7 ms per period. One period is not the whole trip: input + output is often two periods, plus safety.
Look-ahead shorter than one period turns a stall into a click. 10 / 20 / 40 ms in the desk are scheduler settings. They do not resize the card.

macOS: Core Audio is already the low-latency path. Do not invent an ASIO layer.
Linux: JACK or PipeWire in the pro-audio profile. A Pulse default often sits in the shared band. ALSA period count is usually 2 or 3.
Android: AAudio exclusive, not the Java AudioTrack path.
iOS: AVAudioSession `.playAndRecord` with a small IO buffer. The OS can raise it under load. Read it back.

## 6. Web MIDI, briefly

Use the platform Web MIDI input. One listener per port.
Status high nibble is the kind: `0xB0` CC, `0x90` / `0x80` notes, `0xE0` pitch bend.
Running status exists on the wire. The browser API already expands it. Do not write a second parser unless you are on a raw USB stream.
Sysex is a separate permission. Do not request it for CC maps.

## 7. Check before you call it done

- `decodeRelative(127, "twos") === -1`, `decodeRelative(65, "offset") === 1`, `decodeRelative(65, "sign") === -1`.
- First pickup tick does not move the target. A later tick within 3/128 does. A tick 50 ms after a catch does, even if it is far.
- The latency line shows samples, period length, band, and heard delay. It does not claim the look-ahead button changed the sound card.
- A learned map beats the factory CC for that device, channel, and number. Other devices keep the factory map.

## Sources

Studied, not vendored. Licenses and commits: `UPSTREAM.md`.

- [Control-Surface](https://github.com/tttapa/Control-Surface) — relative CC curves
- [Mixxx](https://github.com/mixxxdj/mixxx) — soft takeover for DJ faders
- [WebMidi](https://github.com/djipco/webmidi) — browser MIDI events
- [FlexASIO](https://github.com/dechamps/FlexASIO) — WASAPI, WDM-KS, MME, DirectSound
- [miniaudio](https://github.com/mackron/miniaudio) — period size is a hint
- [microsoft/low-latency-audio](https://github.com/microsoft/low-latency-audio) — UAC2 and ASIO on Windows
