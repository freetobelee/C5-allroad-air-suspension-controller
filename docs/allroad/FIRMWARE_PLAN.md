# Firmware plan

No allroad firmware exists yet. Do not change the OAS-Man tree in a docs-only task.

Suggested first env, when code starts: `allroad_j197_dev` in a new project dir or a new PlatformIO env. Do not overload `manifold_v4_dev` (WROOM + Bluepad32). The board is an ESP32-S3.

## What to reuse from OAS-Man

| Piece | Path | Use |
|-------|------|-----|
| Solenoid / output pulse | `OASMan_ESP32/src/components/` | Drive VNL5050s. One output per J197 load, not IN+OUT per corner. |
| Compressor object | `compressor.cpp` | Duty, overheat. Feed it G290 and a pressure reading, not a tank transducer on an ADS1115. |
| Manifold / wheel | `manifold.cpp`, `wheel.cpp` | Height-mode path (`getheightSensorMode`, `readLevelSensorNormalized`). Today height is analogue 0–100. Replace the reader with PWM duty. |
| BLE + `BTOasPacket` | `ESP32_SHARED_LIBS/src/BTOas.h`, `OASMan_ESP32/src/bluetooth/` | Everyday UI. Packet change still updates firmware + controller + app together. |
| Phone app | `MobileApp/oasman_mobile/` | Lee's daily control (decision 23 Oct 2026). |
| Profiles | `MAX_PROFILE_COUNT` | Map four profiles to LL / NL / HL1 / HL2, not free-form PSI. |
| Safe-state habits | `user_defines.h` | Keep `MAX_PRESSURE_SAFETY`. Add a height ceiling. Valves closed on power loss. |

`SIX_VALVE_MANIFOLD` is a shared chamber with tank/exhaust valves. J197 is four strut valves + N311 accumulator + N111 discharge + J403 relay. Do not set `SIX_VALVE_MANIFOLD true` and call it done. Write a J197 valve object.

## What to write new

1. PWM height capture (four channels).
2. J197 levelling state machine (levels, axle order, speed rules, trailer, parking, run-on/sleep).
3. TWAI CAN, listen-only first.
4. E281 K-wire client (dash buttons / LEDs).
5. Diagnostic K-line **module-side** responder (address 34). See [DIAGNOSTICS_VCDS.md](DIAGNOSTICS_VCDS.md).
6. FRAM A/B + CRC.
7. KL30 brownout → drivers off.
8. Board pin map (`BOARD_VERSION_ALLROAD_J197` or equal). Empty until pins are confirmed.

## Height-sensor PWM levelling

Sensors are inductive angle senders. Pin 6 is a PWM duty proportional to arm angle (SSP 243). Left and right arms move opposite: compression raises duty on one side and lowers it on the other. Calibrate per corner; store invert flags.

OAS-Man `analogToHeightPercentage` is the wrong front end. Use PCNT / MCPWM / `pulseIn` / RMT to get duty. Reject a missing pulse (open wire) as a fault; do not hold the last height forever.

Height is the control truth. G291 is plausibility and accumulator logic, not the ride-height loop.

## Levels (SSP 243)

Ground clearance:

| Level | Name | Clearance | Offset from normal |
|-------|------|-----------|--------------------|
| 1 | Low (LL) | 142 mm | −25 mm |
| 2 | Normal (NL) | 167 mm | 0 |
| 3 | High 1 (HL1) | 192 mm | +25 mm. Also parking level |
| 4 | High 2 (HL2) | 208 mm | +41 mm |

E281: one raise/lower step per press. Repeat raise can skip (e.g. LL → HL1). HL2 only after HL1 is set. Repeat lower can drop HL2 → LL in three presses.

These millimetres are **targets after calibration**, not raw PWM. Basic setting (VCDS) writes the offsets into FRAM.

## Raise rear first, lower front first

SSP 243, restated in the control-concepts section:

- Raising: rear axle, then front axle.
- Lowering: front axle, then rear axle.
- Left/right on an axle are equalised (one-sided load).

Do not open all four strut valves together for a level change. Do not dump all four corners as a default.

## Speed lowering

