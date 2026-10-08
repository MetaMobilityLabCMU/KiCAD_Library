# KiCAD_Library

MEMO KiCAD custom parts: a shared library of schematic symbols, PCB footprints, and 3D models for KiCad. Clone it once, point KiCad at it, and every project on your computer can use the same parts.

These instructions are written for KiCad 10. Menu names may differ slightly in older versions.

- [What's in here](#whats-in-here)
- [Guides](#guides)
- [How a part fits together](#how-a-part-fits-together)
- [Set up the library in KiCad](#set-up-the-library-in-kicad) (one time)
- [Add a part to the library](#add-a-part-to-the-library)
- [Conventions](#conventions)
- [Troubleshooting](#troubleshooting)

## What's in here

```
KiCAD_Library/
├── README.md                     this page: setup, and importing downloaded parts
├── Symbol.md                     guide to drawing a schematic symbol
├── Footprint.md                  guide to drawing a PCB footprint
├── custom/                       parts we downloaded or drew, and checked by hand
│   ├── my_symbols.kicad_sym      schematic symbols (all in this one file)
│   ├── my_footprints.pretty/     PCB footprints (one .kicad_mod file per part)
│   └── 3dmodels/                 3D models (.step) for those footprints
└── lcsc/                         parts auto-converted from LCSC / EasyEDA
    ├── lcsc.kicad_sym
    ├── lcsc.pretty/
    └── lcsc.3dshapes/
```

Inside KiCad these appear as the symbol libraries `my_symbols` and `lcsc`, and the footprint libraries `my_footprints` and `lcsc`.

Parts in `lcsc` were converted by a script (`easyeda2kicad`) and have not been checked by hand. Compare the footprint against the datasheet before you order a board that uses one.

## Guides

| Guide | Read it when |
|---|---|
| This README | You are setting up KiCad, or adding a part you downloaded |
| [Symbol.md](Symbol.md) | You need to draw a schematic symbol yourself |
| [Footprint.md](Footprint.md) | You need to draw a PCB footprint yourself |

## How a part fits together

A complete part is three pieces:

| Piece | What it is | File type | Lives in |
|---|---|---|---|
| Symbol | The part as drawn in a schematic | `.kicad_sym` | `my_symbols` |
| Footprint | The pads, holes, and outlines on the board | `.kicad_mod` | `my_footprints` |
| 3D model | The part's shape, for the 3D viewer | `.step` | `custom/3dmodels/` |

KiCad does not join them automatically. Three links do, and each one is plain text that has to match:

| Link | Where it is set | What it must match |
|---|---|---|
| Symbol to footprint | The symbol's **Footprint** field | The footprint's library and name, as `my_footprints:FOOTPRINT_NAME` |
| Pin to pad | Each symbol pin's **Number** | The **Number** on a footprint pad, exactly |
| Footprint to 3D model | The footprint's **3D Models** tab | The model file's path, starting with `${MY_LIBS}` |

Most problems with a part come down to one of these three not matching.

## Set up the library in KiCad

You do this once per computer. It takes about five minutes.

### 1. Clone the repository

Clone it anywhere permanent, with GitHub Desktop or from a terminal:

```bash
git clone https://github.com/MetaMobilityLabCMU/KiCAD_Library.git
```

Note the full path of the folder it creates, for example `C:/Users/yourname/Documents/Github/KiCAD_Library`.

### 2. Create the `MY_LIBS` path variable

Every path in this library starts with `${MY_LIBS}` instead of a real folder, so the same files work on everyone's computer. You tell KiCad what `MY_LIBS` means on yours.

1. In the main KiCad window (the project manager, not one of the editors), click **Preferences > Configure Paths**.
2. Under the **Environment Variables** table, click **+**.
3. Set **Name** to:

   ```
   MY_LIBS
   ```

4. Set **Path** to the folder you cloned, the one that contains `custom` and `lcsc`.
5. Click **OK**.

### 3. Add the symbol libraries

1. Click **Preferences > Manage Symbol Libraries** and open the **Global Libraries** tab.
2. Click the folder icon below the table, browse to `KiCAD_Library/custom`, select `my_symbols.kicad_sym`, and click **Open**.
3. Click the folder icon again, browse to `KiCAD_Library/lcsc`, and select `lcsc.kicad_sym`.
4. Check that the **Nickname** column of the two new rows reads `my_symbols` and `lcsc`.
5. Replace the **Library Path** of each row with the path below, then click **OK**.

Library path for `my_symbols`:

```
${MY_LIBS}/custom/my_symbols.kicad_sym
```

Library path for `lcsc`:

```
${MY_LIBS}/lcsc/lcsc.kicad_sym
```

### 4. Add the footprint libraries

1. Click **Preferences > Manage Footprint Libraries** and open the **Global Libraries** tab.
2. Click the folder icon below the table, browse to `KiCAD_Library/custom`, click the `my_footprints.pretty` folder once, and click **Select Folder**.
3. Do the same for `KiCAD_Library/lcsc/lcsc.pretty`.
4. Check that the **Nickname** column of the two new rows reads `my_footprints` and `lcsc`.
5. Replace the **Library Path** of each row with the path below, then click **OK**.

Library path for `my_footprints`:

```
${MY_LIBS}/custom/my_footprints.pretty
```

Library path for `lcsc`:

```
${MY_LIBS}/lcsc/lcsc.pretty
```

The nicknames have to match these exactly. Symbols point at their footprints by nickname, so a different nickname breaks the link.

### 5. Check that it worked

1. Open the **Symbol Editor** and expand `my_symbols` in the library list. Its parts should be listed.
2. Open the **Footprint Editor**, expand `my_footprints`, and double-click any footprint.
3. Click **View > 3D Viewer**. You should see the part's 3D model sitting on its pads.

If something is missing:

- **A library isn't in the list:** close KiCad completely and reopen it.
- **The pads show but there is no 3D model:** the `MY_LIBS` path is wrong. It must be the top `KiCAD_Library` folder, not `custom` or `lcsc`.

### Getting updates

Pull the repository (`git pull`, or **Fetch/Pull** in GitHub Desktop). New parts show up in the editors, after a restart of the editor if needed.

Designs you already made do not change when you pull. KiCad copies each symbol and footprint into the project when you place it. To bring an existing design up to date, use **Tools > Update Symbols from Library** in the Schematic Editor and **Tools > Update Footprints from Library** in the PCB Editor.

## Add a part to the library

There are three routes, in order of preference:

| Situation | What to do |
|---|---|
| KiCad's built-in libraries have the part | Use it from there. Nothing is added to this library |
| A parts website has a KiCad download | Import it, with the steps below |
| No download exists | Draw the symbol with [Symbol.md](Symbol.md) and the footprint with [Footprint.md](Footprint.md), then do steps 4 to 7 below |

Resistors, capacitors, and common chip packages are already in the built-in libraries. Add a part here only when they don't have it.

In the steps below, `PART_NAME` and `FOOTPRINT_NAME` are placeholders. Replace them with the names of the files you downloaded.

Pull the repository before you start, so you are adding to the latest version.

### 1. Download the part

1. Search for the exact manufacturer part number on a parts site. [SnapMagic](https://www.snapmagic.com), [Component Search Engine](https://componentsearchengine.com), and [Ultra Librarian](https://www.ultralibrarian.com) all offer free KiCad downloads with a free account.
2. Download the symbol and footprint in **KiCad** format. If asked for a KiCad version, pick the newest offered.
3. Download the 3D model too, either included in the same zip or from a separate **Download 3D Model** button.
4. Extract the zip. You should have three files:

| File | Named like |
|---|---|
| Symbol | `PART_NAME.kicad_sym` |
| Footprint | `FOOTPRINT_NAME.kicad_mod` |
| 3D model | `PART_NAME.step` |

Use the exact part number, not a close relative. Variants in the same family often have a different pin count or hole pattern.

### 2. Import the symbol

1. Open the **Symbol Editor**.
2. In the library list on the left, click `my_symbols` once so it is highlighted. The import goes into whichever library is selected.
3. Click **File > Import > Symbol**, select the `.kicad_sym` file, and click **Open**.
4. Confirm the symbol appears on the canvas and is listed under `my_symbols`.
5. Press **Ctrl+S**.

### 3. Import the footprint

1. Open the **Footprint Editor**.
2. Click **File > Import > Footprint**, select the `.kicad_mod` file, and click **Open**. The pads appear on the canvas, but the footprint is not in a library yet.
3. Click **File > Save As**, choose `my_footprints` as the library, keep the suggested name, and click **OK**.
4. Confirm the footprint is listed under `my_footprints`.

### 4. Attach the 3D model

1. Copy the `.step` file into `KiCAD_Library/custom/3dmodels/`.
2. In the Footprint Editor, with the footprint open, click **File > Footprint Properties** and open the **3D Models** tab.
3. Click **+** to add a row, and enter this path, replacing `PART_NAME.step` with your file's name:

   ```
   ${MY_LIBS}/custom/3dmodels/PART_NAME.step
   ```

4. Look at the preview. If it is empty, the file name or path does not match.
5. If the model is tilted or off the pads, fix it with the **Rotation** and **Offset** boxes on the same tab:
   - **Rotation X or Y**, `90` or `-90`: stands the part the right way up.
   - **Rotation Z**: turns it to face the right way on the board.
   - **Offset X, Y, Z** (mm): slides the pins into the holes and sets the body on the board surface.
6. Click **OK**, then press **Ctrl+S**.

Never browse to the file and leave a path that starts with `C:/Users/...`. That path only exists on your computer, so the model would be missing for everyone else.

### 5. Link the symbol to the footprint

KiCad does not link them automatically. The link is a text field on the symbol named **Footprint**, written as `library:footprint`. When you place the symbol in a schematic and update the PCB, KiCad reads that field to pick the footprint. It then connects each symbol pin to the pad that has the same Number.

1. In the **Symbol Editor**, open the symbol and click **File > Symbol Properties**.
2. Set the **Footprint** field to the line below, replacing `FOOTPRINT_NAME` with the footprint's name as it appears under `my_footprints` in the Footprint Editor:

   ```
   my_footprints:FOOTPRINT_NAME
   ```

3. Click **+** to add a field, and name it exactly:

   ```
   LCSC Part #
   ```

4. Set that field's value to the part's LCSC number, which is the letter `C` followed by digits. JLCPCB's assembly export reads this field.
5. Click **OK**, then press **Ctrl+S**.

To check the link, reopen Symbol Properties and click the library icon at the end of the Footprint cell. The footprint browser should open with the right footprint selected.

### 6. Check the part against the datasheet

Downloaded parts are usually right, but not always. Before anyone orders a board with it, confirm:

- **Pin numbers:** every symbol pin Number has a footprint pad with exactly the same Number, and both match the datasheet. On power connectors, confirm which pad is positive and which is negative.
- **Pad positions:** hole and pad spacing match the recommended layout drawing in the datasheet.
- **3D model:** the body sits on the board and the pins line up with the pads.

### 7. Share it

Commit and push so everyone else gets the part on their next pull:

```bash
git add -A
git commit -m "Add PART_NAME (LCSC number)"
git push
```

A normal part changes three things: `custom/my_symbols.kicad_sym`, one new `.kicad_mod` file in `custom/my_footprints.pretty/`, and one new `.step` file in `custom/3dmodels/`.

## Conventions

- **One part per commit**, with the part number and LCSC number in the message.
- **Pull before you add a part, push right after.** All symbols live in one file, so two people adding symbols at the same time causes a merge conflict.
- **Always use `${MY_LIBS}` in paths**, never a path from your own computer.
- **Don't rename the libraries.** The nicknames `my_symbols`, `my_footprints`, and `lcsc` are stored inside the parts.
- **Don't edit KiCad's built-in libraries.** Changes there are lost when KiCad updates. To modify a built-in part, save a copy into `my_symbols` or `my_footprints` and edit the copy.
- **Fill in `LCSC Part #`** on every symbol that JLCPCB will assemble.
- **Give a symbol and its footprint the same name** where you can, so they are easy to pair up.
- **Treat pin and pad Numbers as exact text.** `VIN` and `Vin` do not match.
- **Run the checker before committing:** **Inspect > Check Symbol** in the Symbol Editor and **Inspect > Footprint Checker** in the Footprint Editor.
- **Check the license before committing downloaded models** if this repository is ever made public. Some vendors do not allow their symbol, footprint, or 3D files to be redistributed.

## Troubleshooting

| Problem | Cause and fix |
|---|---|
| A library is not in the list | Close KiCad completely and reopen it. If it is still missing, add it under **Preferences > Manage Symbol Libraries** or **Manage Footprint Libraries** |
| KiCad did not ask Global or Project when creating a library | It only asks when a project is open. With no project open, the library is added as Global |
| Pads show but there is no 3D model | The `MY_LIBS` path is wrong, or the model path in the footprint does not start with `${MY_LIBS}` |
| The 3D model is tilted or off the pads | Set Rotation and Offset on the footprint's **3D Models** tab. See [step 4](#4-attach-the-3d-model) |
| "Duplicate pin" warnings on a symbol | Pin Numbers are empty or repeated. See [Symbol.md](Symbol.md#fixing-duplicate-pin-warnings) |
| A pad has no net after updating the board | Its Number does not match any symbol pin Number. See [Footprint.md](Footprint.md#link-the-footprint-to-the-symbol) |
| An old design did not change after a pull | Designs keep their own copy of each part. See [Getting updates](#getting-updates) |
| Git warns that LF will be replaced by CRLF | Harmless. It is a line-ending difference between Windows and the repository |
| Git reports a conflict in `my_symbols.kicad_sym` | Two people added symbols at the same time. Pull before adding a part, and push right after |
