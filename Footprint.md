# Making a PCB footprint

How to draw a footprint for `my_footprints` when no download exists for the part. For importing a downloaded part, see the [README](README.md#add-a-part-to-the-library). For the matching symbol, see [Symbol.md](Symbol.md).

These instructions are written for KiCad 10, with JLCPCB as the manufacturer.

- [How a footprint links to its symbol](#how-a-footprint-links-to-its-symbol)
- [Before you draw](#before-you-draw)
- [Common footprints in KiCad's libraries](#common-footprints-in-kicads-libraries)
- [Coordinates](#coordinates)
- [The layers](#the-layers)
- [Pad standards](#pad-standards)
- [The example part](#the-example-part)
- [Draw the footprint](#draw-the-footprint)
- [Use the Footprint Wizard](#use-the-footprint-wizard)
- [Reuse one of KiCad's 3D models](#reuse-one-of-kicads-3d-models)
- [Link the footprint to the symbol](#link-the-footprint-to-the-symbol)
- [How the editor behaves](#how-the-editor-behaves)
- [Common mistakes](#common-mistakes)
- [Checklist before you commit](#checklist-before-you-commit)

## How a footprint links to its symbol

A footprint is everything one part needs on the board: copper pads, holes, and outlines. It connects to its schematic symbol in exactly one way:

**A pad connects to the symbol pin that has the same Number.**

There is no mapping table and no other setting. If the symbol has a pin with Number `7`, the footprint needs a pad with Number `7`, and that pad lands on that pin's net.

- **The match is exact.** `VIN` and `Vin` are different Numbers. So are `1` and `01`.
- **Numbers can be text.** `GND1` is a valid Number, as long as both sides use it.
- **Pin Names play no part.** Only the Number links a pin to a pad.

Decide the Numbers once, on the symbol, and copy them onto the pads.

## Before you draw

- **Reuse first.** Standard packages such as SOIC, QFN, and 0.1" pin headers are already in KiCad's built-in libraries. Use those directly.
- **Start from a near match.** If a built-in footprint is close, open it, click **File > Save As**, save it into `my_footprints` under a new name, and edit the copy. Never edit the built-in one.
- **Use the Footprint Wizard** for standard shapes, such as SOIC or QFN, when KiCad's libraries don't have the exact size. See [Use the Footprint Wizard](#use-the-footprint-wizard).
- **Use the datasheet's recommended PCB layout.** It is a separate drawing from the one of the part itself, usually titled "recommended land pattern" or "PCB layout". Development boards and modules often have a "dimensions" drawing instead.
- **Check which side the drawing shows.** Footprints are drawn looking down at the top of the board. If the drawing is labelled "bottom view", mirror it.
- **Find out what each dimension is measured from.** Some are from the board edge and some are from the centre of the first hole. Mixing them up shifts everything.
- **Work in the datasheet's units.** Switch the editor between mm and mils so you can type the datasheet's numbers without converting.

## Common footprints in KiCad's libraries

Most small parts on a board use standard packages that KiCad already has. Use these directly, and only draw or download a footprint for parts that aren't standard.

To use one, set the symbol's **Footprint** field to the name below, or search for the short name, such as `R_0603`, in the footprint chooser. **Tools > Assign Footprints** in the Schematic Editor does many parts at once.

### Chip sizes for resistors, capacitors, and similar parts

Small rectangular parts are named by their size, using a four-digit code. Shops such as LCSC list the imperial code. KiCad's footprint names include both codes.

| Imperial code | Metric code | Size (mm) | Hand soldering | Typical use |
|---|---|---|---|---|
| `0402` | `1005` | 1.0 × 0.5 | Hard | Dense boards assembled by JLCPCB |
| `0603` | `1608` | 1.6 × 0.8 | Possible with care | **The usual default.** Small, and widely stocked as JLCPCB Basic parts |
| `0805` | `2012` | 2.0 × 1.25 | Easy | Hand-assembled boards, and larger capacitor values |
| `1206` | `3216` | 3.2 × 1.6 | Very easy | Higher power or voltage, fuses, large capacitors |

Larger parts handle more power and voltage. Typical resistor power ratings:

| Size | Typical power rating |
|---|---|
| `0402` | 1/16 W |
| `0603` | 1/10 W |
| `0805` | 1/8 W |
| `1206` | 1/4 W |

Ceramic capacitors lose much of their capacitance when a voltage is applied, and small packages lose more. For large values such as 10 µF and above, choose `0805` or bigger, and check the part's datasheet.

KiCad also has a `_HandSolder` version of each chip footprint, with longer pads that are easier to solder with an iron. Use those on boards you'll assemble by hand.

### Passive parts

| Part | Footprint |
|---|---|
| Resistor | `Resistor_SMD:R_0603_1608Metric` (change the size code as needed) |
| Ceramic capacitor | `Capacitor_SMD:C_0603_1608Metric` |
| Inductor or ferrite bead | `Inductor_SMD:L_0805_2012Metric` |
| Fuse | `Fuse:Fuse_1206_3216Metric` |
| Electrolytic capacitor | `Capacitor_SMD:CP_Elec_6.3x5.4`. The numbers are the can's diameter and height in mm |
| Crystal | `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm` |

### Diodes and LEDs

| Part | Footprint |
|---|---|
| LED | `LED_SMD:LED_0603_1608Metric` |
| Small signal or Schottky diode | `Diode_SMD:D_SOD-123`, or `D_SOD-323` for smaller |
| Power diode or TVS diode | `Diode_SMD:D_SMA`. `D_SMB` and `D_SMC` are larger, for more power |

### Transistors and regulators

| Part | Footprint |
|---|---|
| Small transistor or MOSFET, 3 pins | `Package_TO_SOT_SMD:SOT-23` |
| Small regulator or logic chip, 5 or 6 pins | `Package_TO_SOT_SMD:SOT-23-5`, `SOT-23-6` |
| Linear regulator, medium power | `Package_TO_SOT_SMD:SOT-223-3_TabPin2` |
| Medium-power transistor or regulator | `Package_TO_SOT_SMD:SOT-89-3` |
| Power MOSFET or regulator (DPAK) | `Package_TO_SOT_SMD:TO-252-2` |

### Chips

| Package | Footprint | Pin pitch |
|---|---|---|
| SOIC-8 | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` | 1.27 mm |
| SOIC-16 | `Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` | 1.27 mm |
| MSOP-8 | `Package_SO:MSOP-8_3x3mm_P0.65mm` | 0.65 mm |
| TSSOP-16 | `Package_SO:TSSOP-16_4.4x5mm_P0.65mm` | 0.65 mm |
| QFN | `Package_DFN_QFN` library. Search for the pin count, such as `QFN-32` | Usually 0.5 mm |

The numbers in a chip footprint's name give its body size and pin pitch. For example, `SOIC-8_3.9x4.9mm_P1.27mm` is 8 pins, a 3.9 × 4.9 mm body, and 1.27 mm pitch. Match all three against the datasheet before choosing one.

### Connectors and test points

| Part | Footprint |
|---|---|
| 0.1" pin header | `Connector_PinHeader_2.54mm:PinHeader_1x04_P2.54mm_Vertical` (change the pin count as needed) |
| JST PH, 2 mm pitch | `Connector_JST:JST_PH_B2B-PH-K_1x02_P2.00mm_Vertical` |
| JST XH, 2.5 mm pitch | `Connector_JST:JST_XH_B2B-XH-A_1x02_P2.50mm_Vertical` |
| Test point | `TestPoint:TestPoint_Pad_D1.5mm` |

Connector footprints must match the exact part number of the connector you buy. Check the manufacturer's part number in the footprint's name against the datasheet.

Mounting holes are covered in [PCB.md](PCB.md#mounting-holes).

### Choosing a size

- **For boards JLCPCB will assemble,** use `0603` by default and `0402` where space is tight. Check that the part you want is a Basic part in that size.
- **For boards you'll solder by hand,** use `0805` and the `_HandSolder` footprints.
- **Keep sizes consistent across a board.** Fewer sizes means fewer mistakes and simpler ordering.
- **Check the package against the part.** The footprint must match the size printed in the part's datasheet or LCSC listing. A `0603` resistor won't fit a `0805` footprint well.

## Coordinates

Every pad and line is placed by typing an X and Y position, not by eye.

| Rule | Detail |
|---|---|
| Origin | Position 0, 0 is the footprint's anchor point |
| X | Increases to the right |
| Y | Increases **downward** |
| Through-hole parts | Put pad 1 at 0, 0 |
| Surface-mount parts | Put the centre of the body at 0, 0 |

To turn a drawing into positions, measure everything from pad 1:

- A row with 2.54 mm pitch running down the page has pads at Y = `0`, `2.54`, `5.08`, `7.62`, and so on.
- A second row 7.62 mm to the right has the same Y values at X = `7.62`.
- A board edge 1.27 mm above and left of pad 1 is at X = `-1.27`, Y = `-1.27`.

## The layers

`F.` means the front (top) of the board and `B.` means the back (bottom).

| Layer | What it is |
|---|---|
| `F.Cu` / `B.Cu` | Copper on the top and bottom: pads and traces |
| Inner layers | Copper inside the board, on boards with four or more layers |
| `F.Adhesive` / `B.Adhesive` | Glue dots used by some assembly processes. Rarely used |
| `F.Paste` / `B.Paste` | Where solder paste is applied to surface-mount pads |
| `F.Silkscreen` / `B.Silkscreen` | Ink printed on the board: outlines, labels, pin 1 marks |
| `F.Mask` / `B.Mask` | Openings in the solder mask (the coloured coating), exposing copper for soldering |
| `User.Drawings` | Your own drawings and dimensions. Not manufactured |
| `User.Comments` | Notes. Not manufactured |
| `User.Eco1` / `User.Eco2` | Spare layers. Not manufactured |
| `Edge.Cuts` | The outline of the whole board, and any cutouts the factory mills |
| `Margin` | An optional keep-back line inside the board edge |
| `F.Courtyard` / `B.Courtyard` | The keep-out box around each part, used to detect overlapping parts |
| `F.Fab` / `B.Fab` | The true outline of each part's body, for assembly drawings. Not printed |
| `User.1` to `User.4` | More spare layers. Not manufactured |

When you make a footprint, you only draw on three of them: `F.Fab`, `F.Silkscreen`, and `F.Courtyard`. The pads look after copper, mask, and paste themselves.

Never draw a part's outline on `Edge.Cuts`. That tells the factory to cut a hole of that shape through your board.

## Pad standards

### Through-hole pads

| Item | Guideline |
|---|---|
| Hole, round lead | Largest lead diameter + 0.2 mm |
| Hole, square or flat pin | The pin's diagonal + about 0.1 mm |
| Pad diameter | Hole + 0.7 mm is typical |
| Smallest pad for JLCPCB | Hole + 0.5 mm, which leaves a 0.25 mm copper ring |
| 0.1" (2.54 mm) header pin | 1.0 mm hole, 1.7 mm pad |
| Pad 1 shape | Rectangular |
| All other pads | Circular, or oval where the pitch is tight |
| Wide blade pins | An oval (slot) hole instead of a round one |

Round hole sizes up to the nearest 0.05 mm. A hole that is slightly large still solders well. A hole that is too small means the part does not fit.

### Surface-mount pads

- **Size and position:** copy the datasheet's land pattern exactly. Pads are normally a little longer than the part's leads, so solder can form a visible fillet at the end.
- **Shape:** rounded rectangle.
- **Paste and mask:** leave the pad's own clearance settings at 0, so they follow the board's defaults.
- **Large centre pads:** give the pad its own Number and connect it as the datasheet says, usually to ground.
- **Gap between pads:** keep at least 0.2 mm of bare board between neighbouring pads.

### Mounting pegs and screw holes

| Setting | Value |
|---|---|
| Pad type | NPTH, Mechanical |
| Hole | Peg diameter + 0.1 mm |
| Number | Left empty |

### Numbering conventions

| Part type | Convention |
|---|---|
| Chips | Pin 1 at the marked corner, then counter-clockwise viewed from above |
| Connectors | The numbering in the manufacturer's datasheet |
| Modules and dev boards | The labels printed on the board, or 1 to N around the edge |

Whichever you choose, the symbol has to use the same Numbers.

## The example part

The steps below use one made-up part so the numbers are concrete: a small plug-in module on standard 0.1" header pins.

| Property | Value |
|---|---|
| Pins | 8, in two rows of 4 |
| Pitch along a row | 2.54 mm |
| Distance between rows | 7.62 mm |
| Board edge | 1.27 mm beyond the pin centres on every side |
| Numbering | 1 to 4 down the left row, 5 to 8 up the right row |

Viewed from above, with each pad's position:

```
            X = 0     X = 7.62
Y = 0        [1]        (8)
Y = 2.54     (2)        (7)
Y = 5.08     (3)        (6)
Y = 7.62     (4)        (5)
```

Replace these values with the ones from your own datasheet.

## Draw the footprint

### 1. Create it

1. Open the **Footprint Editor** and click `my_footprints` in the library list.
2. Click **File > New Footprint**.
3. Enter a name and choose through-hole or SMD. Use the same name as the symbol where you can.

### 2. Place the first pad

1. Click **Place > Pad**, click once on the canvas, and press **Esc**.
2. Double-click the pad and fill in its properties.

For the example part, pad 1 is:

| Setting | Value |
|---|---|
| Number | `1` |
| Pad type | Through-hole |
| Shape | Circular for now. It becomes Rectangular at the end of step 3 |
| Position X, Y | `0`, `0` |
| Pad size | `1.7` |
| Hole | Round, `1.0` |

Every pad you place afterwards starts with the settings of the last pad you edited. You then only change its Number, its position, and its shape if needed.

### 3. Build each row with Create Array

A row of evenly spaced pads is one pad copied several times. Select the first pad of the row and press **Ctrl+T**.

Set the dialog like this:

| Setting | Value |
|---|---|
| Horizontal count / Vertical count | How many pads across and down. Use `1` for the direction you are not copying in |
| Horizontal spacing / Vertical spacing | The pitch. A negative value copies up or to the left |
| Grid Position | **Source items remain in place** |
| Item Source | Duplicate selection |
| Renumber pads | Ticked |
| Initial Pad Number | **From start value** |
| Pad Numbering Scheme | Continuous (1, 2, 3...) |
| Pad numbering start | The Number of the pad you selected |
| Pad numbering increment | `1` |

Two of these matter most:

- **Source items remain in place.** The other choice, Centre on source items, spreads the copies on both sides of your pad, which shifts the whole row.
- **From start value.** This makes the row count up from the Number you give it.

For the example part, the left row counts downward and the right row counts upward, so the right row is built from its bottom pad with a negative spacing:

| Row | First pad Number | First pad X, Y | Vertical count | Vertical spacing | Numbering start | Result |
|---|---|---|---|---|---|---|
| Left | `1` | `0`, `0` | `4` | `2.54` | `1` | `1` at the top, `4` at the bottom |
| Right | `5` | `7.62`, `7.62` | `4` | `-2.54` | `5` | `5` at the bottom, `8` at the top |

For the right row, first place a new pad, set its Number to `5` and its position to `7.62`, `7.62`, then press **Ctrl+T** on it.

Afterwards:

1. Read the Numbers along each row and confirm they are in order.
2. Place any pads that don't fit a sequence, such as `GND1` or `VIN`, one at a time. Set each one's Number and position by hand.
3. Double-click pad 1 and change its shape to Rectangular. This is done last because an array copies the shape of the pad it starts from.

If a part's Numbers jump around, split each row into runs that do count in order, and make one array per run.

### 4. Draw the outlines on three layers

A footprint needs a border on three layers and nothing more. Each one is usually a single rectangle.

| Layer | What to draw | How to size it | Line width |
|---|---|---|---|
| `F.Fab` | The true outline of the part's body | Exactly the body size from the datasheet | `0.1` |
| `F.Silkscreen` | The outline as printed on the board | The body outline, kept at least 0.2 mm away from every pad | `0.15` |
| `F.Courtyard` | The keep-out area | The body and all pads, plus a margin on every side | `0.05` |

Use a courtyard margin of 0.25 mm for ordinary parts and 0.5 mm for connectors and plug-in modules. Add more on any side where something sticks out past the body, such as a cable or a connector that overhangs.

To draw each rectangle:

1. Click the layer's name in the **Layers** panel on the right, so it becomes the active layer.
2. Click **Place > Rectangle** and draw a rectangle of any size.
3. Double-click it, type the two corner positions and the line width, and click **OK**.

For the example part, the board edge is 1.27 mm beyond the pin centres, so the body runs from 0 − 1.27 to 7.62 + 1.27 in both directions:

| Layer | Corner 1 X, Y | Corner 2 X, Y | Line width | Worked out as |
|---|---|---|---|---|
| `F.Fab` | `-1.27`, `-1.27` | `8.89`, `8.89` | `0.1` | The body outline |
| `F.Silkscreen` | `-1.27`, `-1.27` | `8.89`, `8.89` | `0.15` | The same outline. The pads are 0.42 mm inside it, so it clears them |
| `F.Courtyard` | `-1.77`, `-1.77` | `9.39`, `9.39` | `0.05` | The body outline plus 0.5 mm |

Then add an orientation mark on `F.Silkscreen`, so the part cannot be fitted backwards. Use whichever suits the part:

- **A dot or short line beside pad 1**, for chips and small parts.
- **An outline of a distinctive feature**, such as a connector or a notch, for modules.

The mark should still be visible after the part is soldered on. If a silkscreen line would cross a pad, shorten it or move it outside the pads.

### 5. Position the text labels

KiCad adds three text labels to every new footprint. Keep all three, and move them so they sit clear of the pads.

| Text | Layer | What it becomes | Where to put it |
|---|---|---|---|
| `REF**` | `F.Silkscreen` | The part's reference, such as `U1`, printed on the board | Just outside the outline, where the part will not cover it |
| The footprint's name | `F.Fab` | The part's value, in assembly drawings only | Inside or beside the outline |
| `${REFERENCE}` | `F.Fab` | The reference again, in assembly drawings only | Inside the outline |

Only `REF**` is printed on the real board.

### 6. Attach the 3D model

Follow [step 4 of the README](README.md#4-attach-the-3d-model). The path must start with `${MY_LIBS}`.

For a standard package, you can reuse one of KiCad's own models instead of finding one. See [Reuse one of KiCad's 3D models](#reuse-one-of-kicads-3d-models).

### 7. Check and save

1. Click **Inspect > Footprint Checker** and fix anything it reports.
2. Use the measure tool to confirm the pitch, the distance between rows, and the overall span.
3. Press **Ctrl+S**.
4. Print the footprint at 100% scale and set the real part on the paper. Every pin should land on its pad.

## Use the Footprint Wizard

The Footprint Wizard builds common package shapes from a handful of numbers. It places and numbers the pads and draws the outlines for you. It has wizards for families such as SOIC / SSOP / TSSOP, DIP, QFN, QFP, and BGA.

Use it when a part has a standard shape but KiCad's libraries don't have the exact size. Check the built-in libraries first. For a common package such as SOIC-8, KiCad already has the footprint, and often the part's symbol too, so you may not need to make anything.

### Steps

1. **Open the wizard.** In the Footprint Editor, click **File > New Footprint Using Footprint Wizard**, or the wizard button on the top toolbar.
2. **Choose the wizard.** Click the **Select Wizard** button at the top of the wizard window, and pick the one for your package family.
3. **Fill in the parameters** in the list on the left. The preview on the right updates as you type.
4. **Send it to the editor.** Click **Export footprint to editor** on the wizard's toolbar. The footprint opens in the Footprint Editor.
5. **Save it.** Click **File > Save As**, choose `my_footprints`, and give it a clear name.
6. **Check it and finish it** as below, then add a 3D model.

### Worked example: SOIC-8

The example part is the NXP TJA1051T CAN transceiver (LCSC `C38695`), in a SOIC-8 package with a 3.9 mm (150 mil) body. KiCad already has this exact footprint as `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm`, so in practice you'd use that. It makes a good example because you can compare your result against it.

Choose the SOIC / SSOP / TSSOP wizard, then fill in the parameters. Their names may differ slightly between versions, so match them by meaning:

| Parameter | Value | What it is |
|---|---|---|
| Number of pads | `8` | Total pins, 4 per side |
| Pitch | `1.27` | Distance between neighbouring pins |
| Row spacing | `4.95` | Pad centre to pad centre, across the body |
| Pad length | `1.95` | Pad size along the lead |
| Pad width | `0.6` | Pad size across the lead |

Where the numbers come from:

- **Pitch** is in the datasheet's package drawing, usually labelled `e`.
- **Pad size and row spacing** come from the datasheet's recommended land pattern, if it has one. The values above match KiCad's own SOIC-8 footprint, which follows the IPC-7351 standard.
- **Body size** (3.9 × 4.9 mm here) is in the package drawing, and is only used for the outlines.

Save it as, for example, `SOIC-8_3.9x4.9mm_P1.27mm`.

### Check what the wizard made

The wizard does the arithmetic, but check its output before using it:

- **Pad numbering:** pad 1 is marked, and the pads are numbered 1 to 4 down one side and 5 to 8 up the other, counter-clockwise viewed from above.
- **Dimensions:** use the measure tool (**Ctrl+Shift+M**) to confirm the pitch and row spacing.
- **Outlines:** `F.Fab`, `F.Silkscreen`, and `F.Courtyard` are all present. Add any that are missing, as in [step 4](#4-draw-the-outlines-on-three-layers).
- **Pad Numbers** match the symbol's pin Numbers. See [Link the footprint to the symbol](#link-the-footprint-to-the-symbol).
- **Compare with KiCad's version,** when there is one. Open the built-in footprint next to yours. They should match closely.
- **Run Inspect > Footprint Checker.**

## Reuse one of KiCad's 3D models

KiCad installs 3D models for all its standard packages. A footprint you made for a standard package, by hand or with the wizard, can use one of them.

### Option 1: Copy the model into the library (recommended)

1. **Find KiCad's model folder.** In the main KiCad window, open **Preferences > Configure Paths** and look at `KICAD10_3DMODEL_DIR`. On Windows it is usually:

   ```
   C:\Program Files\KiCad\10.0\share\kicad\3dmodels
   ```

2. **Copy the model.** Models are grouped by library. For SOIC-8, open `Package_SO.3dshapes` and copy `SOIC-8_3.9x4.9mm_P1.27mm.step` into `KiCAD_Library/custom/3dmodels/`.
3. **Attach it.** Open your footprint, click **File > Footprint Properties**, and open the **3D Models** tab.
4. **Enter the path.** Click **+** and enter the path to your copy:

   ```
   ${MY_LIBS}/custom/3dmodels/SOIC-8_3.9x4.9mm_P1.27mm.step
   ```

5. **Check the preview,** as below. Click **OK**, then press **Ctrl+S**.
6. **Commit both files:** the footprint and the `.step` model.

This works on every computer that has the library, and keeps working after a KiCad upgrade. KiCad's 3D models are licensed to allow this kind of reuse.

### Option 2: Point to KiCad's own copy

Skip the copy, and enter KiCad's path in step 4 instead:

```
${KICAD10_3DMODEL_DIR}/Package_SO.3dshapes/SOIC-8_3.9x4.9mm_P1.27mm.step
```

This is quicker, and works for anyone with KiCad 10 and its 3D models installed. The variable has the version number in it, though, so after an upgrade the model may go missing until the path is fixed. That's why Option 1 is preferred for this library.

### Check the preview

KiCad's models are centred where KiCad's own footprints put their origin. For surface-mount packages, that is the middle of the body.

- **If your footprint's origin is also at the centre,** the model sits on the pads without changes.
- **If the model is shifted** so its leads miss the pads, your footprint's origin is somewhere else, often pad 1. Fix it with **Offset X** and **Offset Y** until the leads land on the pads.
- **If pin 1 is at the wrong end,** check that the model's dot or chamfer is at pad 1's end. If it isn't, set **Rotation Z** to `180`.

## Link the footprint to the symbol

1. In the Symbol Editor, open the symbol and click **Edit > Pin Table**. The Number column is your list.
2. In the footprint, confirm there is a pad for every Number in that list, spelled identically.
3. In the symbol, click **File > Symbol Properties** and set the **Footprint** field to:

   ```
   my_footprints:FOOTPRINT_NAME
   ```

For the example part, the two sides line up like this:

| Symbol pin Number | Symbol pin Name | Footprint pad Number | Pad position X, Y |
|---|---|---|---|
| `1` | `VIN` | `1` | `0`, `0` |
| `2` | `GND` | `2` | `0`, `2.54` |
| `3` | `D0` | `3` | `0`, `5.08` |
| `4` | `D1` | `4` | `0`, `7.62` |
| `5` | `D2` | `5` | `7.62`, `7.62` |
| `6` | `D3` | `6` | `7.62`, `5.08` |
| `7` | `GND` | `7` | `7.62`, `2.54` |
| `8` | `3V3` | `8` | `7.62`, `0` |

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

## How the editor behaves

- **Pads ignore the active layer.** A pad's layers come from its pad type. A through-hole pad gets copper on every layer and mask openings on both sides, whichever layer is selected when you place it.
- **Drawn shapes use the active layer.** Lines, rectangles, and text land on whichever layer is selected. Select the layer before you draw.
- **A shape on the wrong layer is easy to fix.** Double-click it and change its layer. There is no need to redraw it.
- **New pads copy the last pad you edited**, including its size, hole, and shape.
- **Unsaved changes show as an asterisk** after the footprint's name in the library list.

## Common mistakes

- **Pad Numbers that don't match the symbol.** `1` on the footprint and `GND1` on the symbol, or `VIN` on one and `Vin` on the other.
- **Mirrored footprint.** Drawn from a bottom-view drawing without flipping it. The part only fits on the wrong side of the board.
- **Dimensions measured from the wrong place.** Taken from the board edge when the drawing measures from the first hole, or the other way round.
- **Holes too small.** Sized to the nominal lead instead of the largest one, or to the side of a square pin instead of its diagonal.
- **Wrong row spacing.** The pitch along a row is right but the distance between rows was misread. Measure both.
- **An array that shifted.** Create Array was left on Centre on source items, so the first pad is no longer where it was placed.
- **Part outline on `Edge.Cuts`.** The factory cuts it out of the board.
- **Silkscreen over pads.** The outline crosses a pad and is removed at the factory.
- **No courtyard.** The design rule check then cannot warn you when two parts overlap.
- **3D model path from one computer.** Always start the path with `${MY_LIBS}`.

## Checklist before you commit

- [ ] The footprint is drawn as seen from the top of the board.
- [ ] Pad 1 is at 0, 0 (through-hole) or the body is centred on 0, 0 (SMD).
- [ ] Pad pitch, row spacing, and overall span match the datasheet.
- [ ] Each hole fits the largest lead, and each pad leaves at least a 0.25 mm ring.
- [ ] Every electrical pad has a Number, and each Number matches a symbol pin exactly.
- [ ] Pad 1 is rectangular and the rest are not.
- [ ] `F.Fab`, `F.Silkscreen`, and `F.Courtyard` each have an outline, and nothing is on `Edge.Cuts`.
- [ ] No silkscreen crosses a pad, and there is an orientation mark.
- [ ] `REF**` sits outside the outline.
- [ ] The 3D model path starts with `${MY_LIBS}` and the model sits on the pads.
- [ ] **Inspect > Footprint Checker** reports nothing.
- [ ] A 1:1 printout matches the real part.
- [ ] The symbol's Footprint field is set to `my_footprints:FOOTPRINT_NAME`.
