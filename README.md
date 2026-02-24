# Mingalobox

Mingalobox is a **wireless Hitbox / arcade-style game controller** project built around **nRF52840 + RP2040** (controller + USB dongle), with RP2040 running GP2040 firmware.

> For me, this is a milestone—both a summary of what I’ve accomplished so far and a new challenge. As a fighting game enthusiast, it’s rare to be able to build a device that truly meets my own expectations. I’ve also really enjoyed learning in a new domain and working through problems along the way.

This repository contains:

- `hardware/`: schematics/PCB/BOM/Gerbers and enclosure models.
- `firmware/`: nRF52840 wireless link firmware prototype based on **nRF Connect SDK (NCS) / Zephyr** Gazell samples (device + host). This part is under `LicenseRef-Nordic-5-Clause` and includes restrictions (for example, “Nordic IC only”).

> Note: `firmware/` currently focuses on validating the nRF52840 Gazell link. The RP2040 side is expected to run third-party GP2040(/GP2040-CE) firmware (not included in this repo).

## System Architecture (Target)

Wireless mode:

`nRF52840 transmitter (controller)` → `nRF52840 receiver (dongle)` → `Waveshare RP2040-Zero (GP2040 firmware)` → `PC / PS4 / PS5, etc.`

Wired mode:

- Optionally solder a **Raspberry Pi Pico (RP2040)** onto the controller PCB and flash GP2040, then the controller can be used standalone as a **wired** device (no dongle required).

## Repository Layout

- `firmware/controller/`: Gazell **Device** role (controller-side prototype)
- `firmware/dongle/`: Gazell **Host** role (dongle-side prototype)
- `hardware/controller/`: controller PCB/schematics/BOM/Gerbers
- `hardware/dongle/`: dongle PCB/schematics/BOM/Gerbers + 3D enclosure models

## Firmware: Build & Flash (NCS / Zephyr)

### Prerequisites

You need a working **Zephyr/NCS** environment and `west`.

#### About the “west workspace”

`west` is Zephyr/NCS’ workspace and build tool. Typically you create an NCS **west workspace** (containing `zephyr/`, `nrf/`, `nrfxlib/`, etc.), then build this repo’s `firmware/*` apps inside that workspace.

This repository itself is **not** a full west workspace (no `west.yml` / `.west/` at the repo root). Recommended: place this repo somewhere inside your existing NCS workspace, or build it as an external application from the workspace.

### Build

Example for `nrf52840dk_nrf52840` (nRF52840 DK). `sample.yaml` also lists `nrf52dk_nrf52832` and `nrf52833dk_nrf52833`.

```bash
west build -b nrf52840dk_nrf52840 -s firmware/controller -d build/controller
west build -b nrf52840dk_nrf52840 -s firmware/dongle     -d build/dongle
```

### Flash

```bash
west flash -d build/controller
west flash -d build/dongle
```

### Notes

- These samples exchange 1-byte payloads over Gazell and use the DK buttons/LEDs (`dk_buttons_and_leds`) for a quick demo.
- To complete the full pipeline “buttons → wireless → RP2040(GP2040) → USB HID”, additional integration is required on both controller/dongle sides (real button inputs and the interface/protocol to RP2040).
- `gpio_output_voltage_setup_3v3()` in both `main.c` writes UICR `REGOUT0` and triggers a reset (non-volatile configuration write). Use with care on production hardware.

## Hardware

Hardware files live in `hardware/`:

- Key parts (current version):
  - nRF52840: Ebyte `E73-2G4M08S1C` module (ceramic antenna)
  - Controller RP2040: `Raspberry Pi Pico` (optional, for wired GP2040 mode)
  - Dongle RP2040: `Waveshare RP2040-Zero` (USB side running GP2040)
- `*/sch/*.json`, `*/pcb/*.json`: schematic/PCB source files (editor depends on the export source)
- `*/gerber/*.zip`: Gerber packages
- `*/bom/*.xlsx`: BOMs
- `hardware/dongle/3DShellModel/*.stl`: dongle enclosure models

## License

- Unless stated otherwise, author-written content in this repository is licensed under `Apache-2.0` (see `LICENSE` and `NOTICE`).
- `firmware/controller/` and `firmware/dongle/` include Nordic Semiconductor ASA sample/derived code and are marked with `SPDX-License-Identifier: LicenseRef-Nordic-5-Clause` (see `LICENSES/LicenseRef-Nordic-5-Clause.txt`).
  - This Nordic license includes restrictions (for example “Nordic IC only”), so those parts are **not** licensed under `Apache-2.0`.

## Feedback

Issues are welcome (no guarantee of response time or acceptance). For PRs, please open an issue first to align on scope/approach.
