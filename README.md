# Bulk Rail Loaders: Pooled Plus

A fork of
[Modern Bulk Rail Loaders: Pooled](https://github.com/solvemon/bulk-rail-loaders-pooled)
by Sølve Monteiro, based on the Factorio 2.1 port and save migration by
[kfsone](https://github.com/kfsone/bulk-rail-loaders-pooled). It adds exact
loading for Cybersyn 2 deliveries and reworked graphics. It replaces the
original mod and cannot be enabled together with it.

**Existing saves:** disable or remove the original mod (or kfsone's
*Unofficial 2.1 Fix*) before enabling this one. Silos, pools and their contents
are kept, and the mod's script state is rebuilt from the placed entities on
first load. Each station's **Load/Unload mode** cannot be recovered from the
map: every station comes back as *Load*, so reopen unloading stops and switch
them back to *Unload*, or use the `set_station_mode` remote call below. The
wagon count remembered by the stop panel resets; the gap style is inferred
from the silo layout.

A pooled-inventory rework of the bulk rail loader concept. Every station has
**one** hidden cargo inventory (the *pool*); each visible silo is a Factorio 2.0
`proxy-container` — a physical window onto that shared pool. Because there is
only one inventory per station:

* Contents can never get unbalanced between wagons. A train's order completes
  whenever the **station total** suffices — no stuck partial loads.
* There is no balancing logic and no recurring UPS cost for it.
* Factory inserters/loaders can feed or drain **any** silo; it all lands in the
  same pool.

Requires **Factorio ≥ 2.1.0**. Factorio 2.0 users should use the original mod.

## Usage

1. Open a train stop and use the **Bulk loaders** panel: pick the wagon count,
   locomotive gap, and the station mode (**Load** / **Unload**).
2. Silos are placed automatically over each wagon slot; blocked positions get
   ghosts you can revive after clearing obstructions.
3. Loading stations show a hopper structure over each wagon; unloading
   stations show a tipper grate beneath the track.

### Circuit network / Cybersyn

Wire **exactly one** silo (any of them) to your circuit network or Cybersyn
combinator — every silo reports the full shared pool, so wiring two silos
doubles the reading.

### Cybersyn 2: exact loading

With [Cybersyn 2](https://mods.factorio.com/mod/cybersyn2) installed, a train
that arrives at a loading station on a Cybersyn 2 delivery is loaded with
**exactly** that delivery's items and amounts; everything else stays in the
pool, so one station can provide several items. Trains without a Cybersyn 2
delivery are loaded with the whole pool as before. The map setting
*Cybersyn 2: load exact delivery amounts* turns this off.

**Unload stations need an inactivity timeout.** Silos unload a whole train in
one tick, so the train leaves before the circuit network shows the new stock,
and Cybersyn 2 then orders the same delivery again. On the station combinator
of every unload station, set *Inactivity mode* to **After delivery** with a
timeout of at least 1 second.

### Remote interface

`remote.call("mbrlp", "set_station_mode", surface_name, x, y, mode)` sets the
station whose train stop is within 0.5 tiles of `(x, y)` to `"load"` or `"unload"`
(same as the stop panel's switch, silos are redrawn). Returns `true` on success,
`false` for bad arguments or no station found. Example:
`/c remote.call("mbrlp", "set_station_mode", "nauvis", -427, 55, "unload")`.
`remote.call("mbrlp", "get_station_mode", surface_name, x, y)` returns the
station's mode (`"load"`/`"unload"`) or `nil`. Handy for re-setting modes after
the state rebuild described above.

## Design notes

* The pool is a hidden, indestructible container placed at the train stop. Its
  usable size is `silo capacity × silo count`, enforced with an inventory bar.
* Mining a silo loses nothing (items live in the pool). Removing the *last*
  silo of a station spills the pool contents on the ground rather than voiding
  them.
* Stations are uniform: the whole station either loads or unloads. Mixed
  stations would mix incoming and outgoing cargo in one pool.
* Hand-placed silos attach to the nearest train stop within 100 tiles
  (preferring stops that already have silos), or get a private pool if none.

## Licensing / attribution

* Code: by Sølve Monteiro (original mod), kfsone (Factorio 2.1 port, save migration, remote interface) and
  tarioch (this fork).
* All graphics (`graphics/loader-*.png`, `graphics/unloader-*.png`) are from
  [railloader by mspielberg (therax)](https://github.com/mspielberg/factorio-railloader),
  licensed **LGPL-3.0** (see `LICENSE-railloader-graphics`). Keep that license
  file and this attribution if you redistribute the mod. The unloader images
  are modified: shortened from 6 to 4 tiles to match the silo, with a
  wall drawn around the pit. The loader images are modified too: upscaled
  to double resolution with Real-ESRGAN, given a brushed-metal finish on the
  frame, and their baked shadow is lighter and soft-edged. The vertical
  loader also gets the horizontal one's ribbed front panel over the top of
  its track opening.

## Development

Symlink into your mods folder under the mod's name (the folder name must match
`info.json`):

```
ln -s /path/to/this/repo ~/.factorio/mods/bulk-rail-loaders-pooled-plus
```
