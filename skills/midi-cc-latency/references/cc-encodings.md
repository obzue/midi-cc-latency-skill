# Relative CC encodings

A relative encoder sends a delta in one 7-bit byte. DAWs disagree on the packing.
Control-Surface names the packings the rest of the industry already uses.
These tables are the behavior, not their source.

## Two’s complement (`twos`)

Low 7 bits of a signed step. Positive steps are 1…63. Negative steps wrap: 127 is −1, 126 is −2, down to 64 which is −64.
Some encoders also send 64 for “no move”. Treat 64 as zero or a still wheel twitches.

Reaper calls this relative mode 1. Tracktion uses it too.

## Binary offset (`offset`)

Center is 64. Step = byte − 64. 65 is +1, 63 is −1, 0 is −64, 127 is +63.

Reaper calls this relative mode 2. If a jog only moves one way, it is almost always this curve selected as two’s complement, or the reverse.

## Sign-magnitude (`sign`)

Bit 6 is the sign. The low 6 bits are the distance. 1 is +1. 65 (0x41) is −1. 127 is −63.
0 and 64 are both zero (64 is a negative zero).

Mackie Control wheels use this. Reaper calls it relative mode 3.

## Two addresses (not a curve)

Korg’s inc/dec mode sends the distance on the base CC when the step is positive, and on the next CC when the step is negative.
Learn both numbers. Do not fold them into `sign`.

## How to tell them apart on a desk

Turn the encoder one detent right, then one detent left, and write down the bytes.

| Right | Left | Curve |
| --- | --- | --- |
| 1 | 127 | twos |
| 65 | 63 | offset |
| 1 | 65 | sign |

If the right detent is 65 and the knob goes the wrong way, you picked `sign` instead of `offset`. Change the curve. Do not negate the platter on top.

## Absolute is still absolute

Pots and faders are `value / 127`. Do not run them through `decodeRelative`.
14-bit pairs (CC 0–31 MSB plus CC 32–63 LSB) are absolute at 0…16383. This desk does not require that for the mixer. Pitch bend is the 14-bit path used for jog pressure.
