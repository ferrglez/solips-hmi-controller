# SOLIPS HMI Controller — Rev A

**Hardware design · KiCad · Engineering review pending**

> **Status:** Rev A design submitted for peer review. **Not approved for fabrication.** ERC/DRC were reported clean in the working KiCad project; electrical, mechanical and physical validation are still pending.

## Overview

SOLIPS HMI Controller Rev A is a dedicated four-layer PCB intended to offload the graphical human–machine interface (HMI) from an Arduino UNO Q host. The host provides semantic application data; the STM32 executes the graphical application locally and drives a GC9A01 circular TFT display.

The design follows the engineering baseline dated **30 September 2026**. That document freezes the major architecture but explicitly leaves component-specific verification and fabrication gates open. The current KiCad implementation must be compared with that baseline during review.

## System architecture

```text
Arduino UNO Q / QRB2210 (Debian)
           │ USB 2.0
           ▼
CP2102C-A01-GQFN24 (USB–UART bridge)
           │ USART1 · 3.3 V · PA9/PA10
           ▼
STM32U585CIT6 (LQFP48)
           │ SPI1 + DMA
           ▼
GC9A01 · 240 × 240 · RGB565
```

**Design rationale:** The UNO Q retains system and application logic, while the dedicated STM32 handles `VehicleState`, `UIManager`, LVGL and graphical assets. This reduces the HMI rendering workload on the host. Firmware migration and performance are **not yet demonstrated on this PCB**.

## Key engineering choices

| Subsystem | Rev A baseline | Rationale / verification |
| --- | --- | --- |
| Main MCU | STM32U585CIT6, LQFP48 | Dedicated local HMI processing; confirm package pinout, power domains and firmware budget |
| Host interface | CP2102C-A01-GQFN24 | USB–UART with DTR/RTS for planned boot/reset control; validate electrical implementation |
| Serial | USART1 on PA9/PA10 | Shared application communication and planned ROM bootloader update path |
| Display | GC9A01, 240×240, RGB565 | SPI1 + DMA, write-only interface; confirm actual module and connector pinout |
| Supply | USB 5 V input; 3.3 V LDO ≥500 mA | Simpler bring-up and no switching regulator; confirm load, inrush and thermal budget |
| Debug | SWD, manual BOOT/RESET | Recovery path independent of the USB bridge |
| PCB | 4 layers | Top signals/components, inner GND reference, inner power, bottom signals; verify actual stackup and return paths |

## KiCad project organization

The schematic is organized into five hierarchical sheets:

1. `01_POWER.kicad_sch` — USB input and regulation
2. `02_USB_UART.kicad_sch` — USB-C and CP2102C
3. `03_MCU.kicad_sch` — STM32 and boot/reset control
4. `04_DISPLAY.kicad_sch` — GC9A01 interface
5. `05_DEBUG.kicad_sch` — SWD and manual controls

The **current authoritative KiCad files** should be placed in `hardware/`, including the top-level `.kicad_sch`, `.kicad_pcb`, `.kicad_pro`, all five hierarchical sheets, and any required custom symbol/footprint libraries or board rules. The duplicate `- copia` folder is a local backup, not a second design release.

## Layout and routing

- Four-layer PCB with inner reference and power planes.
- Placement and routing performed in KiCad; FreeRouting was used as a routing aid, followed by manual completion and DRC cleanup.
- USB D+/D−, ground continuity, via transitions, connector orientation and mechanical fit still require an engineering review.
- The most recent DRC/ERC status was reported as clean by the designer; review the committed `DRC.rpt` / `ERC.rpt` and confirm they were generated **after the final edits**.

## Review checklist

- [ ] Cross-check actual KiCad schematic against STM32U585CIT6 and CP2102C manufacturer datasheets
- [ ] Verify USB-C CC resistors, ESD, USB data routing, shield and VBUS protection
- [ ] Verify MCU VDD/VDDA/VCAP and all decoupling connections
- [ ] Validate LDO part, USB current budget and worst-case thermal dissipation
- [ ] Verify DTR/RTS–BOOT0/NRST polarity, default states and STM32 option bytes
- [ ] Verify SWD header orientation and physical accessibility
- [ ] Confirm GC9A01 **actual module** pinout, supply, backlight and connector geometry/orientation
- [ ] Check actual four-layer stackup, copper returns, clearances and manufacturability
- [ ] Regenerate ERC/DRC on the final committed project
- [ ] Close fabrication gates FG-01 / FG-02 / FG-03 with evidence

## Planned validation

The baseline defines T01–T11: power, USB enumeration, UART, SWD, manual and automatic bootloader, CLI flashing, application boot, display, HMI screens and host protocol. **These are planned tests, not completed tests.**

## Documentation

- [`docs/Engineering_Specification.docx`](docs/Engineering_Specification.docx) — original engineering requirements and schematic baseline (v1.0, 2026-09-30).

## Review request

**Requested decision:** `APPROVED FOR NEXT STAGE` / `CHANGES REQUIRED` / `NOT APPROVED`.

**No fabrication order should be placed until the open electrical, mechanical and fabrication-gate checks are resolved.**
