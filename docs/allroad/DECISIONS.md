# Decisions

Locked for design v0.1 unless Lee reverses one in writing.

| # | Decision | Why | Date |
|---|----------|-----|------|
| 1 | **Pigtail adapter first.** Donor J197 plugs (or plugs cut from a salvage module) to Deutsch DT or TE AMPSEAL 35 on the PCB. v0.2 option: desolder a dead J197 header and reuse it. | No PCB-mount housing found. Pigtail is buildable now and benchable. | 23 Oct 2026 |
| 2 | **MCU: ESP32-S3-WROOM-1-N8R2**, not OAS-Man's ESP32-WROOM-32. | GPIO, native USB, TWAI. Same Arduino-ESP32 family so OAS-Man classes port. | 23 Oct 2026 |
| 3 | **CAN: TJA1051T/3 or TCAN1462-Q1 + PESD2CAN.** Listen-only until the bus is decoded. Termination unpopulated. | OpenHaldex-C6 front end. J197 is not the sole terminator. | 23 Oct 2026 |
| 4 | **Two L9637D.** One diagnostic K-line (address 34). One E281 K-wire. | SSP 243: the two K-wires are not the same net. | 23 Oct 2026 |
| 5 | **Eight VNL5050S5-E low-sides** for N148–N151, N111, N311, J403 coil, K134. Compressor motor current stays on the stock relay. | Protected automotive low-side. Motor current is too high to switch on-board. | 23 Oct 2026 |
| 6 | **Power: LM74700-Q1 + P-FET, SMDJ33CA, LM5164-Q1 to 5 V, 3.3 V LDO, two TPS7B4253-Q1 for sensor 5 V.** ADC brownout on KL30 parks valves closed. | Reverse, load dump, and brownout are known kills. | 23 Oct 2026 |
| 7 | **Storage: FM24CL16B FRAM, A/B copies, CRC32, sequence counter.** | Countermeasure for stock EEPROM / VCDS 65535. | 23 Oct 2026 |
| 8 | **Environment: conformal coat, IP67 box, vent membrane, plugs facing down.** | Water in the cargo-area module is a known kill. | 23 Oct 2026 |
| 9 | **Licence: firmware GPL-3.0 (OAS-Man). Copied OpenHaldex-C6 blocks keep MIT notices.** | Fork obligations. | 23 Oct 2026 |
| 10 | **Two user interfaces.** OAS-Man phone app over BLE for everyday use. VCDS-compatible diagnostics at address 34 on K-line. Protocol most likely KWP1281 (unverified). VCDS tests pause the app. | Approved by Lee Welsch. | 23 Oct 2026 |
| 11 | **Unknown pins stay unknown until measured.** | A wrong valve pin on a real car is unsafe. | 23 Oct 2026 |
| 12 | **Height is the control truth.** PWM from G76/G77/G78/G289. G291 is plausibility and accumulator logic. | SSP 243. OAS-Man default is pressure; that does not apply here. | 23 Oct 2026 |
| 13 | **Level change order:** raise rear then front; lower front then rear. Targets 142 / 167 / 192 / 208 mm. | SSP 243. | 23 Oct 2026 |
| 14 | **Refer to the owner as Lee.** No pronouns for Lee in docs. | Lee. | 23 Oct 2026 |

## Rejected or deferred

| Idea | Status | Why |
|------|--------|-----|
| True drop-in PCB header in v0.1 | Deferred to v0.2 | Housing unknown |
| Transmit on drive CAN from day one | Rejected | IDs unknown; listen-only first |
| `SIX_VALVE_MANIFOLD true` as the J197 map | Rejected | Different valve set (accumulator + discharge + relay) |
| Invent VCDS ID / blocks from a wiki | Rejected | Capture the working module |
| All-corner dump on fault or key-off | Rejected | Safety. `ENABLE_AIR_OUT_ON_SHUTOFF` stays false |
| Disable `MAX_PRESSURE_SAFETY` | Rejected | Hard rule |
