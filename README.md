# C5 allroad air suspension controller

Drop-in replacement for the 2001 Audi C5 allroad air suspension control module (J197, `4Z7 907 553 B`, Continental/Hella `5SG 008 352-03`, VCDS address 34).

This repository is Lee Welsch's fork of [OAS-Man](https://github.com/gopro2027/ArduinoAirSuspensionController) (GPL-3.0). The OAS-Man firmware tree is still here. This pass is documentation and hardware scaffolding only. No firmware behaviour has been changed.

**Status:** design v0.1. Most J197 harness pins are unknown. Do not assign unknown pins. Do not flash or drive valves from this work.

## Why

Lee's car is a 2001 Audi allroad C5, VIN `WAUYP64B31N165751`, about 243k miles. The stock module fails in ways that are hard to repair:

- **EEPROM corruption (VCDS fault 65535).** The original module stores calibration in a 24C16-class EEPROM (one source, unverified). A bad write leaves the car with an internal fault and no level control.
- **Water.** The module sits in the left cargo-area side panel. Water ingress is a known kill.
- **Voltage.** Load dump, reverse battery, and brownout damage the stock supply. The replacement parks valves closed on brownout.

Index B (`553 B`) uses the control strategy on SSP 243 p.12, not the Index A strategy.

Lee has two `553 B` modules:

| Unit | HW / SW (Lee) | Date | Role |
|------|---------------|------|------|
| Continental `15.15228-0007.2` | H04 / S025 | 17.08.00 | Thought to be the installed unit |
| Continental `15.15228-0016.2` | H05 / S031 | 27.03.01 | Spare. Label photo in `docs/allroad/sources/module-label.jpg` (printed line reads `H05 / S0331`) |

## Relationship to OAS-Man

OAS-Man is an open-source ESP32 air-suspension controller (manifold + wireless controller + Android app) by [gopro2027](https://github.com/gopro2027/ArduinoAirSuspensionController). Credit and thanks to that project and its contributors.

This fork keeps the OAS-Man tree so solenoid, compressor, manifold, BLE, and the phone app can be reused. J197 is not an aftermarket 8-valve manifold: it levels from height-sensor PWM, talks VAG CAN and K-line, and drives a 6-valve OEM block plus compressor relay. New modules will wrap the OAS-Man core. See [docs/allroad/FIRMWARE_PLAN.md](docs/allroad/FIRMWARE_PLAN.md).

The original OAS-Man README is at [docs/UPSTREAM_OASMAN_README.md](docs/UPSTREAM_OASMAN_README.md).

**Licence:** firmware derived from OAS-Man stays **GPL-3.0**. Hardware blocks copied from [OpenHaldex-C6](https://github.com/Forbes-Automotive/OpenHaldex-C6) keep that project's MIT notice. See [hardware/allroad-j197/LICENSES.md](hardware/allroad-j197/LICENSES.md) and the root [LICENSE](LICENSE).

## Current status

| Area | State |
|------|--------|
| Design notes | v0.1, frozen in `docs/allroad/` |
| Pinout | A few confirmed or likely pins. Most unknown. |
| KiCad | Net-label skeleton only. No harness pin numbers. No connector symbols. |
| Firmware | Unchanged OAS-Man tree. No allroad env yet. |
| VCDS | Plan only. Protocol most likely KWP1281 at address 34. Unverified. |

## Roadmap

1. **Measure** — housings, meter pinout, valve coil ohms, CAN log, E281 logic trace, VCDS capture of a working module. [TEST_PLAN.md](docs/allroad/TEST_PLAN.md)
2. **Pinout** — move pins from unknown to confirmed only after a measurement. [PINOUT.md](docs/allroad/PINOUT.md), [OPEN_QUESTIONS.md](docs/allroad/OPEN_QUESTIONS.md)
3. **Schematic / PCB** — assign connector symbols, finish the v0.1 blocks, pigtail adapter. [HARDWARE.md](docs/allroad/HARDWARE.md)
4. **Firmware** — ESP32-S3 port of OAS-Man core; PWM height loop; FRAM A/B + CRC; CAN listen-only; E281 serial; VCDS responder. [FIRMWARE_PLAN.md](docs/allroad/FIRMWARE_PLAN.md)
5. **Bench** — valves, sensors, brownout park, FRAM corruption test, K-line against VCDS. No car until that passes.
6. **Car, pigtail** — cargo-area install with donor plugs. Keep the working J197.
7. **v0.2 option** — reuse a desoldered J197 header for a true drop-in.

Blocking for v0.2: ELSA/Bentley current-flow diagram, VAG housing numbers, J1-9 conflict, E281 protocol, CAN messages J197 sends.

## Docs

| File | Contents |
|------|----------|
| [docs/allroad/OVERVIEW.md](docs/allroad/OVERVIEW.md) | Vehicle, module, goals |
| [docs/allroad/PINOUT.md](docs/allroad/PINOUT.md) | Confirmed / likely / unknown pins |
| [docs/allroad/HARDWARE.md](docs/allroad/HARDWARE.md) | Blocks, connector strategy |
| [docs/allroad/BOM.md](docs/allroad/BOM.md) | Draft parts |
| [docs/allroad/FIRMWARE_PLAN.md](docs/allroad/FIRMWARE_PLAN.md) | What to reuse, what to write |
| [docs/allroad/DIAGNOSTICS_VCDS.md](docs/allroad/DIAGNOSTICS_VCDS.md) | Address 34 / KWP1281 plan |
| [docs/allroad/TEST_PLAN.md](docs/allroad/TEST_PLAN.md) | What Lee measures next |
| [docs/allroad/DECISIONS.md](docs/allroad/DECISIONS.md) | Locked choices |
| [docs/allroad/OPEN_QUESTIONS.md](docs/allroad/OPEN_QUESTIONS.md) | Unknowns |
| [docs/allroad/sources/](docs/allroad/sources/) | SSP 243 and the module label photo |

Agent entry points: [CLAUDE.md](CLAUDE.md), [AGENTS.md](AGENTS.md).

## Hardware

KiCad skeleton and reference copies: [hardware/allroad-j197/](hardware/allroad-j197/).

## Rule for later agents

Unknown pins stay marked **unknown** until measured. Do not invent pin numbers, housing part numbers, or VCDS ID strings. Capture those from the car or the working module.
