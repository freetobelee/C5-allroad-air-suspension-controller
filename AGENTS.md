# Agents

Conventions for humans and AI coding agents on Lee Welsch's C5 allroad J197 replacement fork.

This file is the fork guide. The upstream OAS-Man specialist roster is [docs/AGENTS.md](docs/AGENTS.md). Agent prompt files live under `.claude/agents/`.

## Project

Replacement air-suspension ECU for a 2001 Audi C5 allroad (VIN `WAUYP64B31N165751`). Target module: J197, `4Z7 907 553 B`, VCDS address 34.

The repo still contains the full OAS-Man firmware. Until an allroad env exists, treat that tree as a library to read, not a product to ship on this car.

Owner: **Lee**. Do not refer to Lee with pronouns.

## Where things live

```
docs/allroad/                 project docs (start here)
docs/allroad/sources/         SSP 243 + module label photo
docs/UPSTREAM_OASMAN_README.md
hardware/allroad-j197/        KiCad skeleton, nets.csv, LICENSES.md
hardware/allroad-j197/reference/   OAS-Man V4.0.1 + OpenHaldex-C6 copies
OASMan_ESP32/                 upstream manifold firmware
Wireless_Controller/          upstream LVGL controller
MobileApp/oasman_mobile/      upstream Flutter app
ESP32_SHARED_LIBS/            shared protocol + user_defines.h
CLAUDE.md                     agent hard rules
```

## Conventions

- Terse, plain prose. No "honestly", "genuinely", "delve", "seamless", or hype.
- Cite sources. Confidence tags: **confirmed**, **likely**, **unknown**, **conflict**.
- **Unknown pins stay marked unknown until measured.** No guessed pin numbers on the schematic, in `nets.csv`, or in firmware `#define`s.
- Do not invent VAG housing numbers, VCDS ID strings, coding, measuring-block layouts, or E281 baud/framing.
- J197 net names in `hardware/allroad-j197/nets.csv` are the board-side names. Harness pins are assigned only after a meter check.
- Corner order in OAS-Man code is FP, RP, FD, RD. VAG valves are N148 FL, N149 FR, N150 RL, N151 RR. Write the mapping down before driving a valve.
- Firmware derived from OAS-Man stays GPL-3.0. Copied OpenHaldex-C6 schematic blocks keep MIT notices.
- Refer to Lee by name.

## Confidence rules (pinout and protocol)

| Tag | Meaning | Allowed action |
|-----|---------|----------------|
| confirmed | Primary source + consistent (SSP text, or Lee measured it) | May go on the schematic and into firmware |
| likely | Secondary source (forum, diagram adjacency) | May appear in docs and as a KiCad note. Not a `#define`. |
| unknown | No pin number | Leave blank. Do not pick a number. |
| conflict | Two sources disagree | Do not pick a winner. Measure. |

## Specialist roster (when work leaves docs)

Use the existing agents. Facts in a specialty stay with that agent.

| Agent | Use on this fork for |
|-------|----------------------|
| project-manager | Session start, sequencing, what is blocking |
| hardware-pcb-expert | KiCad, BOM, drivers, power, connectors, enclosure |
| air-suspension-expert | Height loop, raise/lower order, speed lowering, safe states |
| esp32-expert | S3 bring-up, TWAI, UART/K-line, PWM capture, FRAM I2C |
| ble-expert | OAS-Man BLE app path; closest owner for K-line framing until a VAG agent exists |
| mobile-app-expert | Phone app as everyday UI |
| librarian | Constants, SSP page cites, doc/code mismatch |
| devils-advocate | Before locking a new approach, especially valves or a guessed pin |
| qa-engineer | Before commit, flash, or any claim of "pinout done" |

Startup checklist (project-manager):

1. What milestone? (now: docs v0.1 / measure pinout)
2. Last confirmed working state? (stock J197 in the car)
3. Goal this session?
4. Acceptance test?
5. Rollback? (keep the working J197)

## Do not

- Assign J1/J2 pins from memory or from an AI training prior.
- Treat a 52-pin search hit as a pin count. No primary source.
- Enable compressor or valve outputs on the car before bench tests in [docs/allroad/TEST_PLAN.md](docs/allroad/TEST_PLAN.md).
- Emulate VCDS ID/coding/blocks from Ross-Tech wiki alone. Capture the working module first.
- Change `BTOas.h` on one client only.
- Brick-path OTA changes without a bootable fail path.
