# Overview

Drop-in replacement for the Audi C5 allroad air suspension control module (J197).

This file is the orientation page. Pin numbers, parts, firmware, and VCDS live in sibling files.

## Vehicle

| Field | Value |
|-------|--------|
| Car | 2001 Audi allroad C5 (4Z7) |
| VIN | `WAUYP64B31N165751` |
| Miles | about 243k |
| Owner | Lee Welsch |
| Module location | Left cargo-area side panel. Both plugs reachable by opening only the toolbox cover ([quattroworld](https://forums.quattroworld.com/allroad/threads/25759.phtml)). |

## Stock module

| Field | Value |
|-------|--------|
| VAG address | J197, VCDS address **34** |
| Part | `4Z7 907 553 B` |
| Supplier | Continental / Hella `5SG 008 352-03` |
| Internals (one source, unverified) | ST10F269 MCU and 24C16 EEPROM |
| Index A vs B | SSP 243: `553 A` and `553 B` use different control strategies. Lee's units are B. Follow the B section (SSP 243 p.12). |

Lee has two `553 B` modules:

| Printed HW/SW | Date | Continental | Serial | Role |
|---------------|------|-------------|--------|------|
| H04 / S0255 | 17.08.00 | `15.15228-0007.2` | 000377 | Lee identifies as H04/S025. Thought to be the installed unit. |
| H05 / S0331 | 27.03.01 | `15.15228-0016.2` | 001511 | Lee identifies as H05/S031. Spare. Photo: [sources/module-label.jpg](sources/module-label.jpg). |

Both labels also read `4Z7 907 553 B` and `5SG 008 352-03`.

## What the stock system does

Source: [SSP 243](sources/ssp243.pdf) (Audi self-study, 4-level air suspension).

- Fully supporting air springs at all four corners. Pressure-accumulator system.
- Four ground-clearance levels: **142 / 167 / 192 / 208 mm** (low / normal / high 1 / high 2). Parking level = high 1.
- Level from four inductive angle sensors (PWM to J197). Right-hand sensors powered by J197; left-hand sensors powered by headlamp-range unit J431, which J197 itself feeds so run-on still works.
- Six solenoids: corner valves N148–N151, accumulator N311, discharge N111. Compressor motor V66 via relay J403. Warning lamp K134.
- Dash unit E281 talks to J197 on a dedicated K-wire (not the diagnostic K-line).
- Drive CAN plus a few hardwired signals (speed, trailer F216, doors, central-lock pulse, KL50).
- Self-diagnosis on the diagnostic K-line at address 34.

Level change order (SSP 243): **raise rear axle first, then front. Lower front axle first, then rear.**

Speed lowering (SSP 243, Index strategy; confirm on Lee's B unit before coding it as law):

- HL2 → HL1 above 35 km/h. HL2 request only below 30 km/h.
- HL1 → normal above 80 km/h. HL1 request only below 75 km/h.
- No automatic raise to HL1/HL2 while driving, except parking-level raise after lock.

## Why replace it

1. **EEPROM / fault 65535.** Calibration lives in a small EEPROM. A bad write produces VCDS 65535 (internal/control-module fault) and kills levelling. The replacement stores calibration in FRAM with A/B copies, CRC32, and a sequence counter.
2. **Water.** Cargo-area location. Replacement: conformal coat, IP67 box, plugs facing down, vent membrane.
3. **Voltage.** Reverse, load dump, brownout. Replacement: ideal-diode + TVS + buck, ADC watch on KL30, valves closed if the rail sags.

New OEM modules are scarce and expensive. Used `553 B` units carry the same EEPROM and water risk.

## What this project is

A board and firmware that:

- Plugs into the stock harness (pigtail first; true drop-in header later if a housing is identified).
- Drives the stock valve block, compressor relay, lamp, and J431 feed.
- Reads the stock height sensors, G291, G290, and the hardwired inputs.
- Levels to the four SSP heights with the stock raise/lower order and speed rules.
- Speaks drive CAN (listen-only until decoded).
- Speaks E281 on its K-wire (protocol unknown until traced).
- Emulates VCDS address 34 so Ross-Tech still works.
- Offers the OAS-Man phone app over BLE for everyday use (Lee, 23 Oct 2026).

It is **not** a new manifold, not an AirLift clone on this car, and not a pressure-only controller.

## Relationship to OAS-Man

This repo is a fork of [gopro2027/ArduinoAirSuspensionController](https://github.com/gopro2027/ArduinoAirSuspensionController) (OAS-Man), GPL-3.0. The upstream README is [../UPSTREAM_OASMAN_README.md](../UPSTREAM_OASMAN_README.md).

Reuse: solenoid / compressor / manifold classes, BLE stack, phone app. Do not reuse OAS-Man's pressure-only loop as the vehicle's height truth, and do not enable `SIX_VALVE_MANIFOLD` until the J197 valve map is measured and written down.

CAN / power-front-end ideas come from [Forbes-Automotive/OpenHaldex-C6](https://github.com/Forbes-Automotive/OpenHaldex-C6) (MIT). Keep that notice on copied blocks.

## Interfaces (approved by Lee, 23 Oct 2026)

1. **Everyday:** OAS-Man Android app over BLE.
2. **Workshop:** VCDS (or compatible) at address 34 on the diagnostic K-line.

VCDS output tests and basic settings outrank app commands.

## Status

Design v0.1. Docs and a KiCad net-label skeleton. Most pins unknown. No allroad firmware env. The working J197 stays in the car until the replacement is bench-proven.

## Read next

1. [PINOUT.md](PINOUT.md) — what is known, what is not
2. [HARDWARE.md](HARDWARE.md) — blocks and connector plan
3. [FIRMWARE_PLAN.md](FIRMWARE_PLAN.md) — what to write
4. [TEST_PLAN.md](TEST_PLAN.md) — what to measure
5. [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) — blockers
