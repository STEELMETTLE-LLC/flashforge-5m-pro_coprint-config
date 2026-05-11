# Flashforge Adventurer 5M Pro — ChromaSet Klipper Config

Public Klipper configuration pack for the Flashforge Adventurer 5M Pro with ChromaSet multi-filament feeder add-on. Built and tested by STEELMETTLE LLC.

---

## Known Hardware Issues with the ChromaSet Product

### 1. Feeder Motor Grip Failure
The ChromaSet feeder motors have insufficient idler tension. The drive gear cannot grip filament without manual pressure applied externally. Symptoms:
- Motor clicks and skips on filament load
- Tube bows outward under pressure (feeder pushing faster than head can accept)
- Motor will move filament only when you press on the drive gear by hand

**Root cause:** Idler spring rate and/or geometry is inadequate. Even at maximum adjustment the idler does not generate enough clamping force to pull filament against resistance in the bowden path.

**Workaround:** Manually guide filament through the feeder path during load until it reaches the chroma head selector.

**Permanent fix:** Replace ChromaSet feeder units with quality extruders (Orbiter v2.0, BMG, or LGX Lite) wired to the same cp_Driver step/dir/enable pins.

### 2. Feeder Motor Current Not Configurable
The cp_Driver board communicates via USB serial only. There is no TMC UART or SPI configuration exposed to Klipper — motor run current is fixed at the board's hardware default (unknown, likely 0.4-0.6A). This compounds the grip issue above.

### 3. ChromaSet Slot Sensors Not Wired to Klipper
The per-slot filament present/absent LEDs on the ChromaSet are hardware-only. The sensor signal lines are not connected to any GPIO on the cp_Driver MCU and cannot be polled by Klipper. The only active filament sensor is the main machine `runout_sensor` at `!PB14` (motherboard).

### 4. Feeder Rotation Distance Mismatch (Fixed in This Config)
Factory kcm.cfg shipped with `rotation_distance: 7.6798493425843` for ex1-ex4. The chroma head extruder was calibrated to `rotation_distance: 4.1884`. This caused Klipper to command mismatched step counts between feeder and head, contributing to tube pressure.

**Fix applied:** ex1-ex4 calibrated by measurement — commanded 100mm, measured 98.17mm (3.865in actual), new value: `7.539`.

### 5. cp_macro.cfg Duplicate Macro Conflict
The factory cp_macro.cfg defines `LOAD_FILAMENT`, `START_PRINT`, `END_PRINT`, `PAUSE`, `RESUME`, and `CANCEL_PRINT` — all duplicating macros in macros.cfg. The last-included file wins in Klipper, silently overriding the primary macros. The cp_macro.cfg `LOAD_FILAMENT` did a blind 100mm extrude with no sensor check.

**Fix applied:** Duplicate `LOAD_FILAMENT` and `PARK_FILAMENT` removed from cp_macro.cfg.

---

## Firmware Fixes Implemented

| Fix | File | Description |
|---|---|---|
| Feeder rotation_distance calibrated | `kcm.cfg` | ex1-ex4 updated from 7.6798 to 7.539 (measured) |
| Duplicate LOAD_FILAMENT removed | `cp_macro.cfg` | Removed 100mm blind extrude override |
| Extruder control block conflict resolved | `printer.cfg` / `chroma_head.cfg` | SAVE_CONFIG PID lives in printer.cfg, chroma_head.cfg does not re-declare control |
| AUTO_BED_LEVEL wipe coordinates fixed | `macros.cfg` | Corner-origin machine: wipe X was -50 (out of range), fixed to X=55/X=155 |
| bed_mesh.cfg duplicate speed values removed | `bed_mesh.cfg` | Removed duplicate speed/horizontal_move_z entries |
| Chroma head rotation_distance calibrated | `chroma_head.cfg` | 4.5552 → 4.1884 (commanded 100mm, measured 91.95mm) |
| Serial paths stabilized | `kcm.cfg` / `chroma_head.cfg` | Switched from ttyACM0/1 to /dev/serial/by-id/ stable paths |

---

## Configuration Structure

| File | Purpose |
|---|---|
| `printer.cfg` | Top-level includes + SAVE_CONFIG PID/offset blocks |
| `printer.base.cfg` | Machine kinematics, steppers, bed, fans, endstops |
| `chroma_head.cfg` | ChromaSet chroma head MCU (cp_Head) — extruder, hotend, servo |
| `kcm.cfg` | ChromaSet driver MCU (cp_Driver) — ex1-ex4 feeder steppers, T0-T4 macros |
| `macros.cfg` | Custom macros: START_PRINT, END_PRINT, AUTO_BED_LEVEL, LOAD_FILAMENT, etc. |
| `macros-pro.cfg` | AD5M Pro specific macros |
| `cp_macro.cfg` | ChromaSet vendor macros: FILAMENT_CHANGE, SET_FILAMENT, tool selection UI |
| `bed_mesh.cfg` | Bed mesh calibration settings |
| `input_shaper.cfg` | Input shaper configuration |
| `client.cfg` | Mainsail/Fluidd client macros |
| `ecm_1.cfg` – `ecm_4.cfg` | ECM expansion boards for EX5-EX16 (uncomment in printer.cfg to enable) |

## ECM Expansion Note

- Base setup supports EX1 through EX4 via `kcm.cfg`
- To add more feeders, enable ECM configs in `printer.cfg`:
  - `ecm_1.cfg` for EX5–EX8
  - `ecm_2.cfg` for EX9–EX12
- Update MCU serial mappings in each ECM file to match your hardware

## Coordinate System

This machine uses a **corner-origin** coordinate system (not center-origin):
- X: 0–200mm
- Y: 0–197mm
- Any macros using negative X or Y coordinates will be out of range
