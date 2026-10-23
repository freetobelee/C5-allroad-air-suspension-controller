# Pinout

**Rule:** unknown pins stay unknown until measured. Do not invent a number. Do not put a guessed pin on the schematic or in a `#define`.

Confidence:

| Tag | Meaning |
|-----|---------|
| confirmed | Primary source, or Lee measured it |
| likely | Secondary source. Docs and KiCad notes only |
| unknown | No module pin number |
| conflict | Sources disagree. Measure. Do not pick a winner |

Board net names are in [../../hardware/allroad-j197/nets.csv](../../hardware/allroad-j197/nets.csv). Those are signal names, not harness pins.

Notation `2/nn` means connector 2 (J2, large), pin `nn`, as SSP 243 writes it.

## Connectors

| Plug | Seen on module | Pin count | VAG housing no. | Terminals | Confidence | Source |
|------|----------------|-----------|-----------------|-----------|------------|--------|
| J2 (connector 2) | Large, natural/beige | 50 or more (SSP 243 uses `2/50`) | Unknown | Unknown; likely 1.5 mm or MQS 0.63 | Pin-count lower bound confirmed | SSP 243 pin labels; module photos |
| J1 (connector 1) | Small, black | Unknown; at least 9 (forum cites pin 9) | Unknown | Unknown | unknown | Forum (from the design brief) |
| T10bb (valve block) | At the valve block, not the module | 10 | Unknown | Unknown | likely | Forum, wiring-diagram mentions |
| Level sensor (each) | At each sensor | 6 (pins 2 and 3 unused) | Unknown | Unknown | confirmed | SSP 243 level-sensor table |

No ETKA or catalogue listing for the J197 mating housings was found. One search summary said "52-pin"; no primary source. Next: photograph each housing face-on and read the moulded VAG number (usually `4B0` / `8D0` / `1J0 973 xxx`).

The KiCad sheet has no connector symbols until those counts are verified.

## Confirmed

| Pin | Signal | Board net | Source |
|-----|--------|-----------|--------|
| 2/10 | Trailer signal from F216 (F216 pulls J197 to earth when the trailer plug is in) | `TRAILER_IN` | SSP 243 text ("to the self-levelling suspension control unit J197 (pin 2/10)") |
| Sensor pin 1 | Earth. Left sensors from J431; right sensors from J197 | `GND` (right-hand) | SSP 243 level-sensor table |
| Sensor pin 5 | 5 V supply. Left from J431; right from J197 | `V5_SENS_RH` (right-hand) | SSP 243 level-sensor table |
| Sensor pin 6 | PWM angle (all four sensors to J197) | `SENS_FL_PWM` / `SENS_FR_PWM` / `SENS_RL_PWM` / `SENS_RR_PWM` | SSP 243 level-sensor table |
| Sensor pin 2 | Vacant | — | SSP 243 |
| Sensor pin 3 | Vacant | — | SSP 243 |
| Sensor pin 4 | Analogue voltage, left sensors only, to headlamp range | — (not a J197 input) | SSP 243 |

Confirmed means the **sensor-end** assignment, or the one SSP sentence that names J197 pin 2/10. It does not mean the J197 cavity for the PWM wires is known.

## Likely

| Pin | Signal | Board net | Source | Caveat |
|-----|--------|-----------|--------|--------|
| 2/01, 2/02, 2/04, 2/05, 2/25, 2/49, 2/50 | E281 button-unit lines (exact function of each unknown) | `E281_*` | SSP 243 diagram 243_013 | Diagram places E281 pins 3, 1, 7, 5, 9, 10 next to those J197 pins. The text layout does not prove which wire is which. K-wire, raise backup, KL58s, and LED lines cannot be told apart without ELSA. Trailer / speed / E281 may differ on `553 B`; SSP 243 describes `553 A` in places. |
| J1 4 and J1 9 | Dump/exhaust valve N111 | `OUT_N111` | Forum (from the design brief) | Conflicts with the speed-signal claim on pin 9. |
| T10bb-10 | 5 V to pressure sensor G291 | `V5_PRESS` | quattroworld 2016 | Valve-block end, not a J197 pin. |

## Conflict

| Pin | Claim A | Claim B | Board net | Action |
|-----|---------|---------|-----------|--------|
| 9 (plug not stated) / J1-9 | N111 dump/exhaust (forum, "J1 4 and 9") | Road speed, white/blue (forum) | `OUT_N111` vs `SPEED_IN` | Do not assign. Meter both. SSP 243 says the speed signal is a hardwired square wave from the dash (also on CAN). It does not give a J197 pin. |

## Unknown (signals exist; module pin does not)

These nets are on the KiCad skeleton. **No J197 pin number is known.** Do not fill the pin column.

| Signal | Board net | Why it exists | J197 pin |
|--------|-----------|---------------|----------|
| Battery | `KL30` | SSP 243 functional diagram | unknown |
| Ignition | `KL15` | SSP 243 | unknown |
| Ground | `GND` | SSP 243 | unknown |
| G291 pressure (analogue) | `PRESS_G291` | SSP 243 (voltage proportional to pressure) | unknown |
| G291 5 V | `V5_PRESS` | Needed to feed G291; T10bb-10 is likely the sensor-end 5 V | unknown at J197 |
| G290 compressor temperature | `TEMP_G290` | SSP 243 | unknown |
| Door contact | `DOOR_IN` | SSP 243 auxiliary signals (wake) | unknown |
| Central-lock / vehicle-locked pulse | `CL_PULSE` | SSP 243 | unknown |
| Starter (KL50) | `KL50_IN` | SSP 243 | unknown |
| Road speed (hardwired) | `SPEED_IN` | SSP 243; also on CAN | unknown (see conflict) |
| Drive CAN H / L | `CAN_H`, `CAN_L` | SSP 243 | unknown |
| Diagnostic K-line | `KLINE_DIAG` | SSP 243; address 34 | unknown |
| E281 K-wire | `E281_KWIRE` | SSP 243; separate from diagnostic K | unknown (among the 2/xx set) |
| E281 raise backup | `E281_RAISE` | SSP 243: raise button is a redundant extra interface | unknown |
| J403 compressor relay coil | `OUT_J403` | SSP 243 | unknown |
| N111 discharge | `OUT_N111` | SSP 243 | unknown (see conflict) |
| N148 FL | `OUT_N148` | SSP 243 | unknown |
| N149 FR | `OUT_N149` | SSP 243 | unknown |
| N150 RL | `OUT_N150` | SSP 243 | unknown |
| N151 RR | `OUT_N151` | SSP 243 | unknown |
| N311 accumulator | `OUT_N311` | SSP 243 | unknown |
| K134 warning lamp | `OUT_K134` | SSP 243 | unknown |
| Power feed to J431 (HRC / left sensors) | `J431_PWR` | SSP 243 | unknown |

Also present in SSP 243 and **not** yet given a board net: Terminal 56, Terminal 58s / 58d (E281 illumination), ESP-button lines, headlamp-range "level change" signal to J431. Add nets only when a measurement or ELSA page names them.

## How to promote a pin

1. Lee photographs the housing and counts cavities.
2. Continuity or voltage is measured with KL15 off and on (see [TEST_PLAN.md](TEST_PLAN.md)).
3. Result is written here with instrument, date, and photo/log name.
4. Tag becomes **confirmed**.
5. Then, and only then, the pin may be added to the KiCad connector symbol and to firmware maps.

Forum or SSP adjacency moves a pin to **likely**, not to confirmed.
