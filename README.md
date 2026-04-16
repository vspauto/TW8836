# TW8836 Firmware

Legacy embedded firmware for the Intersil/Techwell `TW8836` video processor family. This repository contains a recovered firmware source tree targeting a `DP80390` / 8051-class MCU environment, with support for video input selection, scaling, on-screen display, settings persistence, and board-level peripherals commonly used in TW8836 evaluation or product designs.

The checked-in configuration is centered on a `TW8836` build with an `800x480` TCON panel path, analog and digital video inputs, SPI-based OSD assets, touch and keypad input, and flash-backed EEPROM emulation. The codebase also includes a UART monitor/debug shell, BT.656 routing support, and HDMI bridge integration using the `EP907M`.

> [!NOTE]
> This repository appears to be a preserved/imported firmware tree rather than a fully reconstructed buildable project. It includes historical release notes and source files, but it does not currently include a checked-in Keil project/workspace file or a fully reproducible build setup.

## What This Repository Contains

This tree is organized as a monolithic embedded firmware project with most source files placed at the repository root.

- `main.c` implements boot flow, core initialization, system setup, and the main runtime loop.
- Input pipeline modules handle source detection, timing, routing, and format-specific behavior:
  - `Decoder.c`
  - `aRGB.c`
  - `DTV.c`
  - `BT656.c`
  - `InputCtrl.c`
  - `measure.c`
- Output, scaler, and display-control modules configure the panel path and runtime video processing:
  - `Scaler.c`
  - `OutputCtrl.c`
  - `Settings.c`
  - `InitReg.c`
- OSD and UI modules implement both Font OSD and SPI OSD menu systems:
  - `SOsdMenu.c`
  - `SOsdMenuMap.c`
  - `FOsdMenu.c`
  - `FOsdDisplay.c`
  - `FOsdString.c`
  - `FOsdTable.c`
  - `OSDFont.c`
  - `OSDSPI.c`
  - `DemoMain.c`
- Platform and peripheral support covers CPU bring-up, buses, persistent settings, human input, and the debug shell:
  - `CPU.c`
  - `i2c.c`
  - `SPI.c`
  - `EEPROM.c`
  - `TouchKey.c`
  - `Remo.c`
  - `HID.c`
  - `monitor.c` and related `monitor_*.c` files
- The [`Data`](./Data) directory contains compiled-in data assets such as initialization tables and font data includes.
- Register-definition headers such as `EP907M_RegDef.H`, `EP9351_RegDef.h`, `EP9553_RegDef.h`, and `EP9x53RegDef.h` document external HDMI bridge or companion-chip register layouts used by the firmware.

## Configuration Snapshot

The currently active configuration in `config.h` indicates the following baseline setup:

- Target model: `MODEL_TW8836`
- CPU/toolchain context: `DP80390`, 8051-family MCU, Keil C style codebase
- Firmware version macro: `FWVER 0x003`
- Default panel path: `PANEL_TCON`
- Default panel resolution: `800x480`
- Panel clock range: `30` to `45` MHz
- EEPROM mode: flash-backed EEPROM emulation via `USE_SFLASH_EEPROM`
- OSD mode: `SUPPORT_SPIOSD` with Font OSD menu support enabled
- Human input features: `SUPPORT_TOUCH`, `SUPPORT_KEYPAD`, RC5 remote configuration
- Enabled video/input features:
  - `SUPPORT_CVBS`
  - `SUPPORT_SVIDEO`
  - `SUPPORT_COMPONENT`
  - `SUPPORT_PC`
  - `SUPPORT_HDMI`
- Active HDMI bridge variant: `SUPPORT_HDMI_EP907M`

This README documents the checked-in active configuration, not every historical compile-time option left in comments or alternate branches of `config.h`.

## Build And Toolchain Notes

This repository appears intended for classic Keil C51-style embedded development rather than a modern standalone build system.

Evidence in the tree includes:

- `L51_BANK.A51`, which points to Keil linker/code banking usage
- `REG390.H`, a Keil-era 8051/DP80390 support header
- Use of `<intrins.h>` in multiple source files
- Widespread use of memory qualifiers and conventions such as `XDATA`, `IDATA`, `CODE`, `DATA`, and `sfr`

At the moment, no Keil project/workspace file is checked into the repository. There are no `.uvproj`, `.uvprojx`, `.uvopt`, or equivalent IDE project files present in the current tree. That means exact build reproduction will likely require reconstructing:

- Target/device selection
- Compiler memory model and banking settings
- Include paths
- Linker configuration
- Startup/runtime options
- Output image settings for the intended board or flash layout

The source tree should therefore be treated as a preserved firmware codebase with strong toolchain clues, not as a guaranteed one-command build.

## Repository Layout

The repository is compact but dense:

- `44` C source files
- `65` header files
- `1` assembler banking file: `L51_BANK.A51`
- `1` release-history text file: `Release.txt`
- `4` data include assets under `Data/`

High-level layout:

```text
TW8836/
|-- Data/
|   |-- DataComponent.inc
|   |-- DataInitMonitor.inc
|   |-- DataInitPC.inc
|   `-- RamFontData.inc
|-- main.c / config.h / TW8836.h / reg.h / typedefs.h
|-- input-processing modules
|-- scaler/output/display modules
|-- OSD and menu modules
|-- monitor and debug modules
|-- peripheral support modules
`-- release notes and register-definition headers
```

## Runtime Capabilities

Based on the checked-in modules and active configuration, this firmware tree includes support for:

- UART monitor/debug shell for interactive inspection and commands
- Font OSD and SPI OSD rendering paths
- Input switching across analog and digital video sources
- Video measurement, scaling, and panel timing control
- EEPROM or flash-backed settings persistence
- Touch key, keypad, and IR remote handling
- BT.656 routing/output control
- HDMI bridge support via the `EP907M`

## Known Gaps And Caveats

- No build project/workspace file is checked in, so the original IDE setup is incomplete.
- Hardware dependencies are external to the repo. Building and running this firmware still depends on the original board design, panel, flash layout, and connected peripherals.
- Some files appear archival or experimental rather than clean production modules, especially `junk.c`.
- `Release.txt` indicates this tree was copied from another source and preserves historical bugs, TODO items, and release notes.
- Some includes reference assets or headers that are not present as standalone files in the current repository, suggesting parts of the original environment may be missing. Examples include `SpeedoMeter.h`, `Clock.h`, `SlideMenu.h`, `GridLine.h`, `Compass.h`, `OSD_Image\\Harry_NoRLE_Header.c`, and `\\data\\WideModeData.txt`.
- This README does not claim that the code has been rebuilt, flashed, or validated on hardware from the current repository state.

## Getting Started For Maintainers

If you are approaching this codebase for restoration or documentation work, start with these files:

1. `Release.txt` for provenance and historical notes
2. `config.h` for the active build-time feature selection
3. `main.c` for boot flow and runtime structure
4. `InputCtrl.c` for source switching and pipeline entry points
5. `Scaler.c` for output scaling behavior
6. `monitor.c` for the debug shell and maintenance surface area
7. `SOsdMenu.c` for the SPI OSD UI path
8. `SPI.c` for flash access, EEPROM emulation, and SPI OSD transfer helpers

For build restoration, the most practical next step is to recreate a Keil project and capture the exact target, memory model, banking, include-path, and linker settings needed by this source tree.

## Licensing And Provenance

Source headers in this repository include copyright notices such as:

- `Copyright (C) 2011~2012 Intersil Corporation`

The release history also records an import note:

- `260415 Copy from https://github.com/lgnq/tw8836/tree/master/demo/Source36`