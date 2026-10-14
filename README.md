# MIDI CC and driver latency

Agent skill for ObzueAI Group and WarriorLabs Machines.

It covers MIDI learn (absolute, three relative encoder curves, soft takeover, buttons) and how to read a real audio-driver buffer instead of treating scheduler look-ahead as latency.

Studied from Control-Surface, Mixxx, WebMidi, FlexASIO, miniaudio, and Microsoft’s low-latency UAC2 driver. GPL source was not copied. See `UPSTREAM.md`.

Install by placing this folder at `.grok/skills/midi-cc-latency/` so the agent loads `SKILL.md` before touching MIDI or the latency readout.
The matching code in the studio is `src/studio/midi-cc.ts`.
