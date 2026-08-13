# Retro Handheld Gaming Console

A fully custom-built retro gaming handheld, built from discrete components rather than a pre-made kit — designed, wired, and coded from scratch around a Raspberry Pi Zero 2W running RetroPie.

## Motivation

Off-the-shelf retro handhelds abstract away the hardware entirely. This project was built to actually understand and control every subsystem — display driving, input handling, audio, power delivery, and battery management — instead of just flashing an emulator image onto a commercial board. It doubled as a hands-on exercise in integrating several hardware domains (digital I/O, analog sensing, power electronics) with embedded Linux into a single, physically packaged product.

## Features

- 3.5" SPI TFT display (RetroPie-driven emulation UI and gameplay)
- Custom physical controls: D-pad and action buttons, dedicated volume control, and custom speed/turbo (fast-forward) buttons
- I2S audio output through a dedicated amplifier
- Wi-Fi and Bluetooth connectivity (ROM transfer, wireless controllers, etc.)
- Rechargeable dual-cell Li-ion battery pack with protection circuitry and live voltage monitoring
- Fully 3D-printed enclosure/shell

## Resources

- Budget Sheet: https://docs.google.com/spreadsheets/d/1srqWW1JxbvY3oqgRoeJ9-f11bGZaD1YcLMDTcUJRfcI/edit?usp=sharing
- CAD Models: https://cad.onshape.com/documents/c98114f3d22adb09159bdb3a/w/b0b86f17a521fcfeee855476/e/23fd8016c83f44d86354cdb0?renderMode=0&uiState=6a7dff6e3107582e18cbe90c
- Circuit Diagram: https://app.cirkitdesigner.com/project/7a7ec9c0-ae09-4871-93af-93d7c8916aee

## Hardware

| Subsystem | Components |
|---|---|
| Compute | Raspberry Pi Zero 2W, RetroPie |
| Display | 3.5" SPI TFT, ILI9486 driver, running stable at 16MHz SPI |
| Input | MCP23017 I/O expander reading all physical buttons, exposed to RetroPie as a virtual gamepad via a custom `uinput` driver (`mcp_gamepad.py`) |
| Audio | PCM5102 I2S DAC → PAM8403 amplifier |
| Connectivity | Onboard Wi-Fi and Bluetooth |
| Battery | 2× Li-ion cells wired in parallel (1S pack) |
| Charging | TP4056 module |
| Protection | Dedicated 1S BMS between the raw cells and the rest of the circuit, handling over-charge/over-discharge/over-current protection (see *Design Iterations* below) |
| Power rail | MT3608 boost converter, stepping the battery up to a stable 5V rail |
| Battery monitoring | Voltage-divider sensor into a PCF8591 external ADC over I2C (calibrated multiplier), since the Pi's GPIO has no native analog input |
| Enclosure | Custom-designed, 3D-printed shell |

## Software & Debugging

- Wrote `mcp_gamepad.py`, a custom driver that polls the MCP23017 over I2C and injects button presses into the OS as a virtual gamepad via `uinput`, and fixed a D-pad input loop bug in it
- Resolved wavy/unstable SPI display output
- Fixed boot hangs by disabling `networkd-wait-online` and `networking.service`
- Wrote battery logging/plotting scripts to track pack voltage over time and validate the monitoring circuit

## Performance Tuning

- CPU governor set to `performance`
- Framebuffer corrected to 480×320 to match the display
- Switched the GBA emulation core to `lr-gpsp` for better performance
- Disabled Bluetooth during heavy emulation sessions to reduce overhead where not needed

## Design Iterations: Power System

The power system went through a real debugging and redesign cycle rather than working on the first attempt:

1. **Symptom:** The Pi would spontaneously reset while running on battery, even though the display and amplifier stayed powered through the reset — with the 5V rail sagging to ~4.1–4.2V during the event.
2. **Root cause:** Diagnosed (through systematic voltage/current testing) as the TP4056 module's onboard protection IC (DW01A/FS8205A) tripping under load current spikes — not a fault in the battery or the MT3608 boost converter itself. Powering the Pi via direct solder pads instead of micro-USB removed the USB port's onboard input filtering, making the issue more sensitive.
3. **Immediate fix:** Bypassed the TP4056's protection IC output and wired the battery's B+/B- straight to the MT3608 boost converter input.
4. **Proper fix:** Replaced the stopgap bypass with a standalone 1S BMS rated for higher discharge current, placed inline between the raw cells and the rest of the circuit. The TP4056 was kept dedicated to charging only, now fed from the BMS's protected output. (A TP5100 charging module was considered and rejected — no USB port, requires wire soldering.)

## Skills Involved

- **Embedded electronics:** power system design (charging, protection, boost conversion), I2C/SPI peripheral integration, analog signal conditioning and voltage-divider sensing
- **Low-level hardware debugging:** root-causing an intermittent boot failure by tracing a voltage sag under load back to a protection IC tripping on current spikes, through systematic voltage/current testing
- **Embedded Linux:** boot process tuning, service management, framebuffer/driver configuration
- **Software:** custom `uinput` gamepad driver bridging GPIO hardware to the OS, battery logging/monitoring scripts
- **Mechanical design:** 3D-printed enclosure design
- **Systems integration:** combining independently-sourced modules (display, input, audio, power, connectivity) into one coherent, packaged device, iterating on the design as real-world issues surfaced

## Supported Systems

Currently capable of emulating GBA, PS1, PSP, N64, and other classic consoles/handhelds, running through RetroArch cores under RetroPie.

## Status

Actively maintained — ongoing refinements to battery monitoring accuracy and a potential display upgrade.

**Future plans:**
- Bluetooth controller support so the handheld can double as a Bluetooth gamepad for playing on a laptop/TV
- Further reducing boot time
- General software/UI polish to make the RetroPie front-end feel more finished (custom theming, cleaner menus, etc.)
- Other general quality-of-life improvements as they come up
