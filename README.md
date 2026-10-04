# Creality K1 with Klipper and Cura 5.13

Printer definition, nozzle variants and print profiles for the **Creality K1** (base model, 220 × 220 × 250 mm) in **UltiMaker Cura 5.13**, plus the Klipper changes and the lessons learned while setting it up.

Cura 5.13 ships a definition for the K1 Max only, not for the plain K1. Bending an Ender profile into shape instead leads to trouble with the printable area (see [Pitfalls](#pitfalls)). This repository fills that gap.

## Contents

| Path | Purpose |
|---|---|
| `definitions/creality_k1_klipper.def.json` | Printer definition "Creality K1 (Klipper)" |
| `variants/creality_k1_klipper_0.4.inst.cfg` | Nozzle variant 0.4 mm (also 0.6 and 0.8 mm) |
| `profiles/K1_PLA_0.20.curaprofile` | Print profile PLA |
| `profiles/K1_PETG_0.20.curaprofile` | Print profile PETG |
| `profiles/K1_ABS_0.20.curaprofile` | Print profile ABS |
| `profiles/K1_ASA_0.20.curaprofile` | Print profile ASA |
| `profiles/K1_TPU_0.20.curaprofile` | Print profile TPU |
| `klipper/printer_cfg_changes.cfg` | Blocks for `printer.cfg` |

## Requirements

- Creality K1 with root access and the [Creality Helper Script](https://github.com/Guilouz/Creality-Helper-Script) installed (tested with version 6.3.0)
- Moonraker and Mainsail or Fluidd
- Cura 5.13 on Windows or Linux
- For uploading from Cura: the Moonraker plugin

## Preparing the printer

### Root and Helper Script

1. On the display, enable "Root account information" in the settings.
2. Log in via SSH: `ssh root@<printer-ip>`, password `creality_2023`. Change the password afterwards with `passwd`.
3. Install and start the Helper Script:

   ```
   git clone --depth 1 https://github.com/Guilouz/Creality-Helper-Script.git /usr/data/helper-script
   sh /usr/data/helper-script/helper.sh
   ```

### Installed components

| Component | Purpose |
|---|---|
| Moonraker and Nginx | API and web server |
| Mainsail | Web interface |
| Entware | Package manager |
| Klipper Gcode Shell Command | Required by other macros |
| KAMP | Adaptive mesh and purge line |
| Useful Macros | Helper macros |
| Save Z-Offset Macros | Z offset survives a restart |
| Screws Tilt Adjust | Four-point measurement of bed tilt |
| Guppy Screen | Klipper UI on the touch display |

Notes:

- **Guppy Screen and Improved Shapers are mutually exclusive.** Guppy Screen brings its own shaper calibration. "Improved Shapers Calibrations" has to be removed first.
- **Creality services:** They can be disabled during the Guppy Screen installation. This frees resources, but Creality Cloud and Creality Print stop working.
- **Mainsail on port 80:** In the "Customize" menu via "Remove Creality Web Interface". If the browser still shows the old page afterwards, clear the cache (Ctrl + Shift + R).
- **Moonraker update:** Treat the update offered in Mainsail's update manager with caution. The Helper Script ships its own Moonraker package built for the K1.

### Changes to printer.cfg

The blocks are in `klipper/printer_cfg_changes.cfg`.

- **9×9 mesh:** `probe_count: 9,9` requires `algorithm: bicubic`. The default "lagrange" allows at most 6 points per axis, otherwise Klipper refuses to start.
- **Purge line and mesh before each print:** KAMP's start macro reads the switches `ADAPTIVE_PURGE_LINE` and `ADAPTIVE_BED_MESH`. The KAMP files are read-only, but the values can be overridden in `printer.cfg`.
- **No indentation:** Section names must start in the first column.
- **Restart:** Use `FIRMWARE_RESTART` after changes. A plain `RESTART` occasionally ends with "Can not update MCU config as it is shutdown" or a connection error on the K1.

Run and save the mesh once:

```
G28
BED_MESH_CALIBRATE
SAVE_CONFIG
```

Heat the bed to print temperature first and let it soak for 5 to 10 minutes, with the build plate in place.

## Levelling the bed

On my printer the bed was initially tilted by about 1 mm. After the correction the mesh range is 0.16 mm. What helped:

1. **Check the retaining nuts of the heated bed.** Mine were not properly tight. Loose nuts make every measurement unreliable and can trigger the "Z-axis motor step loss" error during meshing.
2. **Measure with `SCREWS_TILT_CALCULATE`** (after `G28`). The four values should be close together, the absolute number does not matter. Always measure in the same state, either always cold or always warm.
3. **Washers at the mounting points of the heated bed.** The difference between the measured values equals the washer thickness needed. The washers must only sit at the screw location, because the load cells are mounted there. This was the simplest and most precise method.
4. **Adjust the Z lead screws only as a last resort.** The three lead screws (two at the front, one at the rear) are linked by a belt under the base. Each pulley has two grub screws, one of them on a flat of the shaft. The screws are hard to reach, and the power supply and mainboard sit right next to them. Unplug the printer first and put a cloth underneath.
5. **Duplo as a gauge.** A tower of ten Duplo bricks measures 196.5 mm and fits between the base plate and the nut holder. The same tower at all three lead screws shows which one is too high or too low.
6. **Belt tensioner.** After working on the Z belt, loosen the two screws in the slot of the tensioner by one turn, rotate a lead screw by hand and tighten them again. The spring sets the tension.

Do not loosen the collar at the top of the lead screws. It only holds the lead screw in height.

## Setting up Cura

### Copying the files

Close Cura and copy the folders `definitions` and `variants` into the Cura folder:

| System | Target folder |
|---|---|
| Windows | `%APPDATA%\cura\5.13\` |
| Linux (Flatpak) | `~/.var/app/com.ultimaker.cura/data/cura/5.13/` |
| Linux (AppImage) | `~/.local/share/cura/5.13/` |

On Windows this is the folder under `Roaming`, not the cache under `Local`.

### Adding the printer

1. Start Cura.
2. **Add printer → Non-networked printer → Creality3D → Creality K1 (Klipper)**.
3. The header bar must show "0.4mm Nozzle" and a material.

### Importing the profiles

**Preferences → Profiles → Import**, then pick the files you want from `profiles`.

### Uploading via Moonraker

- Address: `http://<printer-ip>:7125/`
- Output format: G-code, not UFP
- File names without umlauts, spaces or special characters

## What the definition sets

- Build volume 220 × 220 × 250 mm, origin at the front left
- **Disallowed area:** the rear 8 mm of the bed
- Skirt at 3 mm distance instead of 10 mm
- Start G-code:

  ```
  START_PRINT EXTRUDER_TEMP={material_print_temperature_layer_0} BED_TEMP={material_bed_temperature_layer_0}
  G92 E0
  ```

- End G-code:

  ```
  END_PRINT
  ```

Homing, levelling or purge lines of your own do not belong in the start G-code. `START_PRINT` takes care of that.

## Print profiles

All profiles are for the 0.4 mm nozzle and 0.2 mm layer height.

| | PLA | PETG | ABS | ASA | TPU |
|---|---|---|---|---|---|
| Nozzle | 220 °C | 245 °C | 260 °C | 260 °C | 230 °C |
| Bed | 60 °C | 75 °C | 100 °C | 100 °C | 50 °C |
| Fan | 100 % | 40 % | 20 % | 30 % | 100 % |
| Outer wall | 200 mm/s | 120 mm/s | 180 mm/s | 180 mm/s | 30 mm/s |
| Inner wall | 300 mm/s | 150 mm/s | 220 mm/s | 220 mm/s | 40 mm/s |
| Infill | 270 mm/s | 150 mm/s | 220 mm/s | 220 mm/s | 40 mm/s |
| First layer | 60 mm/s | 40 mm/s | 50 mm/s | 50 mm/s | 25 mm/s |
| Travel | 500 mm/s | 400 mm/s | 500 mm/s | 500 mm/s | 250 mm/s |
| Acceleration | 10000 | 5000 | 8000 | 8000 | 2000 |
| Retraction | 0.8 mm | 0.8 mm | 0.8 mm | 0.8 mm | 0.6 mm |
| Z hop | 0.4 mm | 0.4 mm | 0.4 mm | 0.4 mm | off |
| Adhesion | Skirt | Skirt | Brim 5 mm | Brim 5 mm | Skirt |

Common to all: 2 walls, 5 top and 3 bottom layers, 15 % infill (grid), line width 0.42 mm outer and 0.45 mm inner, fan off on the first layer.

**Test status:** The PLA values come from the default profile "0.20mm Standard @Creality K1" of [OrcaSlicer](https://github.com/SoftFever/OrcaSlicer) and have been tested on the printer. PETG, ABS, ASA and TPU are cautious starting values and have not been tested yet. Please adjust temperature and flow to your own filament.

For ABS and ASA: keep the door and lid closed. The chamber fan regulates to 35 °C by default. The chamber should stay warmer for these materials.

## Calibration

In this order:

1. **Z offset.** During the first layer, lower it in Mainsail in steps of 0.025 mm until the lines fuse into a closed surface. Separate strands with grooves between them mean the nozzle is too high.
2. **Extruder.** Mark 120 mm on the filament, extrude 100 mm, measure. New `rotation_distance` = old value × measured length ÷ 100.
3. **Flow.** Print a single-wall cube and measure in the middle of the wall. Flow = configured line width ÷ measured wall thickness. Compare against the line width (0.42 mm), not the nozzle diameter.
4. **Pressure advance** per filament.

## Pitfalls

- **Offset in the middle of a print and loud rattling.** The cause was a skirt on the outermost edge of the bed (X/Y 0 to 220). Two stop screws sit at the rear edge of the build plate and stand higher than the plate. The nozzle catches on them, the head loses steps and later runs into the frame. The K1 has no endstop switches and does not notice. Remedy: the disallowed area of this definition and a tight skirt.
- **Square instead of curly brackets.** `[material_print_temperature_layer_0]` is the syntax of PrusaSlicer and OrcaSlicer. Cura only replaces `{...}`. With square brackets `START_PRINT` receives no temperatures and Cura prepends its own heating commands.
- **"Unable to open file" when starting a print.** Usually a file name with umlauts, spaces or special characters.
- **Imported profile is not shown.** Cura ties Creality profiles to a nozzle variant. Without the files in the `variants` folder Cura finds no profile. Remove a printer that was added without the variants and add it again.
- **Upload "completed" but no file.** Use port 7125 and the G-code format in the Moonraker plugin.
- **Mesh gone after a restart.** A mesh is only stored permanently after `SAVE_CONFIG`.
- **81 probe points before every print.** The adaptive mesh only probes fewer points if the slicer supplies object data. With `ADAPTIVE_BED_MESH` set to 0 the saved mesh is loaded instead.

## Sources and credits

- [Creality Helper Script](https://github.com/Guilouz/Creality-Helper-Script) by Guilouz
- [Klipper Adaptive Meshing and Purging (KAMP)](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)
- [Guppy Screen](https://github.com/ballaswag/guppyscreen)
- [OrcaSlicer](https://github.com/SoftFever/OrcaSlicer) for the base values of the PLA profile
- [UltiMaker Cura](https://github.com/Ultimaker/Cura), whose `creality_base` the definition builds on

## Disclaimer

Everything here is based on a single printer. Changes to firmware and mechanics are at your own risk. Only work under the base of the machine with the power cord unplugged.
