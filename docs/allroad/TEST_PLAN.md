# Test plan

Work for Lee, on the car or the bench, before schematic pins or firmware maps are filled in.

Keep the working J197 installed until this list has a CAN log, a VCDS dump, and a meter pin list. The spare (H05, 27.03.01) can come out for photos and coil checks.

Do not invent results. Write measured values back into [PINOUT.md](PINOUT.md) with date and instrument.

## 1. Read the housings

Photograph both module plugs face-on and the moulded VAG number (usually `4B0` / `8D0` / `1J0 973 xxx`).

Count cavities and which cavities have terminals.

Same for T10bb at the valve block and one height-sensor plug.

Outcome: pin counts and housing numbers, or an explicit "no number moulded."

## 2. Multimeter pin verification

KL15 off, then KL15 on. Do not probe CAN or K-line with a low-ohm beep across the pair.

Find, at the module plugs:

- `KL30` (always live vs chassis)
- `KL15` (live with ignition)
- `GND` (continuity to chassis)

Continuity (module unplugged, harness side) to:

- Height sensors pins 1 / 5 / 6 (right-hand sensors should land on this module; left-hand on J431, with `J431_PWR` as the feed)
- T10bb (valves + G291)
- E281
- Trailer F216 (expect earth when a trailer plug is inserted — SSP 243)

Outcome: a table pin → net with voltages or ohms. Promote those rows to **confirmed** in PINOUT.md.

Resolve the J1-9 conflict (N111 vs road speed) here. Do not guess.

## 3. Valve coil resistance

Measure at T10bb and at the J403 relay socket, cold, battery disconnected if that is the safe way on this car:

| Load | Expected | Measured Ω |
|------|----------|------------|
| N111 | unknown until measured | |
| N148 | unknown until measured | |
| N149 | unknown until measured | |
| N150 | unknown until measured | |
| N151 | unknown until measured | |
| N311 | unknown until measured | |
| J403 coil | unknown until measured | |

Do not copy a "typical 12 V solenoid is 10–20 Ω" figure into the firmware current limit. Write the ohms Lee reads.

SSP 243: valve-block ports are colour-coded to the lines. Photograph the colours with the pin numbers.

## 4. CAN capture

Listen-only ESP32 or CANable on the J197 CAN pins (or OBD 6/14 if those are the same drive bus — verify, do not assume).

500 kbit/s is the usual VAG drive-CAN rate; confirm by lock. Log with engine running and while driving.

Outcome: a raw log (candump / SavvyCAN). IDs J197 sends are unknown until this exists.

## 5. E281 / button-line logic trace

Logic analyser on the E281 lines while pressing raise and lower, and while the stock module drives the LEDs.

Note baud, idle level, and whether the line is UART-like or something else.

Include the candidate `2/01 2/02 2/04 2/05 2/25 2/49 2/50` set once those cavities are identified. SSP 243 diagram 243_013 does not assign functions.

Outcome: a capture file and a one-page note (baud, frames or "not UART").

## 6. VCDS baseline (address 34)

On the **working** module, before swapping anything:

- Auto-scan
- ID string and coding / WSC
- Fault codes (include 65535 if present)
- Every measuring-block group (000–255)
- Every adaptation channel
- Basic-settings and output-test screens (screenshots; do not run destructive tests)

Outcome: files for [DIAGNOSTICS_VCDS.md](DIAGNOSTICS_VCDS.md). The replacement copies these; it does not invent them.

## 7. K-line sniff

Tap the diagnostic K-line with an L9637 and a logic analyser or a second ESP32 while VCDS opens address 34.

Capture 5-baud init, sync, key bytes, and the first block exchange.

Outcome: KWP1281 vs KWP2000, baud, address byte.

## Bench (after a board exists)

Not this pass. When a board exists:

1. USB-only bring-up. No KL30.
2. KL30 through a current-limited supply. Brownout: dip the rail, confirm all drivers stay off.
3. Dummy coils (measured ohms) on each low-side. One output at a time.
4. PWM generator into the four height inputs. Confirm duty readback.
5. FRAM: write A, pull power mid-write, confirm boot uses B.
6. VCDS against the bench responder with the captured ID string.
7. Only then, pigtail in the cargo area, working J197 still available to put back.
