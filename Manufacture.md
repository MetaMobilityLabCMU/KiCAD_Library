# Manufacturing with JLCPCB

How to turn a finished board into the files JLCPCB needs, check them, and place the order. Finish the board first with [PCB.md](PCB.md), including its checklist.

These instructions are written for KiCad 10. JLCPCB's order page and services change from time to time, so treat its option names here as a guide and read the page as you go.

- [What JLCPCB needs](#what-jlcpcb-needs)
- [Before you export](#before-you-export)
- [Prepare parts for assembly](#prepare-parts-for-assembly)
- [Export with Fabrication Toolkit](#export-with-fabrication-toolkit)
- [Export with KiCad alone](#export-with-kicad-alone)
- [Check the files](#check-the-files)
- [Order bare boards](#order-bare-boards)
- [Order assembly](#order-assembly)
- [Fix rotated or shifted parts](#fix-rotated-or-shifted-parts)
- [Record what you ordered](#record-what-you-ordered)
- [Common problems](#common-problems)
- [Checklist before you pay](#checklist-before-you-pay)

## What JLCPCB needs

| Order | Files | What they are |
|---|---|---|
| Bare boards | Gerbers and drill files, in one `.zip` | One Gerber file per layer, plus Excellon files listing every hole |
| Boards with assembly | The `.zip` above, plus a BOM and a CPL | The BOM lists which part goes in each position. The CPL (also called the position or pick-and-place file) gives each part's X, Y, rotation, and side |
| A solder paste stencil | The `.zip`, including the paste layers | Paste layers show where solder paste goes |

There are two ways to produce these files:

| Route | Use it for |
|---|---|
| [Fabrication Toolkit](#export-with-fabrication-toolkit), recommended | Everything, in one click, already in JLCPCB's format |
| [KiCad alone](#export-with-kicad-alone) | When the plugin isn't available, or to see exactly what each setting does |

## Before you export

- [ ] **DRC reports 0 errors**, with **Refill all zones** and **Test for parity between PCB and schematic** ticked.
- [ ] **Zones are filled.** Press **B** in the PCB Editor.
- [ ] **The board outline** on `Edge.Cuts` is one closed shape. Without it, JLCPCB holds the order.
- [ ] **The revision text** on the silkscreen is updated for this order.
- [ ] **The project is saved and committed**, so you can find this exact version later.

## Prepare parts for assembly

Skip this section if you are only ordering bare boards.

### Part numbers

Every part JLCPCB will place needs its LCSC part number in a symbol field named exactly `LCSC Part #`. The number is the letter `C` followed by digits, such as `C25804`.

The quickest way to fill many at once is the Symbol Fields Table:

1. In the Schematic Editor, click **Tools > Edit Symbol Fields Table**.
2. Click **Add Field** and name it `LCSC Part #`, if it isn't already there.
3. Type each part's number into its row. Parts with the same value and footprint share a row, so one entry covers all of them.

Look up numbers on the [JLCPCB parts library](https://jlcpcb.com/parts). Each part is listed as one of two types:

| Type | Cost |
|---|---|
| Basic | Already loaded on JLCPCB's machines. No extra setup fee |
| Extended | Loaded specially for your order. A setup fee for each different Extended part |

Choose Basic parts where you can, especially for resistors and capacitors.

### Parts you will fit yourself

Through-hole connectors, modules, and anything you'd rather solder by hand should be left out of JLCPCB's files. Either:

- **On the footprint:** in the PCB Editor, open the footprint's properties (E) and tick **Exclude from position files** and **Exclude from bill of materials**, or
- **On the symbol:** tick **Do not populate** in its properties, and use Fabrication Toolkit's **Exclude DNP components from BOM** option.

The board still gets the pads and holes for these parts. Only the assembly files leave them out.

## Export with Fabrication Toolkit

Install it once from the **Plugin and Content Manager** in the main KiCad window: search for `Fabrication Toolkit`, click **Install**, then **Apply Pending Changes**.

### Run it

1. In the PCB Editor, press **F8** to make sure the board matches the schematic.
2. Click the Fabrication Toolkit button on the top toolbar, or **Tools > External Plugins > Fabrication Toolkit**.
3. Set the options below and click **Generate**.

| Option | Setting |
|---|---|
| Archive name | Blank for the project name, or a pattern such as `${TITLE}_${REVISION}` |
| Additional layers | Blank |
| Plot all active layers | Off |
| Set User.1 as V-Cut layer | Off, unless you drew V-cut lines on `User.1` |
| Use User.2 for an alternative Edge-Cut layer | Off |
| Apply automatic translations | **On**. This corrects most rotation differences between KiCad and JLCPCB |
| Apply automatic fill for all zones | **On** |
| Exclude DNP components from BOM | **On** |
| Generate Backups | On, if you want a copy of the project saved alongside the outputs |

The plugin saves these choices in `fabrication-toolkit-options.json` in the project folder, so you only set them once.

### What it produces

A folder named `production` appears next to the project, containing:

| File | Upload it to |
|---|---|
| The `.zip` of Gerbers and drill files | The **Add gerber file** button on the order page |
| `bom.csv` | The **BOM** slot in the assembly step |
| `positions.csv` | The **CPL** slot in the assembly step |
| A designators list and an IPC netlist | Nothing. They are for reference |

`bom.csv` and `positions.csv` are already in JLCPCB's column format, so they need no editing.

## Export with KiCad alone

These settings follow JLCPCB's own guide for KiCad. All of it is in the PCB Editor unless noted.

### Gerbers

1. Click **File > Fabrication Outputs > Gerbers (.gbr)**.
2. Set **Output directory** to a new folder, for example `gerbers`. Use a plain name with no spaces.
3. Tick these layers:

   | Board | Layers |
   |---|---|
   | Two-layer | `F.Cu`, `B.Cu`, `F.Silkscreen`, `B.Silkscreen`, `F.Mask`, `B.Mask`, `Edge.Cuts` |
   | Four-layer | All of the above, plus `In1.Cu` and `In2.Cu` |
   | Adding a stencil | Also `F.Paste`, and `B.Paste` if there are parts on the bottom |

4. Set the options:

   | Option | Setting |
   |---|---|
   | Plot format | Gerber |
   | Check zone fills before plotting | On |
   | Drill marks | None |
   | Scaling | 1:1 |
   | Plot mode | Filled |
   | Use Protel filename extensions | On |
   | Use extended X2 format | On |
   | Include netlist attributes | On |
   | Coordinate format | 4.6, unit mm |

5. Click **Plot**. The message panel should show no errors or warnings.

### Drill files

1. In the same dialog, click **Generate Drill Files**.
2. Set the options and click **Generate**:

   | Option | Setting |
   |---|---|
   | Output folder | The same folder as the Gerbers |
   | Format | Excellon |
   | PTH and NPTH in single file | Off. KiCad writes separate `-PTH.drl` and `-NPTH.drl` files |
   | Use alternate drill mode for oval holes | On |
   | Origin | Absolute |
   | Units | Millimeters |
   | Zeros format | Decimal format |

3. Zip the whole folder: every Gerber file and both `.drl` files in one `.zip`, with plain English file names.

### BOM, for assembly

1. In the Schematic Editor, click **Tools > Edit Symbol Fields Table** and open the **Export** tab.
2. Set **Format presets** to **CSV** with a comma delimiter, choose a file name, and click **Export**.
3. Open the file in a spreadsheet and rename the column headers:

   | KiCad header | Rename to |
   |---|---|
   | Reference | `Designator` |
   | Value | `Comment` |
   | Footprint | `Footprint` (unchanged) |
   | LCSC Part # | `LCSC Part #` (unchanged) |

### Position file, for assembly

1. In the PCB Editor, click **File > Fabrication Outputs > Component Placement (.pos)**.
2. Set **Format** to CSV, **Units** to millimeters, and choose a single file for both sides. Click **Generate Position File**.
3. Rename the column headers:

   | KiCad header | Rename to |
   |---|---|
   | Ref | `Designator` |
   | PosX | `Mid X` |
   | PosY | `Mid Y` |
   | Rot | `Rotation` |
   | Side | `Layer` |

Rotations from this route are not corrected for JLCPCB. Expect to fix more of them in the placement preview than with Fabrication Toolkit.

## Check the files

Look at the files before ordering, not just the KiCad board.

1. Open **Gerber Viewer** from the main KiCad window.
2. Load the `.zip`, or all the files in the output folder.
3. Turn the layers on one at a time and check:
   - **Copper:** tracks and zones are all there, and the zones are filled.
   - **Mask:** an opening over every pad.
   - **Silkscreen:** text is readable, and nothing sits on pads.
   - **Edge.Cuts:** one clean outline.
   - **Drill files:** a hole in every through-hole pad, via, and mounting hole.
   - **Four-layer boards:** both inner layers are present.

JLCPCB's order page also shows a viewer after you upload, which is the last check before paying.

## Order bare boards

1. On [jlcpcb.com](https://jlcpcb.com), click **Order now**, then **Add gerber file**, and upload the `.zip`.
2. Wait for it to process. Check that the detected **layer count** and **dimensions** match your board.
3. Review the options. For a first board, these are sensible:

   | Option | Setting |
   |---|---|
   | Base material | FR-4 |
   | Layers | As detected. 2 or 4 |
   | PCB quantity | The minimum offered, usually 5 |
   | PCB thickness | 1.6 mm |
   | PCB color | Green. Usually the cheapest and fastest |
   | Surface finish | HASL lead-free. Choose ENIG for very fine-pitch parts |
   | Outer copper weight | 1 oz. Use 2 oz for high-current boards |
   | Via covering | Tented, the default |
   | Mark on PCB | No mark, unless you want a serial number or barcode |

4. Leave the remaining options at their defaults.
5. Open the viewer on the order page and check the board once more.
6. Add it to the cart and pay. JLCPCB's engineers review the files, and email you if there's a problem.

JLCPCB no longer prints an order number on boards by default. You don't need to place any `JLCJLCJLCJLC` marker text.

## Order assembly

Do steps 1 to 4 of [Order bare boards](#order-bare-boards) first, then:

1. Turn on **PCB Assembly** on the same page.
2. Set the assembly options:

   | Option | Setting |
   |---|---|
   | PCBA type | Economic for simple surface-mount boards. Standard for through-hole parts, both sides, or special parts |
   | Assembly side | Top, if all parts JLCPCB places are on the top |
   | PCBA quantity | How many of the boards to assemble |
   | Tooling holes | Added by JLCPCB |
   | Confirm parts placement | Yes, so an engineer checks rotations before building |

3. Click **Next**, then upload `bom.csv` to the BOM slot and `positions.csv` to the CPL slot. Click **Process BOM & CPL**.
4. Review the part matches. Every row should show the part you intended, in stock. For an unmatched row, which is usually one with no `LCSC Part #`, click **Search** and choose a part, or mark it not to be placed.
5. Review the placement preview. Check each chip, diode, LED, electrolytic capacitor, and connector: pin 1 and polarity must match the board's markings. If a part is turned or shifted, see [Fix rotated or shifted parts](#fix-rotated-or-shifted-parts).
6. Confirm the quote, then add it to the cart.

## Fix rotated or shifted parts

KiCad and JLCPCB don't always agree on a part's zero rotation, so some parts appear turned in JLCPCB's placement preview. Fabrication Toolkit's automatic translations fix most of them. For the rest, add a field to the part's symbol and regenerate the files:

| Field name | Value | Effect |
|---|---|---|
| `FT Rotation Offset` | Degrees, such as `90`, `180`, or `-90` | Turns the part. Positive is counter-clockwise |
| `FT Position Offset` | `x,y` in mm, such as `0,-1.5` | Shifts the part |
| `FT Layer Override` | `top` or `bottom` | Forces which side it's listed on |

Then:

1. Press **F8** in the PCB Editor, so the board picks up the new field.
2. Run Fabrication Toolkit again.
3. Upload the new `positions.csv` and check the preview.

Because the field lives on the symbol, the correction is kept for every future order of this board.

## Record what you ordered

After ordering, mark the exact version in the board's repository, so you can always rebuild or compare against it:

```bash
git add -A
git commit -m "Files for JLCPCB order, revision A"
git tag rev-A
git push
git push --tags
```

Keep the `production` folder in the commit, so the files JLCPCB received are saved alongside the design.

## Common problems

| Problem | Cause and fix |
|---|---|
| The order is held for a missing outline | `Edge.Cuts` was not exported, or is not closed |
| Ground plane missing in the viewer | Zones weren't filled before export. Press **B**, or turn on automatic zone fill in Fabrication Toolkit |
| Wrong layer count detected | Inner layers weren't exported on a four-layer board |
| Silkscreen text missing or broken | Text is smaller than 1.0 mm, or thinner than 0.15 mm |
| A BOM row won't match | Its `LCSC Part #` is empty, misspelled, or out of stock |
| A part looks turned in the preview | Add `FT Rotation Offset` to its symbol and regenerate |
| A hand-soldered part appears in the BOM | Exclude it from the BOM and position files, or mark it Do not populate |
| An unexpected extra cost | Extended parts, a non-green colour, ENIG, or assembly on both sides |

## Checklist before you pay

- [ ] DRC is clean with zones refilled, and the project is committed.
- [ ] The Gerbers have been checked in a viewer, including drill holes and the outline.
- [ ] The detected layer count and board size are right.
- [ ] Thickness, colour, finish, and copper weight are what you intended.
- [ ] For assembly: every placed part has a matching, in-stock LCSC part.
- [ ] For assembly: hand-soldered parts are left out of the BOM and position files.
- [ ] For assembly: the placement preview shows every polarized part the right way round.
- [ ] After ordering: the commit is tagged with the revision.
