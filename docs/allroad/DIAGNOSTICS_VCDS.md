# Diagnostics / VCDS

Goal: Ross-Tech VCDS (or a compatible tester) still opens **address 34** and sees a J197-shaped module.

Everything in the emulate table is **unverified** until Lee captures the working module. Do not invent an ID string, coding, or block layout.

## Protocol (unverified)

Most likely **KWP1281**: 5-baud init at address `0x34`, then ~9600 or 10400 baud blocks.

Fallback: **KWP2000 / ISO 14230** if the sniff shows that.

The K-line sniff in [TEST_PLAN.md](TEST_PLAN.md) decides. Do not implement both responders "just in case" as the first cut; implement the one the capture shows.

Physical: L9637D on `KLINE_DIAG`. SSP 243: this is not the E281 K-wire. ESP32-S3 UART; 5-baud init on a GPIO interrupt.

## Coexistence

BLE app and VCDS are separate FreeRTOS tasks. VCDS output tests and basic settings take priority and pause app commands.

## Emulate (after capture)

| Item | Content | Source | Status |
|------|---------|--------|--------|
| ID string | `4Z7 907 553 B` plus Hella/component text and SW version (S025/S031) | Copy **byte-exact** from the capture | unverified |
| Coding / WSC | Lee's current coding value and workshop code | Capture | unverified |
| Fault codes | Stored DTCs plus elaboration, including 65535 (internal/EEPROM). Map board faults (sensor open, valve short) to the original codes | Capture + Ross-Tech wiki | unverified |
| Measuring blocks | Height sensors, pressure, compressor temperature, inputs, valve states | Capture every group 000–255; scale against known heights | unverified |
| Adaptation channels | Height offsets per corner and any level channels | Capture | unverified |
| Basic settings | Height calibration (post-alignment level procedure) | Capture + ELSA procedure | unverified |
| Output tests | Valves and compressor sequence | Capture | unverified |

Address word **34** is confirmed (SSP 243). The rest is not.

## Libraries (tester-side)

These talk **to** a module. They are not a module.

| Repo | Licence | Use |
|------|---------|-----|
| [domnulvlad/KLineKWP1281Lib_ESP32](https://github.com/domnulvlad/KLineKWP1281Lib_ESP32) | GPL-3.0 | ESP32 tester |
| [domnulvlad/KLineKWP1281Lib](https://github.com/domnulvlad/KLineKWP1281Lib) | GPL-3.0 | AVR tester |
| [muki01/OBD2_KLine_Library](https://github.com/muki01/OBD2_KLine_Library) | GPL-3.0 | KW1281 + KWP2000 tester |

Reuse from them: block framing, block counter, complement-ACK, measuring-block formula tables.

**The module-side responder must be new.** No open-source ECU-side KWP1281 implementation was found in the v0.1 pass. Write a responder state machine. Licence: GPL-3.0, same as OAS-Man.

Do not vendor a tester library and "flip it around." Init, timing, and which side sends ACK are not symmetric.

## Capture required before coding the responder

On the working module, before any swap:

1. VCDS auto-scan (whole car, and address 34 alone).
2. ID + coding + WSC, saved as text.
3. Fault list, including a 65535 if one is stored.
4. Every measuring-block group that exists (walk 000–255; empty groups noted).
5. Every adaptation channel.
6. Screenshots of basic settings and output tests (do not run output tests on the car without a plan).
7. Logic-analyser or second-ESP32 sniff of the K-line during "open address 34": 5-baud init, sync, key bytes, first blocks.

Store captures under `docs/allroad/captures/` (directory not created until files exist).

## Safety while emulating

- Output tests move real valves. Gate them the same as a stock basic-setting session: stationary, app paused, explicit enable.
- A failed K-line parse must not leave a valve open.
- If VCDS coding is written, persist it in FRAM A/B, not in RAM only.
