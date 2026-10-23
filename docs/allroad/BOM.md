# Draft BOM

Design v0.1. Quantities are for one controller board. No LCSC line for the J197 housings: those numbers are unknown.

Reference BOMs (do not treat as this board's pick list):

- [../../hardware/allroad-j197/reference/OpenHaldexC6_BOM.csv](../../hardware/allroad-j197/reference/OpenHaldexC6_BOM.csv)
- OAS-Man V4.0.1 schematic PDF in the same `reference/` folder

## Controller board

| Block | Part | Qty | Note |
|-------|------|-----|------|
| MCU | Espressif ESP32-S3-WROOM-1-N8R2 | 1 | Replaces ESP32-WROOM-32 on OAS-Man |
| CAN | NXP TJA1051T/3 or TI TCAN1462-Q1 | 1 | + PESD2CAN. Split 60 Ω termination unpopulated |
| K-line | ST L9637D013TR | 2 | Diagnostics + E281 K-wire |
| Low-side | ST VNL5050S5-E | 8 | N148–N151, N111, N311, J403 coil, K134 |
| Reverse / ideal diode | TI LM74700QDBVRQ1 + AGM30P14MBP or IPD-type FET | 1 | OpenHaldex uses AGM30P14MBP |
| TVS | Littelfuse SMDJ33CA | 1 | Load-dump clamp ahead of the buck |
| Buck 5 V | TI LM5164-Q1 | 1 | Input rated to 100 V |
| 3.3 V | TI TLV76733-Q1 or AMS1117-3.3 | 1 | OpenHaldex uses TPS62933 + AMS1117 |
| Sensor 5 V | Infineon TPS7B4253-Q1 tracking LDO | 2 | RH sensors, G291 |
| PWM inputs | TI SN74LVC1G17 + 10 k / 4.7 V clamp | 4 | Same clamp family as OpenHaldex BZT52C4V7 |
| FRAM | Infineon FM24CL16B-GTR | 1 | 2 KB, 10^14 writes |
| Board connector | TE AMPSEAL 35-pin 776164-1 or Deutsch DT | 1–2 | Pigtail side |
| Mating plugs | Donor J197 plugs (VAG no. unknown) | 2 | Salvage. Photograph the moulded number before cutting |

Passives, ESD, and USB follow OpenHaldex-C6 / OAS-Man V4 once the schematic is more than net labels. Do not copy the OpenHaldex ESP32-C6 or dual TCAN1044 fitment onto this board.

## Off-board (stock, not purchased)

Valve block T10bb, compressor V66, relay J403, sensors G76/G77/G78/G289, G290, G291, E281, lamp K134, J431. The replacement drives these. It does not replace them.

## Datasheets

- [ESP32-S3-WROOM-1](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf)
- [L9637](https://www.st.com/en/automotive-analog-and-power/l9637.html)
- [VNL5050S5-E](https://www.st.com/en/automotive-analog-and-power/vnl5050s5-e.html)
- [LM74700-Q1](https://www.ti.com/product/LM74700-Q1)
- [FM24CL16B](https://www.infineon.com/cms/en/product/memories/f-ram-ferroelectric-ram/)
- [TJA1051](https://www.nxp.com/products/TJA1051)

## Not yet chosen

- Enclosure (IP67, vent membrane)
- Conformal coat (HumiSeal 1B31 or equal)
- J431 power-switch part
- Exact P-FET if AGM30P14MBP is unavailable
- Wire gauge and seal list for the pigtail
