# Making a PCB footprint

How to draw a footprint for `my_footprints` when no download exists for the part, and the rules that keep it manufacturable. For importing a downloaded part, see the [README](README.md#add-a-part-to-the-library). For the matching symbol, see [Symbol.md](Symbol.md).

These instructions are written for KiCad 10, with JLCPCB as the manufacturer.

- [What a footprint is](#what-a-footprint-is)
- [Before you draw](#before-you-draw)
- [Through-hole pads](#through-hole-pads)
- [Surface-mount pads](#surface-mount-pads)
- [Pad numbering](#pad-numbering)
- [The other layers](#the-other-layers)
- [Draw the footprint](#draw-the-footprint)
- [Link the pads to the symbol pins](#link-the-pads-to-the-symbol-pins)
- [Common mistakes](#common-mistakes)
- [Checklist before you commit](#checklist-before-you-commit)

## What a footprint is

A footprint is everything one part needs on the board: the copper pads, the holes, and the outlines. It connects to its schematic symbol through pad numbers. A pad connects to the symbol pin that has the same Number, and nothing else links the two.

## Before you draw

- **Reuse first.** Standard packages such as SOIC, QFN, and 0.1" pin headers are already in KiCad's built-in libraries. Use those directly.
- **Start from a near match.** If a built-in footprint is close, open it, click **File > Save As**, save it into `my_footprints` under a new name, and edit the copy. Never edit the built-in one.
- **Use the datasheet's recommended PCB layout.** It is a separate drawing from the one of the part itself, usually titled "recommended land pattern" or "PCB layout".
- **Check which side the drawing shows.** Footprints are drawn looking down at the top of the board. If the drawing is labelled "bottom view", mirror it.
- **Work in the datasheet's units.** Switch the editor between mm and mils so you can type the datasheet's numbers directly, without converting.
- **Use nominal dimensions for positions**, and the largest lead size when choosing a hole.

## Through-hole pads

| Item | Guideline |
|---|---|
| Hole, round lead | Largest lead diameter + 0.2 mm |
| Hole, square or flat pin | The pin's diagonal + about 0.1 mm |
| Pad diameter | Hole + 0.7 mm is typical |
| Smallest pad for JLCPCB | Hole + 0.5 mm, which leaves a 0.25 mm copper ring |
| 0.1" (2.54 mm) header pin | 1.0 mm hole, 1.7 mm pad |
| Pad 1 shape | Square or rounded rectangle |
| All other pads | Round, or oval where the pitch is tight |
| Wide blade pins | An oval (slot) hole instead of a round one |

Round hole sizes up to the nearest 0.05 mm. A hole that is slightly large still solders well. A hole that is too small means the part does not fit.

Plastic locating pegs and screw holes get a different pad type:

| Setting | Value |
|---|---|
| Pad type | NPTH, Mechanical |
| Hole | Peg diameter + 0.1 mm |
| Number | Left empty |

## Surface-mount pads

- **Size and position:** copy the datasheet's land pattern exactly. Pads are normally a little longer than the part's leads, so solder can form a visible fillet at the end.
- **Shape:** rounded rectangle.
- **Paste and mask:** leave the pad's own clearance settings at 0, so they follow the board's defaults.
- **Large centre pads:** connect them as the datasheet says, usually to ground. Give the pad its own Number.
- **Gap between pads:** keep at least 0.2 mm of bare board between neighbouring pads.

## Pad numbering

The Number on each pad has to match a pin Number on the symbol exactly. Beyond that, follow the convention for the part type:

| Part type | Convention |
|---|---|
| Chips | Pin 1 at the marked corner, then counter-clockwise viewed from above |
| Connectors | The numbering in the manufacturer's datasheet |
| Modules and dev boards | The labels printed on the board, or 1 to N around the edge. Either works if the symbol uses the same |

A Number can be text, such as `GND1` or `VIN`.

## The other layers

Pads sit on the copper layer. Three more layers make the footprint usable.

| Layer | What goes on it | Line width | Rule |
|---|---|---|---|
| `F.Fab` | The exact outline of the part's body | 0.1 mm | Add the text `${REFERENCE}` inside it |
| `F.SilkS` | The outline printed on the board, and the pin 1 mark | 0.15 mm | At least 0.2 mm clear of every pad |
| `F.CrtYd` | A keep-out box around the body and all pads | 0.05 mm | A closed shape, 0.25 mm outside everything |

More on each:

- **Fab** is not printed. It shows the true size of the part in assembly drawings.
- **Silkscreen** is printed, so it must not cross a pad, or it is cut away. Put the pin 1 mark where it is still visible after the part is soldered on. Text should be at least 1 mm tall.
- **Courtyard** is what the design rule check uses to stop two parts overlapping. Use 0.5 mm of margin for connectors, to leave room for the mating plug and your fingers.

## Draw the footprint

1. Open the **Footprint Editor** and click `my_footprints` in the library list.
2. Click **File > New Footprint**. Enter a name and choose through-hole or SMD.
3. Click **Place > Pad** and place one pad.
4. Double-click the pad and set its Number, pad type, shape, size, and hole. Type its exact X and Y position, instead of placing it by eye.
5. For a row of pads, select the pad and press **Ctrl+T** (Create Array). Enter the count and the pitch, and let it number the pads in order.
6. Check the numbering that the array produced, and edit any pad whose Number should be different.
7. Set the origin with **Place > Anchor**: on pad 1 for through-hole parts, at the centre of the body for SMD parts.
8. Draw the body outline on `F.Fab`, the silkscreen on `F.SilkS`, and the courtyard on `F.CrtYd`.
9. Click **File > Footprint Properties**. Add a one-line description, and attach the 3D model as described in the [README](README.md#4-attach-the-3d-model).
10. Click **Inspect > Footprint Checker** and fix anything it reports.
11. Press **Ctrl+S**.
12. Print the footprint at 100% scale and set the real part on the paper. Every pin should land on its pad.

Use the measure tool to confirm the distances that matter most: pad pitch, the spacing between rows, and the overall span.

## Link the pads to the symbol pins

There is no mapping table. The link is the Number.

1. In the Symbol Editor, open the symbol and click **Edit > Pin Table**. This is your list of pin Numbers.
2. In the footprint, set each pad's **Number** to the matching pin Number, character for character.
3. In the symbol, set the **Footprint** field to:

   ```
   my_footprints:FOOTPRINT_NAME
   ```

How the match behaves:

| Situation | Result |
|---|---|
| Pad Number equals a pin Number | The pad joins that pin's net |
| Several pads share one Number | All of them connect to that one pin. Used for shield tabs and split thermal pads |
| Pad with an empty Number | Mechanical only, no connection |
| Pad Number that no pin has | The pad is left unconnected |
| Pin Number that no pad has | KiCad reports an error when you update the board |

To test the link:

1. Place the symbol in a schematic and wire a few of its pins to labelled nets.
2. Click **Tools > Update PCB from Schematic** and read the messages for errors.
3. In the PCB Editor, check that each pad shows the net name of the pin it belongs to.

## Common mistakes

- **Mirrored footprint:** drawn from a bottom-view drawing without flipping it. The part only fits on the wrong side of the board.
- **Holes too small:** sized to the nominal lead instead of the largest one, or to the side of a square pin instead of its diagonal.
- **Wrong row spacing:** the pitch along a row is right but the distance between rows was misread. Measure both.
- **Pad Numbers that don't match the symbol:** `1` on the footprint and `GND1` on the symbol, for example.
- **Silkscreen over pads:** the outline crosses a pad and gets removed at the factory.
- **No courtyard:** the design rule check then cannot warn you when two parts overlap.
- **3D model path from one computer:** always start the path with `${MY_LIBS}`.

## Checklist before you commit

- [ ] The footprint is drawn as seen from the top of the board.
- [ ] Pad pitch, row spacing, and overall span match the datasheet.
- [ ] Each hole fits the largest lead, and each pad leaves at least a 0.25 mm ring.
- [ ] Every electrical pad has a Number that matches a symbol pin.
- [ ] Pad 1 is a different shape, and marked on the silkscreen.
- [ ] Fab outline, silkscreen, and courtyard are all drawn.
- [ ] No silkscreen crosses a pad.
- [ ] The 3D model path starts with `${MY_LIBS}` and the model sits on the pads.
- [ ] **Inspect > Footprint Checker** reports nothing.
- [ ] A 1:1 printout matches the real part.