SSP 243 (confirm on Lee's `553 B` before treating thresholds as final):

| Condition | Action |
|-----------|--------|
| Speed > 35 km/h in HL2 | Lower to HL1 |
| HL2 request only if speed < 30 km/h | Ignore otherwise |
| Speed > 80 km/h in HL1 | Lower to normal |
| HL1 request only if speed < 75 km/h | Ignore otherwise |
| No auto-raise to HL1/HL2 while driving | Except parking-level raise after lock |
| Rise back toward normal | <70 km/h for >120 s; <35 km/h for >30 s; <5 km/h immediately |

Speed comes from the hardwired square wave and from CAN. Until both are verified, require agreement or fall back to the safer (lower) request. Trailer (F216 / `TRAILER_IN`): force manual mode, prefer normal, no motorway auto-lower without a measured stock trace.

Reaction times (SSP 243): height mode ~5 s below 5 km/h (~1 s if extremely low); driving mode ~50 s or 15 min above 10 km/h. Run-on up to 15 min on KL30 after KL15 off. Sleep after that. Wake on door, then KL15, then speed. Cap door-wake cycles (SSP: 15, then 1 min, then KL15/speed only).

Lifting-platform mode: detect a stationary, all-corners-up event and stop discharging. Implementation after a capture, not from memory.

## CAN

Drive CAN. Transceiver as in [HARDWARE.md](HARDWARE.md).

Phase 1: listen-only. Log IDs while driving. Phase 2: emit the messages a stock J197 sends, once a capture exists. Do not invent IDs.

SSP 243 lists J197 as a CAN node exchanging with other drivetrain units. Exact IDs are **unknown**.

## Dash-button serial (E281)

Separate K-wire from diagnostics (SSP 243). E281 encodes raise/lower/mode and drives LEDs from replies.

Protocol, baud, and framing: **unknown**. Trace it ([TEST_PLAN.md](TEST_PLAN.md)). Raise also has a redundant hardwired backup (`E281_RAISE`).

Until the serial is decoded: accept BLE app commands, and treat `E281_RAISE` as a digital input if that pin is confirmed. Do not guess LED bitfields.

## FRAM

FM24CL16B, I2C.

Layout (draft; change freely until first write ships):

```
slot A: magic | seq | payload | crc32
slot B: magic | seq | payload | crc32
```

Payload: per-corner height min/max and invert, level offsets, coding, last selected level, odometer-class counters if needed.

Boot: pick the slot with a good CRC and the higher `seq`. If both fail, run with factory defaults, light K134, set an internal fault (map to 65535 only if the VCDS capture shows that as the stock "EEPROM dead" code).

Write: increment `seq`, write the inactive slot, CRC last. Never update in place.

This is the countermeasure for the stock 65535 EEPROM death.

## Safe states

| Loss | Action |
|------|--------|
| KL30 brownout | All valve drivers off. Compressor off. |
| Height sensor open / implausible | Do not move that corner. Fault. K134. |
| G291 implausible | No accumulator raise. Fault. |
| G290 / temp model over limit | Compressor off. |
| CAN timeout (after TX is enabled) | Keep last safe height. No auto raise. |
| BLE loss | Ignore app. E281 and VCDS still work. Do not dump. |
| VCDS basic settings / output test | Pause app and E281 raise/lower. |

`ENABLE_AIR_OUT_ON_SHUTOFF` stays false.

## Task split (when code starts)

FreeRTOS:

- Control task: height loop, valves, compressor.
- BLE task: existing OAS-Man server.
- CAN task: TWAI RX (and later TX).
- K-line diag task: address 34 responder.
- E281 task: dash serial.
- Do not call `lv_*` from any of these (only relevant if a Waveshare UI is reused).

VCDS output tests and basic settings outrank the app.

## Corner map (write this into code comments before the first flash)

OAS-Man order: **FP, RP, FD, RD**.

VAG: N148 FL, N149 FR, N150 RL, N151 RR.

LHD (Lee's car): FL=FD, FR=FP, RL=RD, RR=RP.

Confirm with a meter on T10bb colour dots (SSP 243: valve-block colours match the lines) before driving a solenoid.
