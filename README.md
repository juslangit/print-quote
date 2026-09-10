# Instant Print Quote

A single web page that prices a 3D printing job from an STL file. Drag a model in
and the browser measures the mesh, estimates filament and print time, and returns
a price built from the shop's own costs — no server, no upload, no slicer.

**Live demo:** https://claude.ai/code/artifact/4473ddab-402c-415e-b363-33e7d97ccf59

---

## Why

Small 3D printing services quote by hand. A customer emails an STL, someone opens
it in a slicer, waits, works out grams and hours, adds a margin and emails back.
That is a quarter of an hour per enquiry, and most enquiries never become orders.

This does the same arithmetic in the browser, instantly, and shows the customer
the working so there is nothing to argue about.

## What it does

- Reads **binary and ASCII STL** with a parser written for this project
- Measures **solid volume, surface area, bounding box and triangle count**
- Models what is **actually extruded** — shell plus infill, not solid volume
- Converts that to **grams** by material density and to **hours** by volumetric flow
- Prices it against **filament cost, machine rate, setup fee, failure allowance
  and margin**, all editable on the page
- Renders the part on a **to-scale build plate** and warns when it will not fit
- Runs entirely client-side. **The STL never leaves the customer's device.**

## The mathematics

**Solid volume** — for each triangle, the signed volume of the tetrahedron formed
with the origin, `dot(a, cross(b, c)) / 6`. Over a closed mesh the exterior
contributions cancel and the enclosed volume remains. Exact for a watertight
model; meaningless for one with holes.

**Surface area** — half the magnitude of the cross product of two edges, per
triangle.

**Extruded volume** — a print is not solid:

```
shell    = min(area × walls × line_width, volume)
interior = volume − shell
infill   = interior × infill_density
extruded = shell + infill (+ supports)
```

The clamp matters: on a thin part the shell *is* the part, and without it the
estimate exceeds the solid volume.

**Mass** — `extruded_cm³ × density`. PLA 1.24, PETG 1.27, ABS 1.04, ASA 1.07,
TPU 1.21 g/cm³.

**Time** — from sustained volumetric flow:

```
flow    = layer_height × line_width × speed × 0.42 × material_factor
seconds = extruded / flow + layers × 0.55 + 240
```

The `0.42` derates the machine's headline speed for acceleration, travel moves
and cooling. `layers × 0.55` is the per-layer change cost, `240` the heat-up,
levelling and purge line.

**Price**

```
material = grams / 1000 × RM per kg
machine  = hours × RM per hour
failure  = (material + machine) × failure %
margin   = (material + machine + failure) × margin %
total    = (material + machine + failure + margin) × qty + setup,  floored at minimum
```

## Details that came from actually printing

- **STL carries no unit.** A file exported in inches and read as millimetres is
  wrong by 25.4×, so the unit is a control rather than an assumption.
- **Binary detection is arithmetic, not a string match.** Plenty of binary STLs
  begin with the word `solid`. A binary file is exactly `84 + triangles × 50`
  bytes; that check is reliable.
- **STL is Z-up, Three.js is Y-up.** The mesh is rotated so the part rests on
  the plate the way it will print.
- **The camera orbits, the part does not tilt.** Rotating the model would swing
  it through the build plate, which reads as wrong to anyone who slices.
- **Warnings a shop would give**: part too large for the selected plate, part
  that only fits rotated 90°, job below the minimum order, TPU at fine layer
  heights, and tall parts that fail more often.

## Limits

These are estimates, not slicer output. Real print time depends on geometry,
orientation, supports and cooling — a tall thin part and a flat part of the same
volume do not take the same time. Support volume is a flat 12% when ticked
rather than a computed overhang analysis. The figure is a quote to confirm,
which is how a shop uses one anyway.

## Running it

Open `index.html` in a browser. No build step, no package manager, no server.
Three.js r128 loads from cdnjs; everything else is in the file.

## Stack

| Part | Choice |
|---|---|
| Rendering | Three.js r128 (UMD, cdnjs) |
| STL parsing | Written for this project |
| Everything else | Vanilla HTML, CSS, JavaScript |

## Licence

MIT
