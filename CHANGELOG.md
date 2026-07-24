# Changelog

All notable changes to the Axe-Fx III SysEx & parameter maps. Everything here was
captured and verified on **firmware 32.02**. Param IDs, model IDs, and value scales are
specific to that firmware; re-verify on your own unit before relying on anything critical.

## 2026-07-23

A parameter-coverage pass that adds **145 newly mapped parameters** to the master map
(**1,643 → 1,788 named parameters**, 45 blocks unchanged) and corrects one important
addressing error. If you built anything against the initial release, read the breaking
fix first.

### Breaking fix — PITCH block `Run` was published at the wrong param ID

In the initial release the PITCH block's Arpeggiator **`Run`** switch was mapped to
**param ID 76**. That is wrong. Device-verified on firmware 32.02 (toggling `Run` in the
editor while reading the block back over MIDI):

- **`Run` is param ID 75** — a plain Off/On switch.
- **Param ID 76 is `Arp Tempo`** — a 78-entry tempo-division selector (e.g. `1/32 DOT`),
  not a switch. (The full enum is not transcribed in the map; read it from firmware if
  you need the exact ordering.)

**Impact:** anyone using the old mapping who toggled `Run` was actually writing the
Arpeggiator tempo-division selector at pid 76, not the Run switch. Update any code that
addresses PITCH pid 75/76.

### Corrected parameter names

- **PITCH** — `Run` moved to pid 75; pid 76 renamed to `Arp Tempo` (see above).
- **MEGATAP** — pid 11 renamed to **`Time Randomize`** (Time-section per-tap randomization;
  previously an unidentified/hidden entry). A prior time-domain (ms) interpretation of this
  pid is superseded — the control displays in percent; its encoding is not yet derived.
- **REVERB** — pid 51 renamed `Mix` → **`Pitch Mix`**, pid 54 renamed `Feedback` →
  **`Pitch Feedback`** (the pitch/shimmer engine's wet mix and feedback; percent, scale ×100).
  These are functional **only on the pitch-capable "Sfx" reverb types**; on other types a
  write is accepted and read back but dropped on preset reload. They are distinct from the
  main reverb `Mix` (pid 13) and echo `Feedback` (pid 62).
- **VOCODER** — the second 24-band level bank (pids 43–66) renamed from bare digits
  `1`…`24` to **`Level 1`…`Level 24`**, so the two banks are addressable by distinct names.

### New parameters

145 parameters added across 30 blocks. Highlights:

| Block | Added | Notable new parameters |
|---|---|---|
| PITCH | +27 | `Run` (75, see fix), `Arp Tempo` (76), `Mode`, `Master Pitch`, `Key`, `Scale`, `Pitch Quantize`, `Pitch Tracking` (35), `Shift 1–4`, `Steps`, `Repeats`, `Diffusion`, … |
| MULTITAP | +25 | `Type` (68), `Phase` (70), `Attack/Release Time`, `Comb Time/Depth`, `Filter Type`, `Ring Mod Frequency/Mix`, `Chorus Rate/Depth`, `Feedback 1→2 … 4→1`, `Kill Dry`, … |
| TENTAP | +17 | `Time 1–6`, `Level 1–10`, `Bypass` |
| MULTIBAND | +16 | full Mid/High band controls — `Threshold`, `Ratio`, `Attack`, `Release`, `Level`, `Detector`, `Mute` per band, plus `Mid/High Crossover` and master `Level`. Low-band controls are now tagged `band: low`. |
| COMP | +9 | `Type` (12, 19-entry compressor model selector, enum published), `Auto Makeup` (6), `Auto Att/Rel` (16), `Compression`, `Dynamics`, `Time`, `Transients`, `Tone`, `Drive` |
| LOOPER | +6 | `Playback Level`, `Overdub Level`, `Low Cut`, `Dry Level`, `Record 2nd Press`, `Max. Loop Time` |
| PEQ | +5 | `Solo 1–5` (25–29) — per-band solo switches |
| ENHANCER | +5 | `Phase Invert`, `Pan Left`, `Pan Right`, `Balance`, `Bypass` |
| PLEX | +4 | `Granule Length` (59), `Reverb Size` (79), `Shimmer Intensity`, `Kill Dry` |
| MEGATAP | +4 | `Shape`, `Alpha`, `Amplitude Randomize` (30), `Kill Dry` |
| VOLPAN | +4 | `Threshold`, `Attack`, `Release`, `Hysteresis` |
| TONEMATCH | +3 | `Start Reference` (2), `Start Local` (3), `Bank` |
| CAB | +1 | `Length` (72, IR length selector) |
| RINGMOD | +1 | `Pitch Tracking` (2) |

Also +1/+2 each in CHORUS, DELAY, DRIVE, FBRETURN, FILTER, FLANGER, FORMANT, GATE,
GRAPHEQ, MUX, PHASER, REVERB, ROTARY, SYNTH, TREMOLO, WAH — mostly the per-block
`Bypass` switch and a few type/mode selectors.

**About value encodings:** these new entries carry param ID, display label, type, page,
and (for selectors) the enum. The **value encoding (scale) is not yet derived** for most
of them unless the entry says otherwise — treat them as addressable but re-derive the
display↔wire scaling yourself before writing continuous values. Where an encoding *has*
been worked out it lives in `param_word_laws.json`, which gained laws this release for the
MULTIBAND (multiband compressor) band family, the compressor attack/release times, several
per-tap delay time/feedback controls, and a few amp/phaser/megatap controls.

### Display-label index

- Added the MULTITAP editor label **`LFO Phase`** (shown under the Diffusor type) as an
  alias of the block's generic **`Phase`** parameter (pid 70).
- The amp graphic-EQ band aliases (`62`…`8K` → `EQ 62`…`EQ 8K`, pids 55–62) were already
  present in the initial release; they were independently re-verified this cycle (no change).

### Documented non-parameters (useful negative knowledge)

These editor controls are **not** block parameters — don't try to address them via the
parameter SET/GET commands:

- **LOOPER transport** (`Record`, `Play`, `Once`, `Reverse`, `Erase`): these are the
  dedicated looper realtime function (SysEx opcode `0x0F`), not stored block parameters.
- **CAB per-slot `M`/`S`** (mute/solo) and **`Auto Align`**: runtime mix/action controls,
  not stored parameters.

### How this release was verified

Each new or corrected entry was confirmed on firmware 32.02 by **setting a distinctive
value (or toggling a switch) in the editor and reading the block back over MIDI** to see
which param ID moved. Entries that were only named from the editor UI (and not yet
confirmed by a write) are flagged as such in their `note` field.
