# Open questions

Unknown stays unknown. This list is the backlog for Lee and for the next agent.

## Blocking v0.2

1. **ELSA / Bentley current-flow for J197.** Full module pinout. Not in this repo.
2. **VAG housing numbers** for J1 (small black) and J2 (large natural). Photograph the moulded `973` number.
3. **J1-9 conflict.** Forum: N111 on J1 4 and 9. Forum: pin 9 = road speed, white/blue. Meter. See [PINOUT.md](PINOUT.md).
4. **E281 protocol.** Baud, framing, LED bits. Trace while pressing raise/lower. SSP 243 diagram 243_013 does not map functions to `2/01`–`2/50`.
5. **CAN messages J197 sends and needs.** Listen-only log, engine running and driving.

## Pinout and connectors

- J2 cavity count: SSP uses `2/50`. A search hit said 52. No primary source for 52.
- J1 cavity count (at least 9).
- Terminal series (1.5 mm vs MQS 0.63 vs mixed).
- Which of `2/01 2/02 2/04 2/05 2/25 2/49 2/50` is K-wire, raise backup, KL58s, LED, or other.
- Module pins for KL30, KL15, GND, CAN H/L, diagnostic K, G291, G290, N148–N151, N311, N111, J403, K134, J431 power, door, CL pulse, KL50.
- Whether trailer / speed / E281 differ on `553 B` vs the `553 A` text in SSP 243.
- T10bb pinout except T10bb-10 as **likely** 5 V to G291 (quattroworld 2016).

## Firmware / protocol

- KWP1281 vs KWP2000 on address 34. Sniff.
- Byte-exact ID string, coding, WSC, measuring blocks, adaptations, basic settings, output-test order.
- Stock mapping of fault 65535 and other DTCs.
- Speed-lowering thresholds on Lee's B unit (SSP 243 numbers are the starting hypothesis).
- PWM frequency and duty range of the height sensors on this car.
- G290 transfer function.
- G291 volts-to-bar.
- What J197 puts on CAN for height, compressor, and faults (other ECUs may expect it).
- Door-wake and central-lock pulse polarity.
- Lifting-platform detection as implemented on `553 B`.

## Hardware

- Exact PCB-mount story for v0.2 (desoldered header footprint).
- J431 power-switch part and current.
- Enclosure that fits behind the toolbox cover with a pigtail.
- Coil ohms (empty table in [TEST_PLAN.md](TEST_PLAN.md)).
- Whether the stock internals are in fact ST10F269 + 24C16 (one source, unverified). Opening a dead module would settle it.

## Process

- Where Lee will drop captures (`docs/allroad/captures/` is the intended folder; not created until files exist).
- Whether the installed unit is the H04 (17.08.00) or the H05 (27.03.01). Lee's current belief: H04 installed, H05 spare.

## Not questions

These are decided. See [DECISIONS.md](DECISIONS.md).

- Pigtail first
- ESP32-S3
- Listen-only CAN
- Two K-lines
- FRAM A/B + CRC
- BLE app + VCDS
- Unknown pins are not guessed
