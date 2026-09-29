# Audio driver latency

Look-ahead and the driver buffer are different delays. Add them. Do not substitute one for the other.

## What to measure

After the audio context is running:

- `baseLatency` — the context’s own block
- `outputLatency` — extra delay the device reports past that block
- sample rate
- buffer in samples, which is `rate * max(outputLatency, baseLatency)` when the context will not tell you a size directly

```
periodMs = samples / rate * 1000
heardMs  = lookAheadMs + periodMs
```

miniaudio’s rule, same idea on every backend: the period you request is a hint.
WASAPI through `IAudioClient3` rounds that hint to a multiple of the fundamental period and clamps it to the driver’s min and max.
If you asked for 128 and the card opened 480, the user is hearing 480. Show 480.

## Windows, from the FlexASIO backend notes

| Backend | Floor that matters | Notes |
| --- | --- | --- |
| MME | Shared engine | Old API. On current Windows it sits on WASAPI shared. Latency numbers run optimistic by about 10 ms |
| DirectSound | About 31 ms on input | Also a wrapper over shared mode. Bad with tiny buffers |
| WASAPI shared | About 10 ms | Mixer, sample-rate conversion, and enhancements are in the path. Reliable numbers. Loopback works here |
| WASAPI exclusive | Single-digit ms | Bypasses the mixer. Other apps lose the device. Bit-perfect if the rate and channel count match |
| WDM-KS | Single-digit ms | Kernel streaming. Exclusive, fussy, no automatic device follow |
| Vendor ASIO | Whatever the interface promises | The real driver. A wrapper is only as good as the backend underneath it |

Microsoft’s UAC2 driver (with Yamaha and Qualcomm) is the USB class driver aimed at this, plus an ASIO face.
The ASIO SDK itself is Steinberg’s and is not in that MIT tree. Do not ship ASIO headers you do not have a license for.

## What the desk should do

- Show look-ahead, interface milliseconds, sample count, period length, band, and heard delay.
- Bands: under 10 ms exclusive, 10–30 ms shared, 30 ms and up legacy.
- Look-ahead buttons (10 / 20 / 40) change the scheduler only.
- Never call a transport reschedule from a volume or fader move. That is a glitch, not latency.
- If the band is shared or legacy, say so in the sentence. Do not offer a fake “lower the buffer” control that the browser cannot honor.
- Tracking target once a real interface is open: 64 or 128 samples. 256–512 when the session is only playing back.

## Other platforms

- macOS Core Audio: already low-latency. Read `outputLatency`. Do not add a Windows driver story.
- Linux: JACK or PipeWire pro profile. Pulse defaults belong in the shared band. Watch period count 2 or 3.
- Android: AAudio exclusive.
- iOS: a small `AVAudioSession` IO buffer, re-read when the OS raises it.
