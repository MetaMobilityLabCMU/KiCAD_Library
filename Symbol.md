# Making a schematic symbol

How to draw a symbol for `my_symbols` when no download exists for the part. For importing a downloaded part, see the [README](README.md#add-a-part-to-the-library). For the matching footprint, see [Footprint.md](Footprint.md).

These instructions are written for KiCad 10.

- [Name and Number](#name-and-number)
- [The example part](#the-example-part)
- [Draw the symbol](#draw-the-symbol)
- [Electrical types](#electrical-types)
- [Parts with several ground or power pins](#parts-with-several-ground-or-power-pins)
- [Link the symbol to its footprint](#link-the-symbol-to-its-footprint)
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
- **The Number must match the footprint pad exactly.** `VIN` and `Vin` are different Numbers.
- **Names are free.** Three pins can all be named `GND`. A Name can also be left empty when the Number already says everything.

A Number does not have to be a digit. Text such as `GND1` or `VIN` is allowed, as long as the footprint pad carries the same text.

For a module or development board, there are two reasonable ways to choose Numbers. Pick one and use it on both the symbol and the footprint.

| Scheme | Example Numbers | Good for |
|---|---|---|
| Position around the edge | `1`, `2`, `3` ... | Any part. Matches how chips are numbered |
| The labels printed on the board | `0`, `1`, `A3`, `GND1`, `VIN` | Boards where people think in the printed labels. Repeated labels such as GND need a suffix to stay unique |

## The example part

The steps below use one made-up part so the values are concrete: a small plug-in module with 8 pins. [Footprint.md](Footprint.md#the-example-part) draws the footprint for the same part.

| Number | Name | Electrical type | Side of the symbol |
|---|---|---|---|
| `1` | `VIN` | Power input | Left |
| `2` | `GND` | Power input | Left |
| `3` | `D0` | Bidirectional | Left |
| `4` | `D1` | Bidirectional | Left |
| `5` | `D2` | Bidirectional | Right |
| `6` | `D3` | Bidirectional | Right |
| `7` | `GND` | Power input | Right |
| `8` | `3V3` | Power output | Right |

Pins `2` and `7` share the Name `GND`. That is allowed, because their Numbers differ.

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
4. Draw the body with the rectangle tool, centred on the origin (the crosshair at 0, 0).
5. Press **P** to add a pin. Fill in its Number, Name, and Electrical type, then click to place it. The end with the small circle is where a wire attaches, so it points away from the body.
6. Repeat for every pin on the part, including ones you don't plan to use.
7. Click **Edit > Pin Table** and read down the Number column. Look for blanks and repeats. The table is also the quickest place to edit many pins at once.
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

Use Power output, not plain Output, for a pin that supplies a voltage rail. Plain Output is for signals.

### Connectors and other passive parts

Some parts only carry current through. They don't drive a signal or need one driven into them to work. Every pin on these parts is **Passive**:

- Connectors and pin headers
- Resistors, capacitors, and inductors
- Fuses, switches, and jumpers
- Test points

This holds even when a connector carries power or a specific signal:

- **Power connectors:** a battery connector's `+` and `-` pins are Passive, not Power input or Power output. The same connector might bring power onto one board and pass it on from another, and Passive suits both.
- **Signal pins:** a connector carrying CAN, UART, or anything else is still Passive. The chip that actually drives the signal, such as a CAN transceiver, has the real types (Bidirectional for CANH and CANL). The rule checker checks the signal there.
- **Shared library symbols:** the same connector symbol may carry different signals in different projects. Passive works for all of them. Show what a pin carries with net labels in the schematic, such as `CAN1+` or `+BATT`, not in the symbol.

Wiring a Passive pin to any other type is fine in the rule checker.

Because a Passive pin isn't a power source, a supply that comes in through a connector, such as a battery, needs a `PWR_FLAG` symbol on that net. Without it, the rule checker reports "Input Power pin not driven by any Output Power pins".

#### The exception: modules with their own electronics

A plug-in module, such as a microcontroller board or a sensor breakout, can look like a connector. But its pins are the module's real inputs, outputs, and supplies, so give them real types, as for a chip.

| The part is... | Pin types |
|---|---|
| Just metal contacts, such as a connector or header | All Passive |
| A board with chips on it, such as a Teensy or a sensor breakout | Real types: Power input, Output, Bidirectional, and so on |

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

Some of KiCad's own symbols for large chips place identical pins exactly on top of each other, so one wire connects them all. With only a few pins, keep them separate, so you can see which pin is which.

## Link the symbol to its footprint

Two things join a symbol to a footprint.

| Link | Where it is set | What it must match |
|---|---|---|
| Which footprint to use | The symbol's **Footprint** field | The footprint's library and name, as `my_footprints:FOOTPRINT_NAME` |
| Which pin goes to which pad | Each pin's **Number** | The Number on the footprint pad, exactly |

Set the Footprint field in **File > Symbol Properties**:

```
my_footprints:FOOTPRINT_NAME
```

To check it, reopen Symbol Properties and click the library icon at the end of the Footprint cell. The footprint browser should open with the right footprint selected.

[Footprint.md](Footprint.md#link-the-footprint-to-the-symbol) covers the pad side, and how to test the link in a schematic.

## Best practices

- **Pins 100 mil (2.54 mm) apart and 100 mil long**, with 50 mil text. This matches KiCad's own libraries, so your symbol lines up with theirs.
- **Keep every pin on the grid.** A pin placed off the grid will not meet wires in a schematic.
- **Arrange pins so the schematic reads well.** For chips: inputs on the left, outputs on the right, power at the top, ground at the bottom. For development boards and modules, copying the physical pin layout is also fine.
- **Don't hide pins.** A hidden pin can connect to a net without anyone seeing it.
- **Don't stack pins on top of each other** unless you mean to. Stacked pins all connect to one wire.
- **Include every pin on the part**, even unused ones, so the symbol and footprint have the same count.
- **Check Numbers against the datasheet**, not against another person's symbol.
- **Give the symbol and its footprint the same name** where you can, so they are easy to pair up.

## Fixing "Duplicate pin" warnings

**Inspect > Check Symbol** reports a duplicate like this:

```
Duplicate pin 'GND1' at location (...) conflicts with pin 'VIN' at location (...)
```

The text in quotes is each pin's **Name**. The thing that is duplicated is the **Number**. Two pins with different Names can still collide if their Numbers are the same.

The usual cause is that the unique labels were typed into the Name field and the Number field was left empty. Every pin with an empty Number then counts as a duplicate of the others.

You can tell from the canvas which field a label is in. Names are drawn inside the body. Numbers are drawn above the pin line, outside the body. A pin with nothing above its line has no Number.

To fix it:

1. Click **Edit > Pin Table**.
2. If rows look merged together, untick the grouping checkbox at the bottom of the table, so each pin has its own row.
3. Find the rows where **Number** is empty or repeated, and give each one a unique value.
4. Change the **Name** back to the plain function, such as `GND`, if you like. Repeated Names are fine.
5. Click **OK** and run **Inspect > Check Symbol** again.

## Checklist before you commit

- [ ] Every pin has a Number, and no Number repeats.
- [ ] Each Number matches a pad on the footprint exactly, including upper and lower case.
- [ ] Each Number matches the datasheet.
- [ ] Electrical types are set, with one Power output per rail at most.
- [ ] Connectors and other parts that only carry current have every pin set to Passive.
- [ ] All pins are on the grid and none are hidden.
- [ ] The Footprint field is set to `my_footprints:FOOTPRINT_NAME`.
- [ ] **Inspect > Check Symbol** reports nothing.
