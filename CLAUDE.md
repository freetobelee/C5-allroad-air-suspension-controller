# CLAUDE.md

Guidance for an AI coding agent working in this repository.

## What this project is

Lee Welsch's fork of **OAS-Man** (GPL-3.0), turned into the documentation home for a drop-in replacement of the 2001 Audi C5 allroad air suspension control module:

- Module: **J197**, part `4Z7 907 553 B`, Continental/Hella `5SG 008 352-03`
- VCDS address **34**
- Car: VIN `WAUYP64B31N165751`, about 243k miles
- Two `553 B` units: H04/S025 (17.08.00, thought installed) and H05/S031 (27.03.01, spare)

The firmware will control a real, occupied, possibly moving vehicle. Conservative by default. Safety first. Reversible steps.

**This pass is docs and scaffolding only.** Do not change OAS-Man firmware, BLE packets, or OTA workers unless a later task says so.

Owner: refer to the owner as **Lee**. Do not use pronouns for Lee.

## Where things live

| Path | What |
|------|------|
| `docs/allroad/` | Project docs. Start here. |
| `docs/allroad/sources/ssp243.pdf` | Audi SSP 243 (4-level air suspension) |
| `docs/allroad/sources/module-label.jpg` | Spare module label (H05, 27.03.01) |
| `hardware/allroad-j197/` | KiCad skeleton, `nets.csv`, licences |
| `hardware/allroad-j197/reference/` | Unmodified OAS-Man V4.0.1 and OpenHaldex-C6 copies |
| `docs/UPSTREAM_OASMAN_README.md` | Original OAS-Man README |
| `OASMan_ESP32/` | Upstream manifold firmware (ESP32-WROOM). Unchanged. |
| `Wireless_Controller/` | Upstream LVGL controller (ESP32-S3). Unchanged. |
| `MobileApp/oasman_mobile/` | Upstream Flutter BLE app. Unchanged. |
| `ESP32_SHARED_LIBS/` | Shared protocol, `user_defines.h`, OTA. A change here rebuilds both firmwares. |
| `docs/AGENTS.md` | Upstream OAS-Man agent roster (still valid for that tree) |
| `AGENTS.md` | This fork's conventions |

## Status (do not overstate)

Design **v0.1**. Pinout is mostly unknown. The KiCad sheet has 30 global net labels and no harness pin numbers. Connector symbols wait on a counted housing.

## Hard rules

1. **Unknown pins stay unknown until measured.** Never invent a J197 / J1 / J2 / T10bb pin number, a VAG housing number, a VCDS ID string, a coding value, or a measuring-block formula. Mark confidence: confirmed / likely / unknown. Cite the source. If there is no source, it is unknown.
2. **No firmware in a docs-only task.** Leave `OASMan_ESP32/`, `Wireless_Controller/`, `MobileApp/`, and `ESP32_SHARED_LIBS/` alone unless the task asks for code.
3. **Name the build env** if you do build. Manifold and controller are different chips and stacks. "It builds" without an env is not evidence. Allroad firmware, when it exists, needs its own named env (suggested: `allroad_j197_dev`).
4. **The BLE protocol never moves on one side alone.** `BTOas.h` + controller + Flutter app together.
5. **`ESP32_SHARED_LIBS` changes rebuild both firmwares.**
6. **Safety limits stay.** Do not exceed or disable `MAX_PRESSURE_SAFETY`. No all-corner-dump default. Define the safe state on BLE / power / sensor / CAN / K-line loss. Park valves closed on brownout. Raise rear first; lower front first (SSP 243). Height targets are 142 / 167 / 192 / 208 mm ground clearance.
7. **A failed OTA must never brick a device.**
8. **Board rev ↔ firmware define must match.**
9. **LVGL is single-threaded** (`LV_OS_NONE`). Do not call `lv_*` from a BLE callback.
10. **Read before write.** Preserve prior values as a `; was:` comment.
11. **Licence.** OAS-Man-derived firmware stays GPL-3.0. Copied OpenHaldex-C6 blocks keep the MIT notice.
12. **Tone.** Terse and plain. Do not use "honestly", "genuinely", "delve", or "seamless". Do not hype. Refer to Lee by name, not by pronoun.

## OAS-Man facts still in this tree

Three programs, one wire protocol:

- Manifold (`OASMan_ESP32/`): BLE peripheral. 8 solenoids, 4 pressure sensors via ADS1115, compressor, SSD1306. ESP32-WROOM, `espressif32@6.10.0` + forked Bluepad32.
- Wireless Controller and Mobile App: BLE clients.
- Contract: `BTOasPacket` (104 bytes) over STATUS / REST / VALVECONTROL.

J197 work will reuse solenoid / compressor / manifold / BLE classes on an **ESP32-S3**, and will add PWM height capture, VAG CAN (TWAI), two K-lines, and FRAM. OAS-Man levels by pressure. J197 levels by height-sensor PWM. Expect new modules around the core, not a drop-in of `SIX_VALVE_MANIFOLD` without a pinout.

Load-bearing OAS-Man constants (quoted from code; do not paraphrase):

- BLE service UUID: `679425c8-d3b4-4491-9eb2-3e3d15b625f0`
- STATUS `66fda100-8972-4ec7-971c-3fd30b3072ac`, REST `f573f13f-b38e-415e-b8f0-59a6a19a4e02`, VALVECONTROL `e225a15a-e816-4e9d-99b7-c384f91f273b`
- `BTOasPacket` = 104 bytes (`uint16_t cmd`, `uint8_t sender`, `uint8_t recipient`, `uint8_t args[100]`); MTU 517; auth within 5 s
- Corner order: **FP, RP, FD, RD** · `MAX_PRESSURE_SAFETY 200`

J197 corner valves in VAG names: N148 FL, N149 FR, N150 RL, N151 RR. Map those to FP/RP/FD/RD explicitly when code is written (LHD: FL=FD, FR=FP, RL=RD, RR=RP).

## Build (upstream tree only, until an allroad env exists)

```bash
# Manifold (ESP32-WROOM). Dev envs: manifold_v2_dev / v3_dev / v4_dev
cd OASMan_ESP32 && pio run -e manifold_v4_dev

# Controller (ESP32-S3). Dev envs: controller_{ws2p8,ws2p8b,ws3p5,ws3p5b,ws1p8knob}_dev
cd Wireless_Controller && pio run -e controller_ws2p8_dev
```

## Agent team

Roster: `AGENTS.md` (this fork) and `docs/AGENTS.md` (upstream OAS-Man specialists under `.claude/agents/`).

Start non-trivial sessions with **project-manager**. Challenge new approaches with **devils-advocate**. Gate commits with **qa-engineer**. For allroad hardware/pinout, **hardware-pcb-expert** plus **librarian**. For levelling, **air-suspension-expert**. For K-line/CAN, **ble-expert** is the closest existing specialist until a VAG-bus agent exists.

## Sources of truth (order)

1. A measurement Lee recorded (meter, capture, photo of a moulded number)
2. `docs/allroad/` in this repo
3. SSP 243 in `docs/allroad/sources/ssp243.pdf`
4. The working module, via VCDS or a K-line sniff
5. ELSA/Bentley current-flow (not in repo yet)
6. Forum posts — hints, never ground truth

If docs and a measurement disagree, the measurement wins and the doc is updated.
