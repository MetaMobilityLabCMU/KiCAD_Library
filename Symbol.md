# Making a schematic symbol

How to draw a symbol for `my_symbols` when no download exists for the part, and the rules that keep it correct. For importing a downloaded part, see the [README](README.md#add-a-part-to-the-library).

These instructions are written for KiCad 10.

- [Name and Number](#name-and-number)
- [Draw the symbol](#draw-the-symbol)
- [Electrical types](#electrical-types)
- [Parts with several ground or power pins](#parts-with-several-ground-or-power-pins)
- [Best practices](#best-practices)
- [Fixing "Duplicate pin" warnings](#fixing-duplicate-pin-warnings)
- [Checklist before you commit](#checklist-before-you-commit)

## Name and Number

Every pin has two text fields, and they do different jobs.

| Field | What it is | Must be unique? | Shown |
|---|---|---|---|
| **Number** | Which physical pad on the footprint this pin connects to | Yes, and never empty | Above the pin line, outside the body |
| **Name** | What the pin does, for people reading the schematic | No, names can repeat | Inside the body |

KiCad connects a symbol pin to the footprint pad that has the same Number. That match is the only thing that makes the board correct, so:

- **Every pin needs a Number.** A pin with an empty Number connects to nothing on the board.
- **No two pins can share a Number.** Two empty Numbers count as the same Number.
- **The Number must match the footprint pad exactly**, character for character.
- **Names are free.** Three pins can all be named `GND`. A Name can also be left empty when the Number already says everything.

A Number does not have to be a digit. Text such as `GND1` or `VIN` is allowed, as long as the footprint pad carries the same text.

## Draw the symbol

1. Open the **Symbol Editor** and click `my_symbols` in the library list.
2. Click **File > New Symbol**. Enter the symbol name and its reference letter:

   | Letter | Part type |
   |---|---|
   | `U` | Chips and modules |
   | `J` | Connectors |
   | `R`, `C`, `L` | Resistors, capacitors, inductors |
   | `D`, `Q` | Diodes, transistors |
   | `SW` | Switches |

3. Set the grid to **50 mil (1.27 mm)** and leave it there while you work.
4. Draw the body with the rectangle tool, centred on the origin (the crosshair at 0,0).
5. Press **P** to add a pin. Fill in its Name, Number, and Electrical type, then click to place it. The end with the small circle is where a wire attaches, so it points away from the body.
6. Repeat for every pin on the part, including ones you don't plan to use.
7. Click **Edit > Pin Table** and read down the Number column. Look for blanks and repeats.
8. Click **File > Symbol Properties** and fill in the fields:

   | Field | Set to |
   |---|---|
   | Value | The part name |
   | Footprint | `my_footprints:FOOTPRINT_NAME` |
   | Datasheet | A link to the datasheet |
   | Description | One line saying what the part is |
   | `LCSC Part #` | The LCSC number, if JLCPCB will assemble the part |

9. Click **Inspect > Check Symbol** and fix anything it reports.
10. Press **Ctrl+S**.

## Electrical types

The type tells KiCad's rule checker what a pin is allowed to connect to, so it can catch mistakes such as two outputs wired together. Set it to what the pin really does.

| Type | Use for |
|---|---|
| Power input | Supply and ground pins that need power fed to them |
| Power output | A pin that supplies a rail, such as a regulator output |
| Input | Pins that only receive a signal |
| Output | Pins that only drive a signal |
| Bidirectional | General-purpose I/O |
| Passive | Connectors, resistors, and anything with no direction |
| No connect | Pins the datasheet says to leave unconnected |

A rail should have exactly one Power output on it. If a part has two pins that both supply the same rail, make one Power output and the other Passive. Two Power outputs wired together are reported as a conflict.

## Parts with several ground or power pins

Keep them as separate pins with the same Name and different Numbers.

| Name | Number |
|---|---|
| `GND` | `GND1` |
| `GND` | `GND2` |
| `GND` | `GND3` |

Three things to know:

- **A shared Name does not connect pins.** In the schematic, wire each one to a `GND` power symbol. That puts them on one net.
- **KiCad expects copper to every pad.** It does not know the pins are already joined inside the part. A ground pour on the board handles this.
- **You may see "Input Power pin not driven"** on a net that has only Power input pins. Place one `PWR_FLAG` symbol on that net to clear it.

## Best practices

- **Pins 100 mil (2.54 mm) apart and 100 mil long**, with 50 mil text. This matches KiCad's own libraries, so your symbol lines up with theirs.
- **Keep every pin on the grid.** A pin placed off the grid will not meet wires in a schematic.
- **Arrange pins so the schematic reads well.** For chips: inputs on the left, outputs on the right, power at the top, ground at the bottom. For development boards and modules, copying the physical pin layout is also fine.
- **Don't hide pins.** A hidden pin can connect to a net without anyone seeing it.
- **Don't stack pins on top of each other** unless you mean to. Stacked pins all connect to one wire, which hides which pin is which.
- **Include every pin on the part**, even unused ones, so the symbol and footprint have the same count.
- **Check Numbers against the datasheet**, not against another person's symbol.

## Fixing "Duplicate pin" warnings

**Inspect > Check Symbol** reports a duplicate like this:

```
Duplicate pin 'GND1' at location (...) conflicts with pin 'VIN' at location (...)
```

The text in quotes is each pin's **Name**. The thing that is duplicated is the **Number**. Two pins with different Names can still collide if their Numbers are the same.

The usual cause is that the unique labels were typed into the Name field and the Number field was left empty. Every pin with an empty Number then counts as a duplicate of the others.

To fix it:

1. Click **Edit > Pin Table**.
2. If rows look merged together, untick the grouping checkbox at the bottom of the table, so each pin has its own row.
3. Find the rows where **Number** is empty or repeated, and give each one a unique value.
4. Change the **Name** back to the plain function, such as `GND`, if you like. Repeated Names are fine.
5. Click **OK** and run **Inspect > Check Symbol** again.

## Checklist before you commit

- [ ] Every pin has a Number, and no Number repeats.
- [ ] Each Number matches a pad on the footprint.
- [ ] Each Number matches the datasheet.
- [ ] Electrical types are set, with one Power output per rail at most.
- [ ] All pins are on the grid and none are hidden.
- [ ] The Footprint field is set to `my_footprints:FOOTPRINT_NAME`.
- [ ] **Inspect > Check Symbol** reports nothing.
