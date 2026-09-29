# Upstream clones

Sparse clones, depth 1, taken on 2026-09-29. Full trees were not copied into this skill.
Control-Surface is GPL-3.0. Mixxx is GPL-2.0-or-later. Their source stays in those repositories.
This skill is an original description of the behavior, written for ObzueAI / Warrior Lab.

| Repository | Role | Commit | License |
| --- | --- | --- | --- |
| https://github.com/tttapa/Control-Surface | Relative CC curves (two’s complement, binary offset, sign-magnitude, Mackie, Reaper, Korg address pair) | `b2d2f11f7505a8560a0d3bbd647d2605e7969850` | GPL-3.0 |
| https://github.com/mixxxdj/mixxx | Soft takeover: ignore the first tick, 3/128 catch window, 50 ms whip | `5fefde07fa8dc8ff5ac255427fe80d2bc2b0065d` | GPL-2.0-or-later |
| https://github.com/djipco/webmidi | Web MIDI input model used by the desk | `11418274d27847c06d4035a06010ac7edcb5c00c` | Apache-2.0 |
| https://github.com/dechamps/FlexASIO | Which Windows backend can go under 10 ms | `e75cc72483a4f0a4cb906bda80463b0c4bd6e8d3` | See the repository |
| https://github.com/mackron/miniaudio | Requested period is a hint; WASAPI may clamp it | `9634bedb5b5a2ca38c1ee7108a9358a4e233f14d` | Public domain or MIT-0, author’s choice |
| https://github.com/microsoft/low-latency-audio | Official UAC2 driver plus an ASIO face. ASIO itself stays under Steinberg’s license | `8ef2a02d670bd0120be46f13792b946f26334a6f` | MIT for Microsoft’s code. ASIO SDK is not included and is not MIT |

Files that were checked out, and only those:

- `Control-Surface/src/MIDI_Senders/RelativeCCSender.hpp`
- `mixxx/src/controllers/softtakeover.cpp`
- `mixxx/src/controllers/softtakeover.h`
- `FlexASIO/BACKENDS.md`, `FlexASIO/README.md`
- `miniaudio/README.md`
- `webmidi/README.md`
- `low-latency-audio/README.md`

Do not drop those GPL files into the app, the skill, or a commercial installer.
The implementation that ships is `src/studio/midi-cc.ts`.
