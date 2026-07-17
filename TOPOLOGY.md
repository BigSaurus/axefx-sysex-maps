# TOPOLOGY — building the grid over SysEx

The Axe-Fx III signal grid is fully controllable over SysEx (not a vendor-pipe-only feature).
Everything below was verified by live MIDI replay on **firmware 32.02** — the unit reflects the
changes, and its own status dump (`0x13`) confirms them.

> **Run topology edits with Axe-Edit CLOSED.** A connected Axe-Edit re-asserts its cached grid and
> will clobber a block you place (cable ops survive because it reads connections live). Placement
> is device-direct: close the editor and send the frames yourself.

All frames use the standard envelope and checksum from [PROTOCOL.md](PROTOCOL.md).

## Cell addressing

The grid is **6 rows × 14 columns, column-major**:

```
cell index = column * 6 + row
```

The main chain in the factory empty preset (0000) lives on **row 2**: Input at column 0 (idx 2),
through Amp at column 5 (idx 32), Cab at column 6 (idx 38), out to Output at column 13 (idx 80).

## Place a block (or shunt)

Two frames — select the cell, then place the block instance there:

```
01 30 00 00 00 00 00 <cellidx>                     (select/prepare the cell)
01 32 00 <blkLo blkHi> 00 00 <cellidx>             (place 14-bit block id at the cell)
```

- **Block instance IDs** come from the master map (e.g. Pitch1 = `0x6E`, Comp1 = `0x2E`).
- A **shunt** is block id `03 08` (1027). Shunts need distinct instance IDs; walk the pool
  (1027, 1029, 1032…) skipping any that no-op.
- You can only place an instance that is **not already on the grid**. The place ACK's flag byte
  tells you which happened: **`00` = newly placed** (cell was free), **`02` = no-op** (that instance
  is already on the grid). After placing, set the block's model and params with the `01 09` SET.

## Delete a block

Deleting is "place the empty block (id 0)" into the cell:

```
01 30 00 00 00 00 00 <cellidx>                     (select the cell — does NOT delete on its own)
01 32 00 00 00 00 00 <cellidx>                     (place block id 0 = empty -> clears the cell)
```

Delete does **not** auto-reconnect neighbours (it leaves a gap). Clearing many cells needs
≥ ~40 ms/cell pacing or the device drops deletes.

## Connect / disconnect a cable

One frame sets the cable state (idempotent — re-sending is a no-op, not a toggle):

```
01 35 00 00 00 00 00 <FLAG> 00 00 00 00 00 00 02 00 <c0> <c1> <c2>
        FLAG (byte 12):  01 = CONNECT      02 = DISCONNECT
```

Only byte 12 differs between connect and disconnect; the 3-byte cable code is the same for a given
source→destination pair. The cable code is computed from the two cell indices:

```
src_idx = src_col*6 + src_row
dst_idx = dst_col*6 + dst_row
c0 =  src_idx >> 1
c1 = (dst_idx >> 2) | ((src_idx & 1) << 6)
c2 = (dst_idx & 3) << 5
```

Cables bind to grid **cell positions** and register even on empty cells. The input-cable link is
explicit even for a plain same-row series connection — there's no implicit series feed.

## Putting it together

Place blocks, set their models/params via `01 09`, and connect them with computed cable codes —
that's a full preset built over SysEx, no UI clicks. The unit's `0x13` status dump verifies the
result.
