# Axe-Fx III SysEx & Parameter Maps

An open reference for the **Fractal Audio Axe-Fx III** MIDI SysEx protocol, reverse-engineered
and device-verified on **firmware 32.02**. These are the maps you need to read and set parameters,
identify amp/cab/drive models, and build a signal chain over SysEx — without starting from zero.

This is **reference data only**: JSON maps plus protocol notes. There is no editor, controller,
or automation here. Use it to build your own tools.

> Not affiliated with or endorsed by Fractal Audio Systems. "Axe-Fx" and "Fractal Audio" are
> trademarks of Fractal Audio Systems. Firmware differs between versions — treat **32.02** as the
> baseline and re-verify anything critical on your own unit before relying on it.

## Start here

1. **[SAFETY.md](SAFETY.md)** — read this first. Which commands write flash, which requests can
   corrupt a live block, which reads mutate state, and how to pace MIDI so you don't wedge the unit.
2. **[PROTOCOL.md](PROTOCOL.md)** — the SysEx frame format, checksum, the parameter SET/GET commands,
   and how values are encoded. This is what makes the JSON maps usable.
3. **[TOPOLOGY.md](TOPOLOGY.md)** — the grid: cell addressing and the frames to place, connect,
   and delete blocks.
4. **[ADVANCED_COMMANDS.md](ADVANCED_COMMANDS.md)** — passive display-text reads, whole-block dumps,
   the modifier protocol, and a positional grid read.

## What's in here

### `maps/`
| File | What it is |
|---|---|
| `axefx_sysex_map.json` | **The master map.** All 45 blocks, ~1,600 named parameters, each with its param ID, value scale, type, and page. Includes a `protocol` header and embedded enums (331 amp models, DynaCab speakers/mics). This one file lets you address any parameter. |
| `device_relationships.json` | Real gear → Axe-Fx model. 331 amps + 87 drives, each with the real amp/pedal it's based on and search terms (e.g. "plexi", "bassman", "tube screamer"). |
| `cab_device_relationships.json` | The same, for cabs: DynaCab speakers/mics + legacy IR cabs → real-world gear. |
| `display_label_index.json` | The knob names shown in the editor mapped to the underlying generic parameter (e.g. a Pro Co RAT's printed "Volume" → the generic "Level" param), per model. |
| `param_word_laws.json` | Display-value → wire-word encoding laws for parameters whose value isn't a simple scale (dB levels, log tapers, linear time ranges). Pair with the master map's param IDs. |

### `enums/`
| File | What it is |
|---|---|
| `amp_model_ids.json` | Flat amp-model name → model-ID (331 models). The simplest amp lookup. |
| `dynacab_enum.json`, `dynacab_speakers.json`, `dynacab_mics.json` | DynaCab speaker and mic selector values. |
| `block_types/*_types.json` | Per-block model/type selectors: every drive pedal, delay type, reverb type, pitch mode, etc. → its ID. |

## How the maps were made

The parameter data was captured two ways, cross-checked against each other:

1. **A MIDI sniffer on the Axe-Edit ⇄ hardware link** — watching the real SysEx traffic the
   editor exchanges with the unit.
2. **A parameter walker** — stepping through every block, model, and parameter in Axe-Edit one at a
   time and recording which SysEx address changed and how each value was encoded, then confirming
   the unit's own readback matched.

Everything here was verified on real hardware (FW 32.02). Notes in the JSON flag which entries were
live-verified vs. inferred.

## License

Data and docs are released under **CC BY 4.0** — free to use, adapt, and build on, with attribution
to **Big Saurus**. See [LICENSE](LICENSE).

Corrections and additions from other firmware versions are welcome — open an issue or PR.
