# Changelog

All notable changes to the Axe-Fx III SysEx & parameter maps. Everything here was
captured and verified on **firmware 32.02**. Param IDs, model IDs, and value scales are
specific to that firmware; re-verify on your own unit before relying on anything critical.

## 2026-08-03

A correctness pass. **66 newly mapped parameters** (**1,921 → 1,987**), still **46 blocks**
and still firmware **32.02**. The headline is not the new rows — it is **27 corrected names
and 15 corrected value scales**, several of which fix entries that were previously wrong in
a way that mattered.

### ⚠ Corrections you should read before writing anything

- **PLEX `pid 90` was named "Detune (Master)". It is not a detune control — it is `Presets`,
  a 45-entry block-preset SELECTOR.** Writing a value to it does not nudge a detune amount;
  it loads a block preset, which rewrites the Plex block's other parameters. If you built
  anything against the previous name, treat that parameter as dangerous and re-check it.
  Renamed, and its hazard raised accordingly.
- **DELAY `pid 81`** was "Phase (Digital Mono)"; it is the block's **LFO 4 Phase**. Its value
  is in **radians on the write path** while the display reads degrees — the two differ by
  57.29578, so sending the number you see on screen is wrong by a factor of π.
- **DELAY `pid 48`** loses its "Target" name and becomes `Unidentified (p48)`. The previous
  name came from a mapping we could not reproduce; an honest unknown beats a confident
  wrong label. (`pid 82` keeps `Target`, now without its misleading type qualifier.)
- **GLOBAL `pids 9–12`** are the **Tuner Offset** settings (previously `Unknown (p9…p12)`),
  each with a scale of 25.
- **MULTITAP `pid 81`** was `Feedback (p81)`; it is `Comb Gain 1`, scale 100.

### Value scales

15 parameters gained or corrected a `scale`. Two shapes are worth calling out because they
bite silently:

- Several `%`-range parameters carry a **display range that is not the write divisor**. A
  control shown as ±200 % can still want the value divided by **100** on the wire. Where we
  have proven the divisor on the device, the scale in this map is the one to write with.
- Phase-family parameters are **radians on the wire, degrees on the display** (57.29578).

### Everything else

- **+66 parameters** across the map, and 50 entries had their `status` updated (mostly rows
  moving from provisional to verified, a few the other way when a witness did not hold up).
- Notes throughout have been tightened. Nothing was removed from a note except internal
  bookkeeping.

**Not updated in this release:** `display_label_index.json` is unchanged pending its own
review pass — the master map, word laws and the two device-relationship files are current.

## 2026-07-24

A device-settings and coverage pass that adds **133 newly mapped parameters** to the master
map (**1,788 → 1,921 named parameters**) and introduces the map's **first system/FC-layout
block**, taking coverage from **45 to 46 blocks**. Still firmware **32.02**, and the map stays
firmware-tracked — re-verify on your own unit before relying on anything critical.

### New block — FC Layout (46th block)

- **FCLAYOUT** — a system/config block (Axe-Edit's "FC Edit" surface) that describes the FC
  foot-controller switch layout; it is **not** a grid effect (not placeable or bypassable). This
  release seeds it with the switch-1 **Tap Category** and **Hold Category** selectors; the full
  per-switch grid is not yet enumerated, so the block is marked `complete: false`.

### System / global setup parameters (GLOBAL, +66)

The GLOBAL block gained the device's **Setup**-menu settings that live outside the edit buffer
(persistent, autosaved — read the current value before you write, there is no reload-to-revert):

- **Tuner** — Reference Pitch and related tuner-page settings.
- **MIDI / general** — MIDI clock send/receive, "Ignore Redundant PC", and related options.
- **External controllers** — the real **External Control 1–12 source** assignments (pids
  1302–1313). This **supersedes and falsifies** the earlier mapping that placed external-control
  sources at pids 80–87: those eight entries are **kept but renamed and marked falsified**, with
  their true identity now listed as unknown. Update any code that addressed pids 80–87.
- **Bypass-CC assignments** — the per-block **MIDI Bypass CC** table (39 entries), i.e. which
  MIDI CC toggles each block's bypass.

### Per-block "Bypass State" reflection parameter (27 blocks)

Most effect blocks gained a read-only **`Bypass State`** entry that mirrors the block's
engaged/bypassed state (function `0x0A`). It is a **reflection of block enable, not proven
independently settable**, and is distinct from a block's own `Bypass` / `Bypass Mode` selector.
Treat it as read-only until proven otherwise.

### More block parameters, including closed type-gated entries

Additional parameters across roughly fifteen blocks, several of them previously type-gated
entries now resolved: **MULTITAP +13**, **CAB +7** (per-type `Level`/`Pan`/`Low Cut`/`High Cut`/
`Smoothing` instances), **TENTAP +6**, **COMP / GATE / PLEX / RESONATOR / IRPLAYER +3** each,
plus one or two each in AMP1, ENHANCER, MEGATAP, PITCH and a dozen other blocks.

### Honest caveats

- **Value encodings are often derived, not write-verified.** Many new entries had their
  display↔wire scale **inferred** (e.g. from sibling parameters) rather than confirmed by a
  round-trip write; those say so in their `note`. **39 new entries still carry an unresolved
  scale** (`scale: null`) — addressable, but derive the scaling yourself before writing
  continuous values.
- **PITCH `Feedback 1–4` (pids 31–34) are under re-verification.** These labels may be a
  routing-matrix mislabel rather than true per-voice feedback; treat them with caution pending
  the next confirmation pass.
- Coverage remains **FW 32.02-specific** — param IDs and scales can move between firmware.

### How this release was verified

New and corrected entries were identified on real hardware (FW 32.02) and cross-checked between
independent methods. Where an entry was set and read back over MIDI it is marked write-verified
in its `note`; entries that are named/identified but not yet write-confirmed are flagged as such.
Value encodings that have been worked out live in `param_word_laws.json`.

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
  not a switch. The full 78-entry enum is transcribed in the map (`values`, index order
  as the firmware presents it: `1/64 TRIP` = 0 … `1/32 DOT` = 5 …).

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
