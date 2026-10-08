# PCB layout

How to turn a finished schematic into a board in KiCad 10, set up for JLCPCB. When the board is done, [Manufacture.md](Manufacture.md) covers exporting the files and ordering. For making parts, see [Symbol.md](Symbol.md) and [Footprint.md](Footprint.md).

The rule values in this guide come from JLCPCB's [capabilities page](https://jlcpcb.com/capabilities/pcb-capabilities). They change from time to time, so check it before relying on a limit.

- [The layout flow](#the-layout-flow)
- [Before you start](#before-you-start)
- [Set up the board for JLCPCB](#set-up-the-board-for-jlcpcb)
- [The grid](#the-grid)
- [Board outline](#board-outline)
- [Mounting holes](#mounting-holes)
- [Placing parts](#placing-parts)
- [Routing](#routing)
- [Copper zones](#copper-zones)
- [Two-layer and four-layer boards](#two-layer-and-four-layer-boards)
- [Silkscreen](#silkscreen)
- [Design rules check](#design-rules-check)
- [3D check](#3d-check)
- [Tool cheat sheet](#tool-cheat-sheet)
- [Layers you draw on](#layers-you-draw-on)
- [Common mistakes](#common-mistakes)
- [Checklist before you export](#checklist-before-you-export)

## The layout flow

| Step | Where | Done when |
|---|---|---|
| 1. Finish the schematic | Schematic Editor | ERC is clean and every symbol has a footprint |
| 2. Set up the board | **File > Board Setup** | Layers, rules, and net classes match JLCPCB |
| 3. Bring the parts over | **Tools > Update PCB from Schematic** (F8) | Every footprint is on the canvas, joined by thin ratsnest lines |
| 4. Draw the board outline | `Edge.Cuts` layer | One closed outline |
| 5. Add mounting holes | Footprints from the `MountingHole` library | Holes sit where the enclosure or standoffs need them |
| 6. Place the parts | Move, rotate, flip | Every part inside the outline, no courtyards overlapping |
| 7. Route | Interactive router (X) | No ratsnest lines left |
| 8. Add copper zones | Filled zone tool | Ground planes drawn and filled |
| 9. Tidy the silkscreen | `F.Silkscreen`, `B.Silkscreen` | Every label readable and off the pads |
| 10. Run the design rules check | **Inspect > Design Rules Checker** | 0 errors |
| 11. Check in 3D | **View > 3D Viewer** (Alt+3) | Parts and connectors look right |
| 12. Export and order | [Manufacture.md](Manufacture.md) | Order placed |

Steps 6 to 9 go back and forth. Expect to move parts while routing.

## Before you start

- **Run ERC in the Schematic Editor** and fix or deliberately exclude every error.
- **Annotate the schematic** with **Tools > Annotate Schematic**, so every part has a real reference such as `R1` instead of `R?`.
- **Give every symbol a footprint.** **Tools > Assign Footprints** lists any that are missing.
- **Add mounting holes as symbols** in the schematic, so they survive later updates. See [Mounting holes](#mounting-holes).
- **Fill in `LCSC Part #`** on every part JLCPCB will assemble.

## Set up the board for JLCPCB

Open the board, then **File > Board Setup**. Its pages are listed down the left side. Set them once, then reuse them (see [Reuse the setup](#reuse-the-setup)).

### Layers and stackup

On the **Physical Stackup** page:

| Setting | Value |
|---|---|
| Copper layers | `2` for most boards, `4` when needed (see [Two-layer and four-layer boards](#two-layer-and-four-layer-boards)) |
| Board thickness | `1.6` mm, JLCPCB's usual default |

The individual layer thicknesses only matter for controlled-impedance work. Leave them at their defaults otherwise.

### Constraints

The **Constraints** page holds the hard limits. The design rules check reports anything below them as an error. Set them a little above JLCPCB's absolute limits, so a board that passes DRC is comfortably manufacturable.

| Constraint | JLCPCB limit (2-layer, 1 oz) | Set it to |
|---|---|---|
| Minimum clearance | 0.10 mm | `0.15` mm |
| Minimum track width | 0.10 mm | `0.15` mm |
| Minimum annular width | 0.18 mm on plated holes | `0.15` mm |
| Minimum via diameter | 0.25 mm | `0.6` mm |
| Minimum through hole | 0.15 mm | `0.3` mm |
| Copper to hole clearance | 0.2 mm | `0.25` mm |
| Copper to edge clearance | 0.2 mm routed, 0.4 mm V-cut | `0.3` mm, or `0.5` mm if the board is V-cut from a panel |
| Hole to hole clearance | 0.2 mm via, 0.45 mm pad | `0.25` mm |
| Silkscreen minimum item clearance | 0.15 mm pad to silkscreen | `0.15` mm |
| Silkscreen minimum text height | 1.0 mm | `1.0` mm |
| Silkscreen minimum text thickness | 0.15 mm | `0.15` mm |

These settings allow vias of 0.3 mm hole and 0.6 mm diameter or larger. JLCPCB can make much smaller ones, down to 0.15 mm hole and 0.25 mm diameter. Lower the via constraints only when a dense board needs it.

### Net classes

A net class is a group of nets that share a track width, clearance, and via size. The **Net Classes** page holds them. Net classes are shared between the schematic and the board.

| Class | Clearance | Track width | Via diameter | Via hole | Use for |
|---|---|---|---|---|---|
| `Default` | `0.2` | `0.25` | `0.6` | `0.3` | Signals |
| `Power` | `0.25` | `0.5` or wider | `0.8` | `0.4` | Supply rails and ground. Size the width from [Trace widths](#trace-widths) |

Assign nets to a class in the lower table on the same page, using a pattern. For example, the pattern `GND` puts the ground net in `Power`, and `+*` puts every net whose name starts with `+` in `Power`.

When you route a net, KiCad starts with its class's track width.

### Pre-defined sizes

On the **Pre-defined Sizes** page, add the widths and vias you'll use often. They then appear in the drop-down lists on the top toolbar, so you can switch width while routing.

| Tracks | Vias (diameter / hole) |
|---|---|
| `0.25`, `0.5`, `1.0`, `2.0` mm | `0.6 / 0.3`, `0.8 / 0.4` mm |

### Text and graphics defaults

On the **Defaults** page under Text & Graphics, set the silkscreen row:

| Setting | Value |
|---|---|
| Line thickness | `0.15` mm |
| Text height and width | `1.0` mm |
| Text thickness | `0.2` mm |

JLCPCB asks for text lines at least one-sixth of the text height, so 1.0 mm text needs about 0.17 mm lines. 0.2 mm leaves a margin.

### Reuse the setup

You don't have to repeat this for every board. Either:

- **Keep a template project** with these settings, and start new boards from it with **File > New Project from Template**, or
- **Copy from an existing board:** in Board Setup, click **Import Settings from Another Board** at the bottom, pick a board that's already set up, and choose which pages to copy.

## The grid

Everything you place snaps to the grid, which keeps parts aligned and coordinates round.

### Recommended grid sizes

| Task | Grid |
|---|---|
| Board outline and mounting holes | `1.0` mm or `0.5` mm, so dimensions are round numbers |
| Placing through-hole parts and connectors | `1.27` mm (50 mil) or `2.54` mm (100 mil), to match 0.1" pin spacing |
| Placing surface-mount parts | `0.5` mm or `0.25` mm |
| Routing tracks | `0.25` mm, or `0.125` mm in tight spots |
| Silkscreen text | `0.25` mm |

Set the grid in the drop-down on the top toolbar, or step through sizes with **N** and **Shift+N**.

### Fast grids

Two grids can be bound to hotkeys, for flipping between coarse and fine without the drop-down. Choose them in the grid dialog (**View > Grid Properties**, or right-click the canvas and pick the grid entry), then switch with **Alt+1** and **Alt+2**.

A good pair is `1.27` mm for placement and `0.25` mm for routing.

### Grid overrides

Grid overrides give each kind of object its own grid, used instead of the main one. For example, footprints can stay on 1.27 mm while tracks and text use 0.25 mm. Set them in the grid dialog, and toggle them all on or off with **Ctrl+Shift+G**.

### Grid origin

The grid origin is the point the grid lines up to.

1. Click **Place > Grid Origin** and click the board's top-left corner.
2. In **Preferences > PCB Editor > Origins & Axes**, set the displayed origin to the grid origin.

Coordinates in every dialog are then measured from the board's corner, so you can type positions straight from a mechanical drawing.

### Snapping

- **Shift+Space** toggles between 45° and free-angle drawing.
- **Shift+S** toggles whether objects snap to items on every layer or only the active one.
- Snapping to pads and track ends is set under **Preferences > PCB Editor > Editing Options** (magnetic items).

## Board outline

The outline tells the factory where to cut the board. It goes on the `Edge.Cuts` layer and must be one closed shape.

1. Select `Edge.Cuts` in the **Layers** panel.
2. Click **Place > Rectangle**, draw roughly, then double-click it and type the exact corners.
3. Check the outline in the 3D viewer (Alt+3). If it's open or broken, the viewer shows no board.

For rounded corners, draw the outline as four lines, select them, then right-click and use **Shape Modification > Fillet Lines** with a 1 to 3 mm radius.

Cutouts and slots inside the board also go on `Edge.Cuts`, as closed shapes.

Size guidance:

- **Price:** at JLCPCB, a small 2-layer board, roughly within 100 × 100 mm, is usually in the cheapest price tier.
- **Edge margin:** keep copper at least 0.3 mm from the outline. The **Copper to edge clearance** constraint enforces this.

## Mounting holes

Add them as symbols in the schematic, so **Update PCB from Schematic** doesn't remove them.

1. In the Schematic Editor, place the symbol `MountingHole` from the `Mechanical` library, once per hole. Use `MountingHole_Pad` instead if the hole should connect to a net, usually `GND`, and wire its pin.
2. Assign each one a footprint from KiCad's `MountingHole` library:

   | Screw | Hole | Footprint |
   |---|---|---|
   | M2 | 2.2 mm | `MountingHole:MountingHole_2.2mm_M2` |
   | M2.5 | 2.7 mm | `MountingHole:MountingHole_2.7mm_M2.5` |
   | M3 | 3.2 mm | `MountingHole:MountingHole_3.2mm_M3` |
   | M4 | 4.3 mm | `MountingHole:MountingHole_4.3mm_M4` |

3. Update the PCB (F8) and place each hole by typing its exact position in its properties (E).

Footprint name endings:

| Ending | What it is |
|---|---|
| None | A bare, unplated hole with no copper |
| `_Pad` | A plated hole with a copper ring, which can connect to a net |
| `_Pad_Via` | A plated ring with small vias around it, for a strong ground connection |
| `_ISO7380`, `_DIN965`, and similar | Keep-out sized for a particular screw head |

Practices:

- **Match the enclosure.** Take hole positions from the enclosure or standoff drawing, not by eye.
- **Edge distance:** keep an M3 hole's centre at least 3.5 to 4 mm from the board edge.
- **Clear space:** keep tracks, parts, and silkscreen out of each hole's courtyard, which shows where the screw head and washer sit.
- **Placement order:** place mounting holes and connectors first, since the mechanics fix their positions.

## Placing parts

Good placement makes routing easy. Most layout problems are placement problems.

Recommended order:

1. Mounting holes and anything else fixed by the enclosure.
2. Connectors, at the board edge and facing outward.
3. The main chips and modules.
4. Parts that must sit close to a particular pin: decoupling capacitors, crystals, and the components around a regulator.
5. Everything else, grouped by function as in the schematic.

Practices:

- **Decoupling capacitors** go as close as possible to the power pin they serve, ideally within 2 to 3 mm, with a short path to ground.
- **Switching regulators** follow the layout example in their datasheet. Keep the input capacitor, switch node, and inductor tight, and keep the switch node's copper area small.
- **Keep noisy and sensitive circuits apart.** Separate switching regulators and motor drivers from analog inputs and sensors.
- **Short, direct power paths.** Put the power input, protection, and regulator in a line, near where power enters.
- **Same orientation for similar parts,** where it doesn't lengthen routing. It makes assembly and inspection easier.
- **Surface-mount parts on one side** if JLCPCB will assemble the board. Assembling both sides costs more.
- **Leave room for a soldering iron** around parts you will solder by hand.
- **Tall parts** go where they won't block connectors, cables, or the enclosure.
- **Keep courtyards from overlapping.** DRC reports overlaps.

Useful tools:

- **Pack and Move Footprints (P)** gathers selected parts into a tidy cluster you can drop in place.
- **Get and Move Footprint (T)** grabs a part by its reference, such as `C4`.
- **Rotate (R)**, **Flip to the other side (F)**, and **Move (M)**.
- **Right-click > Align/Distribute** lines up a row of parts.
- **The ratsnest** shows connections still to be routed. Rotate and swap parts to reduce crossings before routing.

## Routing

### Order of work

1. Power and ground paths, which need width.
2. Critical signals: crystals, USB, sensor inputs, anything fast or sensitive.
3. Everything else.

### Trace widths

Track width sets how much current a trace carries before it heats up. Approximate values for an outer layer with 1 oz copper and a 10 °C temperature rise, from IPC-2221:

| Track width | Current |
|---|---|
| 0.15 mm | 0.6 A |
| 0.25 mm | 0.9 A |
| 0.5 mm | 1.5 A |
| 1.0 mm | 2.4 A |
| 2.0 mm | 4.0 A |
| 3.0 mm | 5.3 A |

Practical defaults:

| Net | Width |
|---|---|
| Signals | `0.25` mm |
| Supplies under 0.5 A | `0.4` to `0.5` mm |
| 1 to 2 A | `1.0` mm |
| 3 to 5 A | `2.0` to `3.0` mm, or a copper zone |
| Over 5 A | A copper zone, and consider 2 oz copper |

Inner layers carry roughly half as much, and JLCPCB's inner layers default to 0.5 oz copper, which carries less again. For exact numbers, use the **Track Width** tab in **Calculator Tools**, opened from the main KiCad window.

### Clearance and voltage

The default 0.2 mm clearance is fine for low-voltage boards, up to about 50 V. Mains and other high voltages need much larger spacing. Look up the requirement in IPC-2221 or the relevant safety standard before laying out such a board.

### Vias

- **Default size:** 0.3 mm hole, 0.6 mm diameter.
- **Power connections:** use 0.4 mm / 0.8 mm vias, and several in parallel. Count on roughly 1 A per standard via as a cautious rule.
- **Ground stitching:** scatter vias joining the ground zones on each layer. See [Copper zones](#copper-zones).
- **Vias in pads:** avoid them on parts you will hand-solder, because solder runs down the hole.

### Routing practices

- **Use 45° bends,** not 90° corners. Avoid acute angles, which can trap etchant.
- **Keep tracks short and direct,** especially for power and fast signals.
- **On two-layer boards,** route most signals on the top, and make short hops on the bottom that cross at right angles. Keep the bottom mostly solid ground.
- **Don't cut the ground plane** with long bottom-side tracks. A signal's return current flows in the ground directly beneath it, and a slot forces a long detour.
- **Differential pairs** such as USB D+ and D− are routed together with **Route > Route Differential Pair** (6), kept equal in length, and kept short. Use JLCPCB's impedance calculator if their impedance matters.
- **Leave no dangling ends.** DRC reports a track stub that goes nowhere.
- **Router mode:** **Route > Interactive Router Settings** picks how the router deals with obstacles: highlight collisions, shove other tracks aside, or walk around them.

### While routing

| Key | Action |
|---|---|
| X | Start a track from a pad |
| / | Switch which way the bend goes |
| V | Drop a via and continue on the other layer |
| W / Shift+W | Next / previous track width |
| Backspace | Remove the last segment |
| End, or double-click | Finish the track |
| Esc | Cancel |
| D | Drag an existing track, with the router keeping it clear of others |

## Copper zones

A zone is an area of copper that fills itself around everything else on its layer. The usual use is a ground plane.

### Add a ground zone

1. Press **Ctrl+Shift+Z** (draw filled zone) and click the first corner.
2. In the dialog, tick the layers (on a two-layer board, both `F.Cu` and `B.Cu`) and choose the net `GND`.
3. Click around the board, slightly outside the outline, and double-click to close. The fill is clipped to the copper-to-edge clearance automatically.
4. Press **B** to fill all zones.

Zones do not update by themselves. Press **B** again after any change, and before exporting.

### Zone settings

| Setting | Value | Why |
|---|---|---|
| Clearance | `0.3` mm | Keeps the pour away from other nets |
| Minimum width | `0.25` mm | Avoids thin slivers of copper |
| Pad connections | Thermal reliefs | Spokes instead of a solid joint, so pads can still be soldered by hand |
| Remove islands | Always | Deletes copper pieces that connect to nothing |

### Practices

- **Stitch the layers together.** Add GND vias every 5 to 10 mm across the board and around its edge, so the top and bottom planes act as one.
- **High-current nets** can use a zone instead of a track, for example a supply rail running from the input connector to a regulator.
- **Check for islands and thin necks** after filling. A ground plane broken into pieces by tracks works poorly.

## Two-layer and four-layer boards

### Change the layer count

1. Open **File > Board Setup > Physical Stackup**.
2. Set **Copper layers** to `4` (or back to `2`), and click **OK**.
3. Optionally name the inner layers on the **Board Editor Layers** page, for example `In1.Cu` as `GND`.
4. Draw zones on the new inner layers.

KiCad only supports even numbers of copper layers. Going down from four to two deletes everything on the inner layers, so decide before any routing goes there.

### A standard four-layer stack

| Layer | Use |
|---|---|
| `F.Cu` | Parts and signals |
| `In1.Cu` | Solid ground plane, unbroken |
| `In2.Cu` | Power planes, or a second ground |
| `B.Cu` | Signals, and parts if any |

Keep `In1.Cu` as one solid ground zone. Signals on the top then always have ground directly below them.

### Which to choose

| Two layers | Four layers |
|---|---|
| Simple boards, low speed, few nets | USB and other fast signals, RF, sensitive analog |
| Lowest cost | Costs more, but routes far more easily |
| A solid ground pour on the bottom works well | A dedicated ground layer, with less noise and better EMC |

If a two-layer board is getting crowded and its ground is split everywhere, four layers is usually the better choice.

## Silkscreen

Silkscreen is the printed ink. It helps people assemble, test, and connect the board.

### Sizes

| Item | Minimum | Recommended |
|---|---|---|
| Text height | 1.0 mm | 1.0 to 1.5 mm |
| Text line thickness | 0.15 mm, and at least one-sixth of the height | 0.2 mm |
| Graphic line width | 0.15 mm | 0.15 to 0.2 mm |
| Distance from pads | 0.15 mm | 0.2 mm |

### What to print

- **Reference designators** next to each part, such as `R1` and `U2`.
- **Polarity and pin 1 marks:** diode bands, `+` on capacitors, a dot at pin 1 of chips, and connector pin 1.
- **Connector labels:** what each pin or connector is for, such as `GND`, `+24V`, `SDA`, or `MOTOR A`. This is the most useful silkscreen on the board.
- **Board name, revision, and date.** Use text variables such as `${TITLE}`, `${REVISION}`, and `${ISSUE_DATE}`, filled in under **File > Page Settings**, so they update in one place.
- **A logo or lab name,** if wanted. **Image Converter** in the main KiCad window turns a picture into a silkscreen footprint.

### Practices

- **Keep silkscreen off pads.** Ink over a pad is removed at the factory and can stop solder wetting. DRC reports it.
- **Keep reference text upright,** using at most two orientations, so it can be read with no more than one turn of the board.
- **Put each label clearly beside its part,** never underneath, where the part hides it.
- **On crowded boards,** keep polarity marks on the silkscreen and move reference designators to the `F.Fab` layer, so they still appear in assembly drawings.
- **Text on the back** goes on `B.Silkscreen`. KiCad mirrors it automatically so it reads correctly from below.

## Design rules check

**Inspect > Design Rules Checker**, then **Run DRC**.

Tick **Refill all zones before performing DRC** and **Test for parity between PCB and schematic**. Fix everything it reports, or exclude an item only when you're sure it's fine.

| Message | Usual cause and fix |
|---|---|
| Missing connection / unconnected items | A ratsnest line is still unrouted. Route it, or check the zone filled |
| Clearance violation | Two nets too close. Move the track, or narrow it |
| Courtyards overlap | Two parts too close. Move one |
| Silkscreen overlap / clipped by solder mask | A label sits on a pad. Move the text |
| Track has unconnected end | A dangling stub. Delete it |
| Hole too close to edge, or copper too close to edge | Move the hole or track inward |
| Board has malformed outline | `Edge.Cuts` is not one closed shape. Find the gap and close it |
| Footprint missing or extra / not in schematic | The board and schematic differ. Update the PCB from the schematic (F8) |

## 3D check

Open **View > 3D Viewer** (Alt+3) before exporting.

- Every part should have a model. A footprint with no model shows only its pads.
- Connectors face outward and overhang the edge as intended.
- Tall parts don't collide with each other or with mounting hardware.
- Polarized parts and chips are rotated the right way.

## Tool cheat sheet

Press **Ctrl+F1** in the PCB Editor for the full hotkey list. Hotkeys can be changed under **Preferences > Hotkeys**.

### Setup and checks

| Action | Hotkey | Menu |
|---|---|---|
| Update PCB from schematic | F8 | Tools |
| Board setup | | File > Board Setup |
| Design rules check | | Inspect > Design Rules Checker |
| 3D viewer | Alt+3 | View |
| Show all hotkeys | Ctrl+F1 | Help |

### Selecting and moving

| Action | Hotkey |
|---|---|
| Move | M |
| Drag a footprint, keeping its tracks attached | D |
| Rotate counter-clockwise / clockwise | R / Shift+R |
| Flip to the other side of the board | F |
| Properties of the selected item | E |
| Get and move a footprint by reference | T |
| Pack and move selected footprints | P |
| Select connected copper (press twice for the whole net) | U |
| Highlight a net | ` (backtick) |
| Clear net highlight | ~ |

### Routing and copper

| Action | Hotkey |
|---|---|
| Route a single track | X |
| Route a differential pair | 6 |
| Add a via while routing | V |
| Switch bend direction | / |
| Next / previous track width | W / Shift+W |
| Toggle 45° / free angle | Shift+Space |
| Draw a filled zone | Ctrl+Shift+Z |
| Fill all zones | B |
| Unfill all zones | Ctrl+B |

### View and grid

| Action | Hotkey |
|---|---|
| Zoom to fit the board | Home |
| Next / previous grid | N / Shift+N |
| Fast grid 1 / 2 | Alt+1 / Alt+2 |
| Toggle grid overrides | Ctrl+Shift+G |
| Measure a distance | Ctrl+Shift+M |
| Switch to the top / bottom copper layer | Page Up / Page Down |

## Layers you draw on

The full description of every layer is in [Footprint.md](Footprint.md#the-layers). On the board itself, you mostly work on these:

| Layer | What you draw there |
|---|---|
| `Edge.Cuts` | The board outline and any cutouts |
| `F.Cu` / `B.Cu` | Tracks and zones |
| `In1.Cu` / `In2.Cu` | Planes, on four-layer boards |
| `F.Silkscreen` / `B.Silkscreen` | Labels, revision text, and logos |
| `User.Drawings` / `User.Comments` | Notes and dimensions that aren't manufactured |

Parts bring their own `F.Fab`, `F.Courtyard`, paste, and mask graphics with them.

## Common mistakes

- **Rules not set for the manufacturer.** DRC passes, but the board breaks the factory's limits.
- **Zones not refilled** before export, so the ground plane is missing or out of date.
- **Board and schematic out of step,** from forgetting to update the PCB after changing the schematic.
- **Outline not closed,** or drawn on the wrong layer.
- **Power traces too thin** for their current.
- **Decoupling capacitors far from their chip.**
- **A ground plane chopped up** by tracks on the bottom layer.
- **Connectors facing inward,** or set back too far from the edge to plug in.
- **Copper or parts under a screw head.**
- **Silkscreen on pads,** too small to read, or missing polarity marks.
- **No connector labels,** so nobody knows which pin is which.

## Checklist before you export

- [ ] The schematic's ERC is clean, and the board is updated from it (F8).
- [ ] Board Setup uses the JLCPCB rules, net classes, and layer count.
- [ ] The outline on `Edge.Cuts` is closed, and the board shows in the 3D viewer.
- [ ] Mounting holes are where the enclosure needs them.
- [ ] Connectors are at the edge, facing out.
- [ ] Decoupling capacitors sit next to their chips.
- [ ] Power tracks are wide enough for their current.
- [ ] Ground zones are drawn, stitched, and filled (B).
- [ ] Silkscreen labels are readable and off the pads, and include polarity marks and connector labels.
- [ ] The board name, revision, and date are on the silkscreen.
- [ ] DRC reports 0 errors, with zone refill and schematic parity ticked.
- [ ] The 3D view looks right.

Next: [Manufacture.md](Manufacture.md).
