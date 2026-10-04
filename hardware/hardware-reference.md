# Radxa Cubie A7S - Hardware Reference (what is physically on the board)

**Board:** RS504 | **Schematic:** V1.10 (2026-02-03) | **SoC:** Allwinner **A733** (`A733MX_HN3`, sun60iw2p1)

**Abbreviations:** **DS** = A733 datasheet (V0.93) | **UM** = A733 User Manual (V0.92) | **DT/DTS** = device tree | **OTP** = one-time-programmable defaults (PMIC) | **abs-max** = absolute maximum rating (DS Table 5-1). Full list in [Sources](#sources).

---

## 0. Purpose, scope, source rules

**Purpose:** a truthful overview of the hardware that is **physically present on the board**.
**Scope:** the **A7S board family** - variants with **4, 6, and 8 GB of RAM**. The goal is a single kernel
that runs on **all** variants, not on one specimen.

### Source hierarchy (binding) - **split by domain**

The two rank-1 sources do **not cover the same set**. Confuse them and you check a rail against a table
it **cannot** even appear in:

| Domain | Authoritative source | Note |
|---|---|---|
| **SoC pins & SoC rails** | **A733 datasheet** (incl. **absolute maximums, Table 5-1** -> Sec. 4.5) | the only source with hard limits |
| **Board nets, population, straps** | **Schematic V1.10** | the only source |
| **DRAM-die rails** (VDD2L, VDD2H, VDDQ, VDD1) | Schematic + **LPDDR5 datasheets (Micron/Rayson)** | nominal values spec-backed (below); only the *exact* abs-max per speed bin are missing |
| Population evidence | board chip photos | rank 2 |
| Running board | **aid only** | confirms presence, **never defines truth** |

> **DRAM-die-rail sources:**
> - **DRAM-die rails** (`VDD1`/`VDD2H`/`VDD2L`/`VDDQ`): spec-backed via **Micron + Rayson LPDDR5
> datasheets** - **JEDEC nominals**: VDD1 = 1.8 V, VDD2H = 1.05 V, **VDD2L = 0.9 V**, VDDQ = 0.5 V.
> These match the net-name targets. **Not "identical for every device":** per both datasheets,
> **VDD2L = VDD2H** is allowed and **VDDQ = 0.3 V** (ODT off) - the NOM values are standard, the configuration is not.
> - **AXP318 register tables** (left blank as "custom" in the V0.1 draft): backed twice, via the
> **kernel driver** (`a733-rfc`) **and** the **final AXP323 V1.1** datasheet.
> - **Allwinner STD reference schematic** (V1.0, 2025-03-28): Allwinner's original rail intent.
>
> **Still missing:** an A733 datasheet newer than V0.93 (none public), the AXP318 datasheet with register
> tables as PDF (exists nowhere, including from Allwinner), and the DRAM **manufacturer** (`AP43274250000` = 0 hits).

> **A running board does NOT show the real hardware state.**
> It shows what firmware and kernel make of it. Two examples from this very project:
> - A DT node is `okay` but no driver binds -> the device looks "absent" (eMMC, Sec. 7.6).
> - A governor is set to `performance` -> that is a **runtime decision**, not a property of the hardware.
>
> **Rule: never infer hardware from an observed runtime state.**
>
> **Purpose of this document:** a **reference baseline** for bring-up, to make faults easier to understand.
> It is **not a source of truth for the kernel** and not a document to keep in sync with software.
> **Software does not belong here** - carry lists, governor, driver binding, and open bugs live at the
> software level.
>
> **The board does not change; the kernel does. Only the former is documented here.**

### Provenance - what is actually backed?

An honest answer to *"is this all schematic, or read off the kernel?"*:

| Area | Backed by | Status |
|---|---|---|
| Population, connectors, straps, pin-mux, GPIO assignment, rail **topology** (who feeds whom) | **Schematic** | complete |
| Chip markings, 26/25 MHz crystals | **Photos** | |
| SoC capabilities, **abs-max**, pin types, IO supplies, thermal sensors | **Datasheet / UM** | |
| **PLL reference 24 MHz** | **UM** (+ BSP + clock tree as cross-check) | |
| Current budgets, inductor ratings | **Schematic p.4/p.8** | |
| Rail **voltages** | **22 of 23** from schematic/DS (**DS Table 5-2, typ column**). **1** kernel gap: DCDC9. | **Sec. 4.2, "Source" column** |
| DRAM-**die** rails (VDD2L/VDD2H/VDDQ/VDD1) | Schematic + **LPDDR5 DS** (nominals). Exact abs-max per bin missing. | (nominals) |
| OPP tables, governor, driver binding | **Kernel/DT** | **not hardware** -> software level, not here |

> **In short:** the **structure** of this document is schematic/datasheet. Among the **voltage values**,
> **exactly one** rail has no source value: **DCDC9** (`ELDO-INPUT` is not a SoC pin) - marked **K** in
> Sec. 4.2, not ground truth. Four rails first suspected as "kernel-only" (BLDO1, DLDO1, DLDO6, RTCLDO)
> are now **datasheet-backed** (Table 5-2, see Sec. 4.0a). Plus **one rail (BLDO2)** where the OTP value
> differs from the net name - **but = Allwinner reference, resolved** (Sec. 8.1).

**Legend:** populated/wired | not brought out (present in SoC, dead on the board) | open/to be measured

---

## 1. Population - what physically sits on the board

The photos are **our own shots of the board** (rank 2 of the source hierarchy) and serve as population
evidence. **Markings are read verbatim**, not filled in from datasheets.

| Part | Type / marking | Function | SoC connection | Evidence |
|---|---|---|---|---|
| **U1** | **Allwinner A733**, marking `ALLWINNER` / `A733` / `MX-HN3` / `R5165BA 27WF2X`. ED-FCCSP, **570 balls**, 15x15 mm | SoC | - | DS p.1; **photo** |
| **UD1** | **LPDDR5**, marking `CA6` / `AP43274250000` | DRAM, 32 bit (2 channels x 16 bit) | DRAM PHY | p.5; **photo** |
| **UP1** | **AXP318W** (X-Powers), marking `AXP318W` / `R5397BA 9781` + X-Powers logo | PMIC, 9x DCDC + LDOs | **I2C `0x36`** on S-TWI0 (PL0/PL1) | p.4; **photo** |
| **U6** | **MaxIO MAE0621A-Q3C**, marking `MaxIO` / `MAE0621A-Q3C` / `AFNBUF803312` / `2443` (week 43/2024) | Gigabit Ethernet PHY | **RGMII**, bank PH (PH0-PH15) | p.15; **photo** |
| **U3** | **Quectel FCU760K**, marking `QUECTEL` / `FCU760K` / `FCU760KAAMD` / FCC ID `XMR2023FCU760K` | WiFi + Bluetooth | **USB** (hub port 1) - **not SDIO** | p.11; **photo** |
| **U9** | **WCH CH334F** | 4-port USB 2.0 hub (**2** ports used) | upstream = SoC **USB1** | p.13; photo (marking hard to read -> **schematic authoritative**) |
| **U8A** | **Samsung KLM8G1GETF-B041** (8 GB eMMC) | eMMC | **SDC2**, bank PC | p.9 - **not photographed**, population unresolved (Sec. 7.6) |
| **U7** | microSD socket `1060803AR009` | SD card | **SDC0**, bank PF; card-detect **PF6** | p.13 |
| **U4** | **HUSB311** | USB-C **PD/CC controller** | **S-TWI1** (PL12/PL13), INT = **PL3** | p.12 |
| **U5** | **BL24C16F** (2 KB) | I2C EEPROM | **TWI0** (PG8/PG9), WP via **PC7** + Q2 | p.16 |
| **U12** | **SGM40666AS** (4.5 A) | input load switch / OVLO | 5 V -> `VCC5V0_SYS` | p.13 |
| **U2, U10** | **SGM2576** x2 | VBUS switch (Type-C J4 / USB-A) | EN = **PL2** / **PM5** | p.12, p.13 |
| **UP11** | **ETA3521** + LP16 1 uH | buck ~**2.4-2.5 V** (`LDO2V5`) | input of the **BLDO/CLDO groups** | p.4 |
| **Q8** | WPM2015-3 (P-FET) | load switch `WIFI_3V3` | gate via **PM0** | p.11 |
| **X1** | **26 MHz** (16 pF, 10 ppm) - **legible on the photo: `26.000 MHz`** | SoC reference clock | DCXO | p.8; **photo** |
| **Y1** | **25 MHz** (12 pF, 10 ppm) - **legible on the photo: `25.000 MHz`** | Ethernet **PHY** clock (own crystal) | - | p.15; **photo** |
| **Y2** | **32.768 kHz** (12.5 pF, 20 ppm) | RTC | X32KIN/OUT | p.8 |
| **Y3** | **12 MHz** (20 pF, 10 ppm) | USB hub clock | - | p.13 |

### 1.1 The photos

**SoC + 26 MHz crystal** - `ALLWINNER A733 MX-HN3`. The crystal to its left legibly reads **`26.000 MHz`**;
this confirms the schematic (X1) **and** the CK-SEL strap ("SOC=26M") independently.
The earlier inventory note "24.000 MHz" was a misread.

![Close-up of the board: a large black BGA package in the center, laser-marked with the Allwinner logo, below it "A733", "MX-HN3" and the batch number "R5165BA 27WF2X". To its left, on green PCB, a gold-framed SMD crystal clearly reading "26.000 MHz", next to it a smaller crystal marked "1432A". Around it traces, vias, and decoupling capacitors; at the bottom edge an FPC connector, top-left a gold-plated mounting hole.](img/soc-a733.jpg)

**DRAM (LPDDR5)** - marking `CA6` / `AP43274250000`.

> **The marking names neither manufacturer nor capacity.** You **cannot read the 4/6/8 GB variant off the
> chip** - no more than off a running system (Sec. 3.2). To learn the size, let boot0 run its scan.

![Close-up of the DRAM package: a black BGA with a rough surface, marked with a square DataMatrix code and the two lines "CA6" and "AP43274250000". No manufacturer logo or capacity marking is present on the package. To its left, green PCB with a row of small decoupling capacitors, vias, and gold-plated test pads.](img/dram-lpddr5.jpg)

**PMIC AXP318W** - `AXP318W` / `R5397BA 9781`, X-Powers logo. Settles the naming question for good:
**AXP318W (silicon) = DT name `x-powers,axp8191`** - one part, two names, no second PMIC.

![Close-up of the PMIC: a square black package with the laser lines "AXP318W" and "R5397BA 9781" and, top-right, the X-Powers logo, a stylized double circle. Around the chip a densely populated periphery: several cube-shaped dark-gray power inductors, many ceramic capacitors, at right a row of larger black parts along the board edge, top-right a mounting hole.](img/pmic-axp318w.jpg)

**Ethernet PHY** - `MaxIO` / **`MAE0621A-Q3C`** / `AFNBUF803312` / `2443`. The **Q3C** variant is thus
photo-backed (relevant for the PHY driver). The crystal below it legibly reads **`25.000 MHz`** = Y1.

![Close-up of the Ethernet PHY: a square QFN package with peripheral pads, marked with the MaxIO logo and the lines "MAE0621A-Q3C", "AFNBUF803312" and "2443". Below the chip a gold-framed SMD crystal clearly reading "25.000 MHz". At left the edge of a shielded connector, at right several small transistors and resistor networks, above them parts with the silkscreen marking "PC-1x".](img/eth-phy-maxio-mae0621a.jpg)

**WiFi/BT module** - `QUECTEL FCU760K` / `FCU760KAAMD`, FCC ID `XMR2023FCU760K`. Shielded module,
u.FL antenna jack bottom-left. **Connected over USB**, not SDIO (Sec. 7.4).

![Close-up of the WiFi/Bluetooth module: a rectangular, silver shielded metal can, laser-marked with "QUECTEL", "FCU760K", the type number "FCU760KAAMD", a serial number, "FCC ID: XMR2023FCU760K" and the marking "Q1-C5662"; to its right a DataMatrix code, below it the CE mark. The module connects to the board through a row of solder pads along its edge. Bottom-left at the board edge a gold u.FL antenna jack, at left a row of gold-plated vias.](img/wifi-bt-quectel-fcu760k.jpg)

**USB hub** - QFN + own crystal. The marking is **not reliably legible** on the photo; the identification
as **CH334F** rests on the **schematic** (U9), not the photo.

![Close-up of the USB hub area: a small square QFN package with peripheral pads whose laser marking is blurry and not reliably decipherable in the photo - at most fragments are visible. Immediately to its right a crystal in a metal can, at left a larger black part. At the top edge two rows of gold-plated vias of a connector, all around capacitors and fine traces.](img/usb-hub-ch334f.jpg)

> **What the photos do NOT show** - stated honestly: the **eMMC** (U8A) is not photographed, its population
> stays open (Sec. 7.6). Also not pictured: connectors, headers, fan connector, button.

---

## 2. What the SoC can do - and what of it the board brings out

### 2.1 Present and usable

| Block | Silicon (datasheet) | On the board |
|---|---|---|
| **CPU** | **"Dual-core Arm Cortex-A76 and Hexa-core Arm Cortex-A55, up to 2.0 GHz"** (DS p.1, verbatim; again p.924) | **2x A76 + 6x A55** - ground truth from the DS. What the DS does **not** give: a **separate** max frequency per cluster (it states only one shared "up to 2.0 GHz"). |
| **RISC-V E902** | 1x up to 200 MHz (DS p.1) | silicon present, not exposed |
| **GPU** | IMG **BXM-4-64 MC1** | |
| **NPU** | "up to 3 TOPs" - **no UM chapter**, only advertised | present (VeriSilicon VIP9000) |
| **Video engine** | dec up to 8K@24 (H.265/VP9/AVS2), enc up to 4K@30 | silicon present |
| **ISP / DE / G2D / DI** | yes | silicon present |
| **PCIe 3.0** | **"1x PCIe3.0 DM, supporting RC and DM"** (DS p.1; **DM = dual mode** = root complex *and* endpoint). 1 lane (COMB1). | connector **J3** (x1, as RC) |
| **DisplayPort** | over **COMB0**, 4 lanes | over **USB-C J4** (DP-Alt mode) |
| **USB 2.0** | DS lists: `1x USB2.0 DRD` + `1x USB2.0 Host` + (the USB2 companion of the `1x USB3.1 GEN2 DRD`) | **all three USB2 PHYs used** - see box below |
| **GMAC** | 1x | RGMII -> MAE0621A |
| **SDC0 / SDC2** | SD 3.0 / eMMC 5.1 | microSD / eMMC |
| **SDC1 (SDIO)** | 1x | **free on header J6** (PG0-PG5) |
| **I2C (TWI)** | 16x | 6 buses used (Sec. 7.1) |
| **UART** | 9x | UART0 = console (PB9/PB10), more on headers |
| **SPI** | 5x | SPI1 on header J11 |
| **PWM** | 3x10 ch | incl. fan (PJ27) |
| **I2S** | 5x | **I2S0 on the headers** (only audio path, see below) |
| **MIPI-CSI** | 3x (4+4+2 lanes) | **1x** 4-lane (MCSIB) on **J5** |
| **JTAG** | yes | **PB0-PB3**, partly on headers J11/J6 |
| **RTC** | yes | (but no backup battery) |
| **Thermal sensors** | 5x | `0=CPUB, 1=DDR, 2=NPU, 3=CPUL, 4=GPU` (UM p.359) |
| **Crypto/TRNG** | AES/SM4/SHA/RSA/ECC, secure boot | silicon present |
| **GPIO** | **159 bonded** (PB,PC,PD,PE,PF,PG,PH,PJ,PK + PL,PM). **No PA bonded, no PI bonded.** The GPIO **IP** has more (bank PI exists, PH/PJ larger) - see Sec. 7.8 | (with that refinement) |

### 2.2 Present in the SoC - dead on the board

Software cannot help here. This is the most important part of the inventory:

| Feature | Why dead | Evidence |
|---|---|---|
| **HDMI 2.0b TX** (4K@60, HDCP 2.2) | **All TX pins unconnected at the SoC:** HTX0P/N, HTX1P/N, HTX2P/N, HTXCP/N, HSCL, HSDA, HCEC, HHPD. Only `HDMI-REXT` (R200, 1.62 K) + supply. **There is no HDMI output.** | p.6 |
| **USB 3.2 Gen2** | *Not dead* - lives on **COMB0** (Type-C J4), board-measured 10 Gbps. Listed here only to correct the assumption that all USB ports are 2.0. See Sec. 7.4. | Sec. 7.4 |
| **UFS 3.0** (2-lane) | All UFS high-speed pins unconnected (TX0/1, RX0/1, REFP/REFM, RST-N). Only supply + REXT. Plus the `RTC-VIO` strap set to "Other" instead of "UFS". | p.6, p.8 |
| **MIPI-DSI, LVDS, RGB** | Bank **PD0-PD9 entirely unrouted**. No display connector. | p.7 |
| **Parallel-CSI, MIPI-CSI A + C** | Only **MCSIB** is routed. MCSIA (PK0-PK9) and MCSIC (PK20-PK25) unrouted. | p.7, p.10 |
| **Analog audio** | For audio the DS lists **only digital** interfaces: 5x I2S, 8-ch DMIC, OWA in/out. **No analog codec / headphone amp is listed anywhere.** And the board fits **no external codec** and has **no audio jack**. -> **No analog audio.** Audio is digital only: I2S0 on a header or over DP. | DS p.1; UM Sec. 1.1, 1.2; schematic (no codec, no jack) |
| **NAND / SPI-NOR** | Bank PC used for eMMC only; not even selected in the boot strap. | p.6, p.7 |
| **GPADC / LRADC to the outside** | GPADC0-4 taken by straps, GPADC5/6 unrouted, LRADC0 tied via 10 K to AVCC. **No ADC channels brought out.** | p.6 |
| **BT-PCM audio** | PCM_CLK/DOUT/DIN/SYNC at the WiFi module -> BT audio only over HCI/USB. | p.11 |

> **Two separate combo PHYs - no either/or between PCIe and DP.**
> DS footnotes [8]/[9] (p.42): COMB0 = "combo PHY of DP1.4b, eDP1.4, and USB3.1" (4 lanes);
> COMB1 = "combo PHY of PCIe3.0 and USB3.1" (1 lane). The "or" applies only to the **single** USB3.1 controller.
> **PCIe (COMB1) + DP (COMB0) at the same time is by design** - exactly this board's configuration.
> The USB3.1 controller rides on **COMB0** (Type-C J4), muxed with DP-Alt via HUSB311 - board-measured
> **10 Gbps**. See Sec. 7.4.

---

## 3. DRAM and the 4 / 6 / 8 GB variants - the family-critical part

### 3.1 What is hardware

- **LPDDR5**, **32 bit** = 2 channels x 16 bit (signals `DQ0_A..15_A` / `DQ0_B..15_B`, each with `CA0..5`).
- **CS0 + CS1 per channel** wired -> **two ranks physically possible**.
- A single DRAM package (UD1). **The fitted size distinguishes the variants**; the board is otherwise
  identical.

### 3.1a DRAM rails - **SoC pin or DRAM die?** (this decides which source applies)

Checked by full-text search in the A733 datasheet and against the DS pin table:

| Net | SoC pin? | DS description (verbatim) | DS **abs-max** | Target (LPDDR5) | From |
|---|---|---|---|---|---|
| **`VCC-DRAM`** | balls 1W10, 1W11 | **"Power Supply for DRAM PHY"** | **-0.3 ... 1.4 V** | **1.05 V** | **DCDC7** |
| **`VCC-DRAML`** | balls 1V14, 1W15, 1Y14 | **"Power Supply for DRAM IO"** | **0.27 ... 1.1 V** | **0.5 V** (/0.3 V) | **DCDC6** |
| **`VDD-DRAM`** | balls 1U8, 1U13, 1V8, 1W17, 1W18 | **"Power Supply for DRAM CORE&PHY"** | **-0.3 ... 1.1 V** | typ **0.8 V** | **DCDC2** |
| **`VDD18-DRAM`** | ball 1V12 | "1.8 V Power Supply for DRAM PHY PLL" | 1.674 ... 1.98 V | 1.8 V | CLDO3 |
| `VCC-VDDQ` | **0 DS hits** | - (DRAM die: `VDDQ`) | **unknown** | 0.5 V (/0.3 V) | DCDC6 (via RP15) |
| `VCC09-VDD2L` | **0 DS hits** | - (DRAM die: `VDD2L`) | **unknown** | 0.9 V | ELDO1 |
| `VDD18-LPDDR` | | - (DRAM die: `VDD1`) | **unknown** | 1.8 V | CLDO1 |
| - | - | (DRAM die: `VDD2H`) | **unknown** | 1.05 V | **= `VCC-DRAM`** |

> **Two rails that are easy to confuse:**
> `VCC-DRAM` (DCDC7) is, per the DS, the **DRAM PHY** supply, **not** the core supply.
> The rail the DS describes as **"DRAM CORE&PHY"** is **`VDD-DRAM`** and hangs on **DCDC2** (typ 0.8 V).

> **The one rail that exists at BOTH ends: `VCC-DRAM`.** It is a SoC pin (DS-specified, abs-max **1.4 V**)
> **and** feeds the DRAM die's **`VDD2H`** (schematic p.5). Only for it does the datasheet limit apply.
> For `VDD2L`, `VDDQ` and `VDD1` it does **not** - they are not in the A733 DS.

> **Where does "VDD2L = 0.9 V" come from?** From **the schematic** (table p.5, LPDDR5 column) and from the
> **net name** (`VCC09-VDD2L` -> "09" = 0.9 V). **Not** from the A733 datasheet (0 hits), **not** from a
> DRAM datasheet, and **not** from JEDEC - none of which we have (Sec. 0). So the 0.9 V target is
> **schematic-backed, but not spec-backed** - an honest limitation.

- **Strap RP15/RP20** (p.4) selects the VDDQ source: LPDDR4 -> DCDC7 (1.1 V); **LPDDR4X & LPDDR5 -> DCDC6**.
  The LPDDR5 variant is fitted, schematic note: **"Default: Use LPDDR5"**.

### 3.2 What the board reports is NOT what is fitted

The reported RAM size is a **firmware decision made during DRAM training in the boot loader**, not a
property of the board. You cannot read the fitted variant from a running system, just as you cannot read
it off the chip marking (Sec. 1.1).

> **Consequence for a family kernel:** a board that reports one size may physically carry another. The
> size **must not** be read off a running system and treated as a hardware fact. The clean path for
> 4/6/8 GB is **self-detection during DRAM training**, not a fixed size.

---

## 4. Power

### 4.0a Cheat sheet - which voltage is stated WHERE?

**The three places to look, once and for all:**

| Tag | Document | Exact location |
|---|---|---|
| **DS-typ** | A733 datasheet V0.93 | **Table 5-2 "Recommended Operating Conditions"** -> Sec. 5.3, **p.77** - *typ column* (min/max are `TBD`) |
| **DS-max** | A733 datasheet V0.93 | **Table 5-1 "Absolute Maximum Ratings"** -> Sec. 5.2, **p.75** - *(6 rails also have a minimum)* |
| **DS-pin** | A733 datasheet V0.93 | **Sec. 4.3 "Detailed Signal Description"**, from **p.53** - *(what the pin IS)* |
| **SCH-S** | Schematic V1.10 | **p.4** "POWER AXP318" - *green voltage annotation next to the rail* |
| **SCH-N** | Schematic V1.10 | *net name carries the voltage*: `VCC33-LCD` = 3.3 V, `VCC18-CODEC` = 1.8 V, `VCC09-VDD2L` = 0.9 V |
| **SCH-T** | Schematic V1.10 | **p.5** (LPDDR5 table), **p.4** (DC8SET strap table), **p.6** (strap tables) |
| **MEAS** | PMIC register | `/sys/kernel/debug/regmap/1-0036/registers` - register address + encoding from `axp2101-regulator.c` |

**Summary - where each rail is documented:**

| Where documented | Rails |
|---|---|
| **Schematic p.4, green annotation** (10) | DCDC3, DCDC4, DCDC5, DCDC6, DCDC7, CLDO1, CLDO2, CLDO3, CLDO5, ELDO6 |
| **Schematic, net name only** (5) | DCDC1 (`3V3_CAM1`), ALDO1 (`VCC33-USB`), BLDO2 (`VCC18-LCD`), BLDO4 (`VCC18-CODEC`), ELDO1 (`VCC09-VDD2L`) |
| **Schematic, table/strap** (2) | DCDC6/DCDC7 (LPDDR5 row, p.5), DCDC8 (DC8SET, p.4) |
| **Datasheet Table 5-2 only** (5) | DCDC2 (VDD-SYS), BLDO1 (VCC-MCSI), DLDO1 (VCC-EFUSE), DLDO6 (VCC-UFS), RTCLDO (VCC-RTC) |
| **Nowhere** (1) | **DCDC9** - `ELDO-INPUT` is not a SoC pin => no datasheet; the schematic prints no value |

> **The schematic annotates only 10 of 21 rails with a number.** Read it alone and half the rails have no
> target value - a gap easily filled with kernel values instead. **For 5 rails the datasheet is the ONLY source.**

### 4.0b Does the rail even need power? - **"in the datasheet" != "present on the board"**

The datasheet only says: **this pin exists in the silicon.** It does **not** say whether the board wires it.
Three questions, three sources - only together do they give an answer:

| Question | Source |
|---|---|
| Does the pin exist? | **Datasheet** (Sec. 4.3 pin description) |
| Is it **wired and decoupled**? | **Schematic** |
| Is the rail **actually on**? | **PMIC register** (enable bit) |

| Rail | Consumer | Function alive? | Pin wired + decoupled? | Rail EN? | Verdict |
|---|---|---|---|---|---|
| DCDC1/2/3/5/7/9, ALDO1, CLDO1/3/5, DLDO1, ELDO1/6 | core, DRAM, CPU, PLLs, IO | yes | yes | **ON** | consistent |
| **BLDO1** | VCC-PE/PK/**MCSI** (camera) | camera unused | yes | **OFF** | **consistently off** - but **banks PE/PK dead as a result** |
| **BLDO4** | VCC18-CODEC | no codec fitted | yes | **OFF** | consistently off |
| **DCDC8** | `VCC-UFS-IO` -> ball 1V20 | **UFS dead** (data pins unconnected) | yes (C216/C217) | **ON** | **correct** - the power domain is wired and **must** be supplied |
| **DLDO6** | `VCC-UFS` (2.5 V) | UFS dead | yes | **ON** | same |
| **DCDC2** -> `VDD08-UFS` (ball 1V22) | UFS controller domain | UFS dead | yes (C218/C219) | **ON** | same |
| **DCDC2** -> `VDD08-HDMI` (ball 1G13) | HDMI **digital** domain | **HDMI dead** (all TX pins unconnected) | yes (C209/C210/C214/C215) | **ON** (0.8 V) | **see below** |
| **CLDO2** -> `VCC18-HDMI` (ball 1F13) | HDMI **analog** domain | HDMI dead | yes (C116/C140/C111/C203) | **OFF** (0 V) | **see below** |

> **The HDMI domain is HALF powered - and that is a deviation from Allwinner's reference.**
> Both HDMI supply pins are **wired and decoupled** on the board. But:
> - `VDD08-HDMI` (ball 1G13) is on **DCDC2** -> **0.8 V, ON**
> - `VCC18-HDMI` (ball 1F13) is on **CLDO2** -> **0 V, OFF**
>
> **Allwinner's STD reference (p.4) turns CLDO2 ON:** it states `VCC18-HDMI EXT DCDC->CLDO2(0.5A) -> 1.8V`.
> On the A7S, CLDO2 = **OFF** (enable register `0x21`/CTL2, bit 4 = 0; `0x30` is the voltage register). -> **deviation from the reference design.**
> Practically harmless (HDMI unrouted, Sec. 2.2), but a partly powered domain can in theory leak through
> level shifters/ESD structures. **The vendor treats two dead functions unequally** - UFS fully supplied
> (= reference-compliant), HDMI only half (= reference deviation).
> **If leakage/heat ever becomes an issue: turn CLDO2 on** (reference-compliant; DCDC2's HDMI branch is on
> anyway). No urgent action.
>
> **Do not "clean up" hastily:** the obvious idea - *"HDMI is dead, so switch off DCDC2's HDMI branch too"* -
> does not work: **DCDC2 simultaneously supplies VDD-SYS, VDD-DRAM and VDD-VE.** The rail cannot be switched off.

### 4.0 Master table - datasheet x schematic x measured

**The three sources side by side, rail by rail.** This is the core of the document.

- **DS-typ** = A733 datasheet, Table 5-2 (target value of the SoC pin)
- **DS-window** = Table 5-1 (abs-max; for 6 rails **with** a lower bound)
- **Schematic** = green annotation (`S`), net name (`N`), table/strap (`T`), or - = no value
- **MEAS** = **read raw from the PMIC register and decoded by hand** (`/sys/kernel/debug/regmap/1-0036`),
  **not** from `regulator_summary`. Register + encoding + enable bit from the driver descriptor.

| Rail | feeds (SoC pin) | **DS-typ** | **DS window (abs-max)** | **Schematic** | **MEAS** (reg) | EN | Verdict |
|---|---|---|---|---|---|---|---|
| **DCDC1** | VCC-IO (3V3 system) | **3.3 V** | <= 3.63 V | 3.3 V `N` | **3300 mV** `0x12=0x17` | ON | |
| **DCDC2** | VDD-SYS, VDD-DRAM, VDD-VE, **AVDD-D-COMB0** | **0.8 V** | **0.744 ... 0.88 V** | - | **800 mV** `0x13=0x9E` | ON | **mid-window** |
| **DCDC3** | VDD-CPUB (A76) | 0.8 V | <= 1.2 V | 0.8-1 V `S` | **800 mV** (idle) | ON | |
| **DCDC4** | VDD-GPU | 0.8 V | <= 1.2 V | 0.8-0.96 V `S` | **800 mV** (idle) | ON | |
| **DCDC5** | VDD-CPUL (A55) + DSU | 0.8 V | <= 1.2 V | 0.8-1 V `S` | **800 mV** (idle) | ON | |
| **DCDC6** | VCC-DRAML (SoC) + VDDQ (die) | **0.5 V** (LPDDR5) | **0.27 ... 1.1 V** | 0.5 V `S`+`T` | **560 mV** `0x17=0x86` | ON | **+12% over target - but clearly INSIDE the window** |
| **DCDC7** | VCC-DRAM = DRAM **PHY** + VDD2H (die) | **1.05 V** (LPDDR5) | <= 1.4 V | 1.05 V `S`+`T` | **1080 mV** `0x18=0xBA` | ON | +3% over target, in window |
| **DCDC8** | VCC-UFS-IO (= DS **VCC12-UFS**, CORE) | **1.2 V** | **<= 1.2 V** (0 margin!) | 1.2 V `T` (DC8SET) | **1200 mV** `0x19=0xC6` | ON | **exactly at abs-max** (UFS dead) |
| **DCDC9** | ELDO-INPUT (**not a SoC pin**) | - | - | - | **1240 mV** `0x1A=0xC8` | ON | **the only source-less rail** |
| **ALDO1** | VCC-PL, VCC33-USB, VCC_3V3_TYPEC | **3.3 V** | <= 3.63 V | 3.3 V `N` | **3300 mV** `0x24=0x1C` | ON | |
| **BLDO1** | VCC-PE, VCC-PK, **VCC-MCSI** | **1.8 V** | **<= 2.1 V** | - | **1800 mV** `0x2A=0x0D` | **OFF** | value ok - **but banks PE/PK unpowered** |
| **BLDO2** | VCC-PD, **VCC-LVDS0**, VCC-PJ, VCC18-LCD | **3.3 V** (AW ref!) | Table 5-1: LVDS <= 2.1 V | **3.3 V** `REF` (AW STD p.4) | **3300 mV** `0x2B=0x1C` | **ON** | **= Allwinner reference**; node really 3.3 V (SWOUT1 via R251); LVDS unrouted -> Sec. 8.1 |
| **BLDO4** | VCC18-CODEC | - | - | 1.8 V `N` | **1800 mV** `0x2D=0x0D` | **OFF** | (no codec) |
| **CLDO1** | VCC-PM, VDD18-LPDDR | 1.8 V | - | 1.8 V `S` | **1800 mV** `0x2F=0x0D` | ON | |
| **CLDO2** | VCC18-HDMI | 1.8 V | - | 1.8 V `S` | **1800 mV** `0x30=0x0D` | **OFF** | (HDMI dead) |
| **CLDO3** | **VDD18-DRAM, VCC-PLL, VCC-CPU-PLL, VCC-DCXO** | 1.8 V | **1.674 ... 1.98 V** | 1.8 V `S` | **1800 mV** `0x31=0x0D` | ON | **critical (all PLLs)** |
| **CLDO5** | VCC-PC (eMMC VCCQ), VCC18-PF, AVDD-H-COMB0 | 1.8 V | <= 2.1 V | 1.8 V `S` | **1800 mV** `0x33=0x0D` | ON | |
| **DLDO1** | VCC-EFUSE | **1.8 V** | <= 2.1 V | - | **1800 mV** `0x34=0x0D` | ON | |
| **DLDO6** | VCC-UFS (DS: UFS **IO**) | **2.5 V** | **<= 2.5 V** (0 margin!) | - | **2500 mV** `0x39=0x14` | ON | **exactly at abs-max** (UFS dead) |
| **ELDO1** | **VCC09-VDD2L** (DRAM **die**) | **not a SoC pin** | **unknown** (no LPDDR5 spec) | **0.9 V** `p.5` | **900 mV** `0x3A=0x10` | ON | **matches the schematic target 0.9 V** (Sec. 3.1a) |
| **ELDO6** | VDD-CPUS + VDD-USB | 0.8 V | <= 1.2 V, VDD08-USB **0.744...0.88** | 0.8 V `S` | **800 mV** `0x3F=0x0C` | ON | |

#### What this table shows

1. **Datasheet and schematic contradict each other NOWHERE.** Where both speak, they agree.
   Where the DS offers several options (LPDDR4 / 4X / 5), the schematic **picks** one.
2. **The measured value matches the target on 18 of 21 rails.** The vendor configures cleanly.
3. **Three deviations - and only ONE is serious:**
   - **DCDC6** (560 instead of 500 mV) and **DCDC7** (1080 instead of 1050 mV): above the *target*, but
     **clearly within the abs-max window**. A systematic vendor setting (identical on a second board). **No risk.**
   - **BLDO2** (3300 instead of 1800 mV): **the only rail whose OTP value exceeds the target pin's abs-max.**
     The node really sits at **3.3 V** (SWOUT1 via fitted R251) - but Allwinner reference, LVDS unrouted. **Sec. 8.1.**
4. **Two rails sit exactly at their abs-max** (DCDC8 = 1.2 V, DLDO6 = 2.5 V) - both UFS, both dead.
   **Zero headroom.** Never raise them.
5. **DCDC9 is the only rail with no source at all** - `ELDO-INPUT` is not a SoC pin, so no datasheet;
   the schematic prints no value.

### 4.1 Topology

```
USB-C J16 (5 V, sink) --> U12 SGM40666AS (4.5 A, OVLO) --> VCC5V0_SYS
                                     |
          +--------------------------+--------------------------+
   AXP318 DCIN (P1,P2,R1,R2)                        UP11 ETA3521 (external buck)
          |                                                |
   DCDC1..9, ALDO (<-PS), DLDO (<-PS), ELDO (<-DCDC9)   LDO2V5-INPUT (~2.4-2.5 V)
          |                                                |
   SW1/SW2 (internal load switches)                 BLDO-INPUT, CLDO-INPUT
          |
   SWOUT1 / SWOUT2
```

**Two traps, or you compute wrong:**
1. **DCDC9 is not a load rail but the pre-regulator of the ELDO group.**
2. **BLDO*/CLDO* hang off ~2.5 V**, not 5 V. An LDO cannot exceed its input ->
   **all B/C LDOs are 1.8 V class.**

### 4.2 Rail map (schematic = target; measured value only as confirmation)

> **PROVENANCE - read this first.** The schematic does **not** annotate every rail with a voltage.
> The **"Source"** column states, for **each individual value**, where it comes from:
> **`S`** = green voltage annotation on schematic p.4 - **`N`** = derivable from the **net name**
> (`VCC33-LCD` -> 3.3 V; `VCC18-CODEC` -> 1.8 V; `VCC09-VDD2L` -> 0.9 V) - **`T`** = from a
> **schematic table/strap** - **`D`** = **datasheet** - **`K`** = **read only from the kernel/PMIC
> register - NOT ground truth**.

| Rail | Feeds | Target | **Source** | Inductor (rating) | Current budget | DVFS |
|---|---|---|---|---|---|---|
| **DCDC1** | 3V3 system + internal SW1/SW2 | 3.3 V | **N** (feeds SWOUT1/2 = `VCC33-*`) | LP1 1 uH **4.7 A** | - | no |
| **DCDC2** | **VDD-SYS** + **VDD-DRAM** + VDD-VE + VDD-COMB0/1 + VDD08-HDMI/UFS | typ **0.8 V** | **D** (DS Table 5-2, VDD-SYS/VDD-DRAM typ 0.8 V) | LP2 0.33 uH **5.1 A** | **>= 5.0 A** | Sec. 8.3 |
| **DCDC3** | **VDD-CPUB** (A76) | **0.8-1 V** | **S** | LP3 0.33 uH **5.1 A** | **5.0 A** | |
| **DCDC4** | **VDD-GPU** | **0.8-0.96 V** | **S** | LP4 0.33 uH **5.1 A** | 2.0 A | |
| **DCDC5** | **VDD-CPUL** (A55) **+ DSU** | **0.8-1 V** | **S** | LP5 0.33 uH **5.1 A** | **5.0 A** | |
| **DCDC6** | VCC-DRAML (SoC IO) + **VCC-VDDQ** (DRAM die) | 0.8 / 0.6 / **0.5 V** | **S** + **T** (LPDDR5 row p.5) | LP6 1 uH **4.7 A** | - | no |
| **DCDC7** | **VCC-DRAM** = DRAM **PHY** (SoC) + **VDD2H** (DRAM die) | 1.1 / **1.05 V** | **S** + **T** | LP7 1 uH **4.7 A** | - | no |
| **DCDC8** | VCC-UFS-IO | **1.2 V** | **T** (strap DC8SET: RP55=1 K, RP57=NC) | LP8 1 uH **4.7 A** | unused | unused |
| **DCDC9** | **ELDO-INPUT** (pre-regulator) | *1.24 V* | **K** - **the only remaining kernel gap.** Neither schematic nor A733 DS gives a value (ELDO-INPUT is **not a SoC pin**). Only derivable: **> 0.9 V** (must feed ELDO1). | LP9 1 uH **4.7 A** | - | no |
| (ETA3521) | `LDO2V5` -> BLDO/CLDO groups | **2.5 V** | **N** (net name `LDO2V5`). An earlier calc of "2.4 V from the FB divider" assumed **V_fb = 0.6 V** - and there is **no ETA3521 datasheet**. The net name carries; the calc does not. | LP16 1 uH **2.3 A** | - | no |
| SWOUT1 (<-DCDC1) | `VCC33-LCD` -> also Ethernet PHY | 3.3 V | **N** | - | IMAX **1 A** (S) | - |
| SWOUT2 (<-DCDC1) | `VCC-CARD`, `VCC-eMMC`, `VCC-PH`, `VCC-IO`, `VCC33-USB-2` | 3.3 V | **N** | - | - | - |
| ALDO1 | `VCC-PL`, `VCC33-USB`, `VCC_3V3_TYPEC`, EEPROM | 3.3 V | **N** + **D** (Table 5-2: `VCC33-USB` = 3.3 V) | - | IMAX 0.6 A (S) | - |
| BLDO1 | `VCC-PE`, `VCC-PK`, **`VCC-MCSI`** | **1.8 V** | **D** - Table 5-2: `VCC-MCSI` typ **1.8 V**; Table 5-1: **abs-max 2.1 V** => BLDO1 **cannot** be 3.3 V. | - | IMAX 0.5 A (S) | - |
| BLDO2 | `VCC-PD`, **`VCC-LVDS0`**, `VCC-PJ`, `VCC18-LCD` | target **1.8 V**, OTP **3.3 V**, **real 3.3 V** | **D + register + REF** - DS: `VCC-LVDS` typ 1.8 V / abs-max 2.1 V. OTP 3.3 V (reg `0x2B`=`0x1C`); **R251 fitted -> SWOUT1 holds the node at real 3.3 V**. = Allwinner reference (p.4), LVDS unrouted -> **Sec. 8.1** | - | IMAX 0.5 A (S) | - |
| BLDO4 | **`VCC18-CODEC`** | 1.8 V | **N** | - | IMAX 0.5 A (S) | - |
| CLDO1 | `VCC-PM`, **`VDD18-LPDDR`** | **1.8 V** | **S** + **N** | - | IMAX 0.3 A (S) | - |
| CLDO2 | `VCC18-HDMI` | **1.8 V** | **S** + **N** | - | IMAX 0.3 A (S) | - |
| **CLDO3** | `VDD18-DRAM`, **VCC-PLL, VCC-CPU-PLL, VCC-DCXO**, AVCC | **1.8 V** | **S** + **N** | - | IMAX 0.3 A (S) | **critical: all PLLs** |
| CLDO5 | **`VCC-PC`** (eMMC VCCQ), `VCC18-PF`, AVDD-H-COMB0 | **1.8 V** | **S** + **N** | - | IMAX 0.5 A (S) | - |
| DLDO1 | `VCC-EFUSE` | **1.8 V** | **D** - Table 5-2: `VCC-EFUSE` typ **1.8 V** (abs-max 2.1 V) | - | IMAX 0.5 A (S) | - |
| DLDO6 | `VCC-UFS` | **2.5 V** | **D** - Table 5-2: `VCC-UFS` (UFS-3.x IO) typ **2.5 V** (abs-max 2.5 V). UFS is dead on the board. | - | IMAX 0.4 A (S) | - |
| **ELDO1** | **`VCC09-VDD2L`** (LPDDR5 die) | **0.9 V** | **N** (`09`) + **T** (p.5 table) | - | IMAX 0.6 A (S) | - |
| ELDO6 | **VDD-CPUS** + VDD-USB | **0.8 V** | **S** | - | IMAX 0.2 A (S) | - |
| RTCLDO | `VCC-RTC` | **1.8 V** | **D** - Table 5-2: `VCC-RTC` typ **1.8 V** (abs-max 2.1 V) | - | - | - |

> **Provenance balance:** of 23 rails, **22 are backed by schematic/datasheet**.
> **Exactly one** kernel gap remains: **DCDC9** (`ELDO-INPUT` is not a SoC pin and appears in no source).
> **Plus BLDO2 - there it is not a "missing value" but an over-abs-max hazard (Sec. 8.1).**
>
> **How the gap arose:** DS **Table 5-2** ("Recommended Operating Conditions") was first dismissed as
> worthless because min/max are `TBD` throughout - **but the typ column is filled** and gives the target
> for **every SoC rail**. Five rails once marked "kernel-only" are stated there in black and white.
> The gap was not in the sources, but in not reading the source to the end.

> **B2 - the headroom on DCDC2/3/5 is practically zero.**
> On **DCDC2** hang, per schematic p.4: `VDD-SYS` + `VDD-DRAM` + `VDD-VE` + `VDD-COMB0/1` + `VDD08-HDMI/UFS`.
> With the current budgets from Sec. 4.3 (schematic p.8): **3500 + 1000 + 500 mA = 5000 mA** - **before**
> COMB0/COMB1 are even counted (HDMI/UFS are dead on the board -> 0).
> Inductor **LP2 is rated 5.1 A**. -> **~98% utilization, under 2% headroom.**
> The same holds for **DCDC3** and **DCDC5** (5000 mA budget each on a 5.1 A inductor).
>
> This is **not a design fault** - these are maxima never drawn simultaneously. But it says two things:
> (1) on these three rails there is **no room** for extra load, and (2) a **voltage increase** on DCDC2
> (Sec. 4.5!) is doubly dangerous - it raises dissipation **and** current on a path already designed to the limit.

| Rail | Feeds | V | IMAX | DVFS |
|---|---|---|---|---|
| SWOUT1 (<-DCDC1) | VCC33-LCD, **Ethernet PHY** | 3.3 V | **1 A** | - |
| SWOUT2 (<-DCDC1) | VCC-CARD, **VCC-eMMC**, VCC-PH, VCC-IO, VCC33-USB-2 | 3.3 V | - | - |
| ALDO1 | VCC-PL, VCC33-USB, VCC_3V3_TYPEC, EEPROM | 3.3 V | 0.6 A | - |
| BLDO1 | VCC-PE, VCC-PK, VCC-MCSI (camera IO) | 1.8 V | 0.5 A | - |
| BLDO2 | VCC-PD, VCC-LVDS0, VCC-PJ, VCC18-LCD | Sec. 8.1 | 0.5 A | - |
| BLDO4 | VCC18-CODEC | 1.8 V | 0.5 A | - |
| CLDO1 | VCC-PM, VDD18-LPDDR | 1.8 V | 0.3 A | - |
| CLDO2 | VCC18-HDMI | 1.8 V | 0.3 A | - |
| **CLDO3** | VDD18-DRAM, **VCC-PLL, VCC-CPU-PLL, VCC-DCXO**, AVCC | 1.8 V | 0.3 A | **critical: all PLLs** |
| CLDO5 | **VCC-PC** (eMMC VCCQ), VCC18-PF, AVDD-H-COMB0 | 1.8 V | 0.5 A | - |
| DLDO1 | VCC-EFUSE | 1.8 V | 0.5 A | - |
| DLDO6 | VCC-UFS | 2.5 V | 0.4 A | - |
| **ELDO1** | **VCC09-VDD2L** (LPDDR5) | **0.9 V** | 0.6 A | - |
| ELDO6 | **VDD-CPUS** + VDD-USB | **0.8 V** | 0.2 A | - |
| RTCLDO | VCC-RTC | 1.8 V | - | - |

**Unused (PMIC pins):** ALDO2/3/4/6, BLDO3/5, CLDO4, DLDO2/3/4/5, ELDO2/3/4/5.
**Also unconnected:** `BKUPBAT` (no RTC battery), `TS` (no thermistor), PMIC `GPIO1-3`, PMIC `GPADC`.
**Remote sense** for DCDC2/3/4/5 present (`VDD-SYSFB/CPUBFB/GPUFB/CPULFB`, "close to soc") -> clean
load regulation on DVFS steps.

> **DCDC6: the schematic says 0.5 V, the PMIC is programmed to 560 mV - but be careful with "measured".**
> - **Ground truth (schematic p.5, LPDDR5 row):** `VCC-VDDQ` / `VCC-DRAML` = **0.5 V**.
> - **What we actually have:** `560 mV` from `regulator_summary`. That is **not a voltage measurement**
> but the driver's **readback path** (register code -> voltage) - and **this very driver has proven
> code<->voltage bugs, including on DCDC6** (`axp2101-regulator.c`, code<->voltage DCDC6-9). The value is **doubly indirect**.
> - **What holds:** a **second, independent board** (6 GB, **vendor kernel 6.6.98**) reads the **same**
> value (likewise DCDC7 = 1080 mV, ELDO1 = 900 mV). Two different kernels, same register => **the register
> really is set this way, systematically by the vendor.** That is solid.
> - **What does NOT hold:** the claim that the coil actually sits at 560 mV.
>
> **So:** document it as *"the vendor programs DCDC6 above the schematic target"* - **do not "fix" it**.
> The real voltage is settled **only by a multimeter at the decoupling cap**, not by `regulator_summary`.

### 4.3 Current budget per SoC rail (schematic p.8 - **not stated this way in the datasheet**)

| SoC rail | max current |
|---|---|
| VDD-CPUB (A76) | **5000 mA** |
| VDD-CPUL (A55) | **5000 mA** |
| VDD-SYS | **3500 mA** |
| VDD-GPU | **2000 mA** |
| VDD-DRAM | **1000 mA** |
| VDD-VE | **500 mA** |
| VDD-CPUS | 50 mA, VCC-ADC 50 mA, VCC-RTC 10 mA |

### 4.4 PMIC DVFS mechanics (AXP318W DS, draft V0.1)

- **DVM** for DCDC2-9, individually via **bit 7 of each voltage register**. **Default: off.**
- **Slew rate** global `REG1BH[5]`: 7.8125 us/step (default) or 15.625 us/step.
- **No shadow/VSEL register** for fast switching.
- **Poly-phase** DCDC2+3 / DCDC4+5 via `REG1BH[6]/[7]` - DS p.24: "requires customization".
- **Defaults are OTP/customer-specific** => **not derivable from the datasheet**, only readable from the
  chip via `i2cdump -y <bus> 0x36`.

> **What the datasheet carries - and what not.**
>
> | | |
> |---|---|
> | **Nominal voltage of each SoC rail** | **Table 5-2, typ column** - filled, complete. (Min/max there are `TBD` - that does not make the table worthless.) |
> | **Hard operating window** | **Table 5-1** - a ceiling for all rails, **and for 6 rails also a floor** (positive minimum, Sec. 4.5). |
> | **OPP table (frequency <-> voltage)** | **missing.** Comes from DT/eFuse, is **silicon-bin dependent** (measured board = bin `vf0400`) -> **software level**. |
> | **Currents / power draw** | **missing** in the DS. The current budget in Sec. 4.3 comes from the **schematic p.8**. |
>
> -> **DVFS cannot be *dimensioned* from the DS** (no OPPs). **But the limits within which DVFS may operate
> are very much there** - and exactly those have not yet been carried into the DT (Sec. 4.6).

---

### 4.5 The permitted voltage WINDOW (DS Table 5-1) - **and what the DT allows**

> **Table 5-1 is NOT just a ceiling.** **Six** rows have a **positive minimum** - for those it is a
> **window**: the rail must also not go **below**.
>
> | Rail with positive minimum | **Window (Table 5-1)** | Hangs on |
> |---|---|---|
> | **VDD-VE** | **0.744 ... 1.1 V** | **DCDC2** |
> | **AVDD-D-COMB0** | **0.744 ... 0.88 V** <- tightest limit of all | **DCDC2** (via `VDD-COMB0`, R75 = 0 ohm) |
> | VDD08-USB | 0.744 ... 0.88 V | ELDO6 (`VDD-USB`) |
> | VDD-UFS | 0.744 ... 0.88 V | DCDC2 (`VDD08-UFS`) - UFS dead |
> | VCC-DRAML | 0.27 ... 1.1 V | DCDC6 |
> | VDD18-DRAM | 1.674 ... 1.98 V | CLDO3 |

| SoC rail | **Permitted window** | Fed by | **The current DT allows** | Violation |
|---|---|---|---|---|
| VDD-CPUB | -0.3 ... **1.2 V** | DCDC3 | 0.5 ... **1.54 V** | top **+28%** |
| VDD-CPUL | -0.3 ... **1.2 V** | DCDC5 | 0.5 ... **1.54 V** | top **+28%** |
| VDD-GPU | -0.3 ... **1.2 V** | DCDC4 | 0.5 ... **1.54 V** | top **+28%** |
| **ELDO6 total** | **0.744 ... 0.88 V** | **ELDO6** | 0.5 ... 1.5 V | **top +70%** (binding = VDD08-USB) |
| + VDD-CPUS | -0.3 ... 1.2 V | ELDO6 | | |
| + **VDD08-USB** | **0.744 ... 0.88 V** | ELDO6 | | <- **the binding limit** |
| **DCDC2 total** | **0.744 ... 0.88 V** | **DCDC2** | **0.5 ... 1.54 V** | **top +75%** **AND too low at the bottom** |
| + VDD-SYS | -0.3 ... 1.1 V | DCDC2 | | |
| + VDD-DRAM | -0.3 ... 1.1 V | DCDC2 | | |
| + VDD-VE | **0.744** ... 1.1 V | DCDC2 | | |
| + **AVDD-D-COMB0** | **0.744 ... 0.88 V** | DCDC2 | | <- **the binding limit** |
| VCC-DRAML | 0.27 ... 1.1 V | DCDC6 | 0.5 ... 2.76 V | top +151% |
| VCC-DRAM | -0.3 ... 1.4 V | DCDC7 | 0.5 ... **1.84 V** | **top +31%** (DRAM PHY!) |
| **VCC-LVDS** | **-0.3 ... 2.1 V** | **BLDO2** | 0.5 ... **3.4 V** | **Sec. 8.1** |
| VCC-MCSI | -0.3 ... 2.1 V | BLDO1 | 0.5 ... 3.4 V | top |
| VDD18-DRAM | 1.674 ... 1.98 V | CLDO3 | 0.5 ... **3.5 V** | both sides |
| VCC-IO / VCC33-* | -0.3 ... 3.63 V | DCDC1 | 1.0 ... **3.8 V** | top +4.7% |

> **DCDC2 is the extreme case.**
> Because **AVDD-D-COMB0** (abs-max **0.88 V**) hangs on DCDC2 via a **0-ohm bridge**, the permitted window
> of this rail is **0.744 ... 0.88 V** - not 1.1 V.
> The current DT allows **0.5 ... 1.54 V**: **75% too high at the top, too low at the bottom.**
> The measured value (**0.800 V**, from OTP) sits **centered and correct** in the window. **The vendor gets
> it right - the DT just does not clamp it.** (-> the rule in **Sec. 4.6**.)
> Aggravating: DCDC2 is declared in the DT as **`npu-supply`** (Sec. 8.3), while an unused NPU OPP table up
> to 1120 MHz sits next to it - and DCDC2 is at its current limit anyway (Sec. 4.2).

### 4.6 The rule: **"not bounded" is not the same as "wrongly set"**

Two rails deviate from the target of the rank-1 sources. **Both follow the same pattern:**

| Rail | Target (rank-1 source) | Measured | Who sets it? | What does Linux do? |
|---|---|---|---|---|
| **VCC-VDDQ** (DCDC6) | **0.5 V** (schematic p.5) | **0.560 V** | **OTP** (vendor) | inherits - no `init-microvolt` |
| **VCC-LVDS** (BLDO2) | **1.8 V** (DS Table 5-2) | **3.3 V** (OTP + SWOUT1 via R251) - Allwinner reference, LVDS unrouted | **OTP** (vendor) | inherits - no `init-microvolt` |

**Backed:** `regulator-init-microvolt` appears **0x** in the board DTS. `reg_bldo2` has **only** a wide
range (`500000`-`3400000`), **no** `min == max`, and `pcie3v3-supply` merely *enables* the rail.
-> **Linux programs none of these voltages. It reads them back from the PMIC register and accepts them**
because they fit the (far too wide) DT window.

> **The resulting rule for a family kernel:**
>
> **1. The issue is not "wrong voltages are set" - it is "they are not bounded."**
> The dangerous values come from **OTP/bootloader**, not from the DT. The DT is not *wrongly set*,
> it is **not set**.
>
> **2. `regulator-min/max-microvolt` must reflect the HARDWARE window (DS Table 5-1), not the PMIC's
> control range.** The PMIC *can* do 3.4 V; the SoC pin tolerates 2.1 V. What the DT writes is a
> **safety guarantee**, not a capability description.
>
> **3. Only then do OTP deviations become VISIBLE.** With correct constraints the regulator core would
> report or correct an OTP value outside the window at boot - instead of silently adopting it. That is
> exactly why the 3.3 V on a 2.1 V pin **went unnoticed**.
>
> **4. This is family-critical:** OTP is **batch-dependent**. Another board of the same family may carry
> different defaults. A kernel that does not clamp relies on the vendor having burned the same values everywhere.

**But take care with rails that have an external 0-ohm second source** (BLDO2/VCC-PD node, Sec. 8.1): there
**SWOUT1 via the fitted R251** holds the 3.3 V - a DT constraint on BLDO2 is ineffective, the LDO would
regulate against the source. Such nodes are changeable **only in hardware**. For the pure DVFS rails
(DCDC2/3/4/5) the rule applies directly and safely.

## 5. Clocks

| Crystal | Frequency | For | Evidence |
|---|---|---|---|
| **X1** | **26 MHz** (16 pF, 10 ppm) | SoC DCXO. Confirmed by **CK-SEL strap** (SEL1=1, SEL0=0 -> "SOC=26M") | p.8 |
| **Y2** | **32.768 kHz** (12.5 pF, 20 ppm) | RTC | p.8 |
| **Y1** | **25 MHz** (12 pF, 10 ppm) | Ethernet PHY (**own** crystal; the SoC clock `EPHY-CLK-25M` is **not** connected, R99 = **NC**) | p.15 |
| **Y3** | **12 MHz** (20 pF, 10 ppm) | USB hub CH334F | p.13 |

---

### 5.1 The PLLs do **NOT** run off the 26 MHz crystal

The crystal is unambiguously 26 MHz (photo + strap + schematic, all consistent). **But the PLLs do not see it.**
There is an **intermediate stage**:

```
DCXO 26 MHz --> PLL_REF --> 24 MHz --> pll-gpu, pll-ddr, pll-npu, pll-peri, pll-ve, pll-video, pll-de, pll-audio
```

**Evidence:**
- **UM (CCU chapter):** reference column per PLL - `PLL_REF ... 26MHz`, but `PLL_DDR / PLL_GPU0 / PLL_VIDEO /
  PLL_VE / PLL_AUDIO / PLL_PERI1 ... **24MHz**`. `PLL_REF` output = **2.496 GHz**.
- **BSP CCU** (`ccu-sun60iw2.c`): **only** `pll-ref` has parent `"dcxo"`. **All** other PLLs
  (`pll-gpu`, `pll-ddr`, `pll-npu`, `pll-peri0/1`, `pll-ve0/1`, `pll-video0/1/2`, `pll-de`, `pll-audio*`)
  have parent `"pll-ref"`.
- **Calculation:** 26 MHz x 96 = **2.496 GHz VCO** (= UM value); output divider /104 -> **24 MHz** (= the PLL_REF output actually used).
  `pll_ref_clk` matches with `n = 80...116` and a 7-bit output divider, `min_rate = 15 MHz`.
- **Clock tree (cross-check):** `dcxo26M 26000000 -> dcxo 26000000 -> pll-ref **24000000** -> pll-gpu/...`

> **Why it matters:** computing N/M of a PLL against **26 MHz** is **systematically ~8% off**.
> The reference is **24 MHz** - for **every** PLL except `PLL_REF` itself.
### 5.2 Buttons, LEDs, Ethernet magnetics

| Part | Type / marking | Function | Connection | Evidence |
|---|---|---|---|---|
| **L7-L10** | BCMEF122P900H x4 | Ethernet magnetics (discrete) | - | p.15 |
| **J2** | RJ45 `HRK1_1B71G59Z...` | Ethernet jack | - | p.15 |
| **SW1** | TS_018B (button) | **FEL/recovery** - the **only** button | pulls `FEL` to GND via 1 K | p.16 |
| **LED1/LED2** | blue / green | status | **PJ26** (blue), **PM2** (green) | p.16 |

> **No power button.** `PMU-PWRON` ends on two test pads (TP12/TP13). A power key needs soldering.
> **No RTC backup battery.** PMIC pin `BKUPBAT` is unconnected -> the RTC loses time without power.

## 6. Straps - hard-wired, they determine boot behavior

All via resistor dividers onto GPADC channels (schematic p.6, tables included there).

| Strap | Populated | Value | Meaning |
|---|---|---|---|
| **BOOT-SEL** (GPADC0) | R5 = 10 K / R9 = **3.9 K** | 505 mV | **`SMHC0 -> EMMC_BOOT -> EMMC_USER -> TRY`** (in the schematic **red** = selected) -> **SD first**, then eMMC, then FEL. No NAND/SPI in the boot path. |
| **USB-UPGRADE / COMBO1** (GPADC3) | 10 K / **27 K** | 1315 mV | **upgrade port = USB0**, **COMBO1 = PCIE** (**blue** = selected). <- **This is where USB 3.0 on COMB1 dies** (it lives on COMB0 instead, Sec. 7.4). |
| **BOARD-ID** (GPADC2) | R7=47 K up, R8=NC, R9=10 K, R10=**27 K** | 1315 mV | **Config-14** (**blue** = selected) |
| **DDR-PARA** (GPADC1 + GPIO **PH16**) | GPIO: R3=NC/47 K up, R4=**1 K** down -> **0**; ADC: R5=10 K up, R6=**2.7 K** down | 382 mV | -> **DDR PARA 2** - **but the schematic marks PARA 11**, see Sec. 8.2 |
| **DC8SET** | RP55 = 1 K, RP57 = NC | - | **DCDC8 = 1.2 V** |
| **CLK-SEL** | SEL1=1, SEL0=0 | - | **SoC = 26 MHz** |
| **RTC-VIO** | R17 = 0 R -> GND | - | mode **"Other"** (not "UFS") |

> **PH16 is dual-purpose:** via RA4 (0 R) as **GMAC1_RSTn_L** (PHY reset) *and* via RA5 (0 R) as
> **DDR-PARA-SEL GPIO**. Read as a strap at boot (1 K pulldown -> 0), later driven as PHY reset.
> A family kernel must know this.

---

## 7. Connections & addressing

### 7.1 I2C buses

> **Provenance:** buses, pins and pull-ups are **schematic** (p.7, p.16). The **I2C addresses are not** -
> the schematic names none. `0x36` comes from the **AXP318W datasheet** (chip property, rank 1);
> `0x50` for the EEPROM is the **typical** BL24C16 value and **not confirmed** - verify on the bus.

| Bus | SoC pins | Pull-ups to | Device | Address |
|---|---|---|---|---|
| **S-TWI0** | PL0 / PL1 | 2.2 K -> VCC-PL | **AXP318 PMIC** | **0x36** (AXP318W DS) |
| **S-TWI1** | PL12 / PL13 | 2.2 K -> VCC_3V3_TYPEC | **HUSB311** (USB-C PD), INT = **PL3** | - |
| **TWI0** | PG8 / PG9 | 2.2 K -> VCC-PL | **BL24C16F EEPROM** (WP = **PC7**) | typ. 0x50 |
| **TWI2** | PD16 / PD17 | 2.2 K -> VCC33-LCD | **header J11** (free) | - |
| **TWI3** | PE3 / PE4 | 2.2 K -> VCC-PE | **camera J5** | - |
| **TWI7** | PJ22 / PJ23 | 2.2 K -> VCC33-LCD | **header J11** (free) | - |

### 7.2 Hard-assigned GPIOs (not freely usable)

| GPIO | Function | | GPIO | Function |
|---|---|---|---|---|
| **PD20** | PCIE_PWR_EN | | **PM0** | USB_WIFI_PWR (load switch Q8) |
| **PD21** | PCIE_WAKEn | | **PM1** | WL-REG-ON -> CHIP_EN WiFi |
| **PD22** | PCIE_PERSTn | | **PM2** | LED green |
| **PD23** | PCIE_CLKREQn | | **PM5** | USB_HOST_EN (VBUS USB-A) |
| **PL2** | USB0-DRVVBUS (VBUS out J4) | | **PJ26** | LED blue |
| **PL3** | TYPEC_INT (HUSB311) | | **PJ27** | fan PWM |
| **PH16** | GMAC1_RSTn_L **+ DDR-PARA strap** | | **PC7** | EEPROM-WP |
| **PF6** | SD card-detect | | **PE5/PE8/PE9** | camera MCLK / STBY / RESET (PE10 = unused) |
| **PB0-PB3** | **JTAG** (TMS/TCK/TDO/TDI) | | **PB9 / PB10** | **UART0 = debug console** |

### 7.3 Connectors

**J11 - 30-pin (2x15):** 3.3 V, 5 V, GND, **TWI7**, **TWI2**, **UART0** (PB9/PB10),
**SPI1** (PD10-PD13), **I2S0-BCLK**, GPIOs **PB0, PB1, PB2** (= JTAG!), **PD14, PJ24, PJ25, PL5, PL6, PL7**.

**J6 - 15-pin:** **PB3** (JTAG TDI), **PM3, PM4**, **I2S0** (LRCK, MCLK, DIN0, DOUT0),
**PG0-PG5 = SDC1/SDIO** (an SDIO device can attach here).

> Audio goes **only** over I2S0 on the two headers (BCLK on J11, rest on J6) - there is no codec.

**J3 - PCIe (`FPC_2X8P`):** **5 V only** + signals + control. **No 3.3 V / 1.8 V pins on the connector** -
the NVMe module makes its own voltages.
-> The DT names `pcie1v8-supply` / `pcie3v3-supply` are **SoC bank supplies**, not connector rails.

**J5 - camera (`CAM_31P`):** MIPI-CSI **4 lanes + clock** (MCSIB), MCLK (PE5), **TWI3**, **RESET = `MCSI-RST-R` -> ball 1A3 = PE9**,
**STBY = `MCSI-STBY-R` -> ball A2 = PE8**, supply **3.3 V and 5 V**. Second clock / CAM1 / PDN1 unused.
*(Correction: an earlier revision said STBY=PE9/RESET=PE10 - off by one; PE10 (ball 1A1) carries no camera net, unused.)*

**J7 - fan (`CONM_1X3`):** GND, **PWM (PJ27)**, **5 V**. **No tach pin.**

**J1 - antenna:** u.FL, **one** antenna shared by WiFi **and** BT (ANT_BT via R27 = NC).

### 7.4 USB - the two Type-C ports have **different** roles

> **Board-measured (kernel 6.18.36).** The earlier claim "no USB3 / all ports USB 2.0" (Sec. 2.2 + box below)
> was **wrong**. Measured:
> - **J4 (USB-C)** delivers **USB 3.2 Gen2, 10 Gbps** - a Kingston DT Max enumerates as `usb 2-1: SuperSpeed Plus Gen 2x1`,
> `10000M`, driver `uas`, mountable as `sda`. The `phy_switcher@10` switches `STATE_SAFE -> STATE_USB`.
> -> The **single** USB3.1 controller rides on **COMB0** (not COMB1), muxed with DP-Alt via HUSB311.
> DS footnote [8] confirms it: COMB0 = "DP1.4b, eDP1.4, **and USB3.1**". COMB1=PCIe is irrelevant here.
> **Throughput board-measured** (NVMe->USB, single-stream `dd`, direct+fdatasync): **>=493 MB/s read, >=211 MB/s write.**
> These are **floors, NOT the port ceiling** - the DT Max is rated **1000/900 MB/s** (SM2320+TLC), so it is
> throttled by the A733 USB3 path (dwc3/xhci, single-stream), not maxed out. For comparison **USB-A (USB2): 23.7 MB/s**;
> NVMe source alone: 562 MB/s. The true USB3 ceiling (fio/high-QD) was **not** characterized.
> - **USB-A (J8)** = **USB 2.0** (measured 480M; a USB3 stick falls back to High-Speed). This is
> **hardware-fixed**: J8 hangs on **port 2 of the CH334F = USB 2.0 hub** (chip table, U9) -> no driver ever lifts it.
> - **J16 (USB-C)** = 5 V sink only.
> - **Trade-off J4** (Type-C nature, not measured): 4-lane DP *or* USB3-SS + 2-lane DP *or* USB3-only.
>
> **Methodology note:** a **positive** capability result (device negotiates 10 G) is **hardware proof and
> permanent** - SuperSpeed negotiation is impossible without wired lanes, and it does not change on a rebuild.
> A **ceiling** (USB-A = USB2) is only reliable if the **schematic** proves it independently (here: the CH334F
> USB2 hub) - otherwise it would be just a build state. **The kernel is the test subject, never the upper bound.**

| Port | Data | Display | Power | PD controller |
|---|---|---|---|---|
| **USB-C #1** (J16, "USBC0") | USB 2.0 -> SoC **USB0** (OTG/DRD) | (SS pins unconnected) | **power input** (CC1/CC2 each **5.1 K** pulldown = pure sink) | **none** |
| **USB-C #2** (J4) | **USB 3.2 Gen2 (10 Gbps, COMB0, board-measured)** *or* USB 2.0 - muxed | **DP-Alt mode, 4 lanes** (COMB0) + AUX over SBU1/SBU2 | VBUS **output** (U2, EN = PL2) | **HUSB311** |

**USB-A:** exactly **one** (J8), USB 2.0, VBUS via U10 (EN = PM5).
**Hub CH334F:** upstream = SoC USB1; of 4 downstream ports **2** are used -> **port 1 = WiFi/BT module**,
**port 2 = USB-A**.

> **Why there are three USB2 PHYs although the DS lists only two USB2 controllers:**
> The datasheet names `1x USB2.0 DRD` + `1x USB2.0 Host` + `1x USB3.1 GEN2 DRD`. But the schematic shows
> three USB2 PHYs (USB0/USB1/USB2). Resolution: **USB2 is the USB 2.0 companion of the USB3.1 DRD controller.**
> Its SuperSpeed half hangs on **COMB0** (Type-C J4), not COMB1, and is **active**: measured 10 Gbps. So J4
> delivers real USB3-SS, muxed with DP-Alt.
> -> Assignment: **USB0 = USB2.0 DRD** (J16), **USB1 = USB2.0 Host** (hub -> USB-A), **USB2/USB3.1-DRD via COMB0** (J4).

### 7.5 Storage

| | Value | Evidence |
|---|---|---|
| **microSD** | socket **U7** on **SDC0** (bank PF), 4 bit, **card-detect PF6**. Bank PF is **dual-supplied** (`VCC-IO` 3.3 V **and** `VCC18-PF` 1.8 V) -> level switching for UHS possible. | p.13; DS pin table |
| **eMMC** | footprint **U8A = Samsung KLM8G1GETF-B041** (8 GB), **fully wired** to **SDC2** (bank PC): DAT0-7, CMD, CLK, RST, DS. VCCQ = **VCC-PC** (1.8 V, CLDO5), VCC = **VCC-eMMC** (3.3 V, SWOUT2). | p.9 |
| **EEPROM** | **U5 = BL24C16F** (2 KB) on **TWI0**; **WP via PC7** + MOSFET Q2 switchable. | p.16 |

### 7.6 eMMC - population is **not** resolved

The schematic provides for the chip and does **not** mark it DNP. But:
- **no photo** of the eMMC area (the 6 shots do not cover it), and
- **no usable statement from a running system**, because no driver binds there anyway
  (compatible mismatch - software level) - a device that does not probe looks identical
  whether populated or not.

**Resolve by:** visual inspection/photo of the board **or** DT fix + boot attempt.
Do **not** infer "not populated" from "does not appear" - exactly that fallacy was in the old docs.

### 7.9 IRQ-to-pin mapping (Przywara's open doubt - checked)

Przywara in the RFC: *"I am not 100% convinced the IRQ number to pin mapping ... works correctly."*
Checked against the **ground truth**: the per-pin table of the **vendor BSP driver** (`pinctrl-sun60iw2.c`,
`SUNXI_FUNCTION_IRQ_BANK(mux, irq_bank, eint)`), which **runs on the real board**.

**Result - the mapping is perfectly regular:**

| | |
|---|---|
| **irq_bank** | = sequential port index: **PB=0, PC=1, PD=2, PE=3, PF=4, PG=5, PH=6, PI=7, PJ=8, PK=9** -> **10 IRQ banks** |
| **eint number** | = **pin number within the bank** (PH16 -> eint16, PJ22 -> eint22, PK0 -> eint0) |
| **EINT mux** | = **`0xE` (Func14)** for all EINT-capable pins |
| **Exceptions** | **0 of 181** EINT-capable pins of the IP (`num != pin` nowhere) |

> **For Przywara:** his `.irq_banks = 10` and `irq_bank_muxes = {0,14,...}` match the vendor BSP exactly.
> The **logical** mapping (bank=port index, eint=pin number) is **trivially regular, without exception** -
> any formula-based approach reproduces it correctly as long as it assumes `eint == pin`. **The remaining
> risk is NOT in the mapping but in the register *location*** (in the new A733 layout the IRQ registers moved
> into the per-bank control block). The *number* question he raises is cleared.

### 7.8 GPIO banks: **IP size vs. bonded** - and a diff against mainline

Comparing the pin truth against **Andre Przywara's mainline pinctrl driver** (`a733-rfc`, RFC Aug 2025)
surfaced an apparent contradiction - which turned out to be an imprecision on our side:

| Bank | **GPIO IP** (live BSP driver + mainline) | **bonded** (DS + pinout V0.91) | Delta |
|---|---|---|---|
| PA | 0 | 0 | - |
| PB PC PD PE PF PG PK | 11 17 24 16 7 15 26 | identical | - |
| **PH** | **20** (PH0-19) | **17** (PH0-16) | +3 IP-only |
| **PI** | **17** (PI0-16) | **0** | **+17 = whole bank IP-only** |
| **PJ** | **28** (PJ0-27) | **6** (PJ22-27) | +22 IP-only |

**Triple-confirmed** (3 independent sources, same numbers):
1. Przywara's **mainline** driver `pinctrl-sun60i-a733.c`: `{0,11,17,24,16,7,15,20,17,28,26}`
2. **Vendor BSP** driver, read **live on the board** (`/sys/kernel/debug/pinctrl/2000000.pinctrl/pins`): PH=20, PI=17, PJ=28
3. Our DS/pinout extraction (the bonded numbers).

> **Resolution:** the GPIO **IP** has **181 pins** in the **main domain** (PB-PK) (incl. **bank PI**, larger PH/PJ);
> of those **139 are bonded**. Plus the **CPUS domain** PL(14)+PM(6) = 20 -> **159 bonded total** (the value in Sec. 2.1).
> On the **package** only the bonded pins appear in datasheet + pinout; the IP-only pins (PI entirely, PH17-19, PJ0-21) do not.
> - The earlier statement *"there is no bank PI"* was right at the **bonding** level, wrong at the **IP** level.
> Correct: **bank PI exists in silicon (17 pins) but is not brought out on the A733 package** ->
> **not usable** on this board, but it *exists*.
> - `PH17/18/19` and `PJ0-21` are likewise **IP-only, unbonded** -> not on the board.
>
> **Przywara's bank sizes are correct** - they describe the IP (as a pinctrl driver must) and **match the
> vendor BSP exactly, verified live on the board.** His RFC uncertainty ("not 100% convinced the IRQ number
> to pin mapping works") concerns the **IRQ mapping**, not the bank sizes - those are right.

### 7.7 Ethernet PHY

- PHY **MAE0621A-Q3C** with **own 25 MHz crystal**; RGMII on bank **PH**; **I/O voltage 3.3 V**
  (VCCIO_PHY <- SWOUT1; SoC bank PH <- SWOUT2 - both sides 3.3 V, consistent).
- The PHY generates its own 1.0 V (`PHY_REG_OUT` -> L11 2.2 uH).
- **RGMII delay / PHYAD straps** (R115-R125): derived from the drawing: PHYAD = 1/0/0,
  **RXDLY = pull-up, TXDLY = pull-down**. **Not verified against the MAE0621A datasheet** -
  cross-check on the board before changing a delay in the driver.

---

## 8. Open points - where the documentation is honestly unfinished

### 8.1 RESOLVED: BLDO2 = 3.3 V is **Allwinner's own reference intent**

> **Resolved by Allwinner's STD reference design** (`A733_STD_AXP318_LPDDR5_FBGA315BALL V1.0`,
> 2025-03-28, p.4 "POWER SEQUENCE"). It states **verbatim**:
> ```
> VCC-PD / VCC-LVDS0 / VCC-PJ / VCC18-LCD EXT DCDC-> BLDO2(0.5A) -> 3.3 V
> ```
> **The 3.3 V is not a Radxa error, not an OTP accident, not a broken blob.** It is **Allwinner's
> documented reference configuration**, which Radxa copied **exactly** - including the misleading net
> name `VCC18-LCD`, which actually carries 3.3 V.

**The node really sits at 3.3 V - backed two ways:**
1. **BLDO2 OTP register** `0x2B` = `0x1C` = **3300 mV** (read raw, bypassing the driver - no decode bug, BLDO uses *one* linear range). **ON** (enable `0x20` bit 7).
2. **R251 is fitted** (placement map, Sec. 8.1b) -> the node also hangs hard on **SWOUT1 = real 3.3 V** (load switch from DCDC1, *not* an LDO). Both sources = 3.3 V.

-> **`VCC-LVDS` (ball 1N6) therefore really sees 3.3 V** - not ~2.4 V. An earlier relief ("the LDO caps at 2.4 V") was **wrong**: it held only for the case *R251 unfitted*, which is not the case here. The node is genuine 3.3 V.

**The contradiction is internal to Allwinner:**

| Allwinner source | `VCC-LVDS` |
|---|---|
| Datasheet Table 5-2 (typ) | **1.8 V** |
| Datasheet Table 5-1 (abs-max) | **2.1 V** |
| **STD reference schematic** p.4 | **3.3 V** (BLDO2) |

Allwinner drives, in their **own** reference design, a pin at 3.3 V that their **own** datasheet rates at 2.1 V abs-max. **Their** contradiction, not ours.

> **Why it does not bite on the A7S** (despite real 3.3 V on a 2.1 V pin):
> 1. **LVDS is not routed** (Sec. 2.2) - an unused, unpowered input.
> 2. It is **Allwinner's documented reference configuration** (p.4), not an oversight - Radxa copied it 1:1.
> 3. **Millions of A733 run this way** -> the 2.1 V row is apparently conservative, or applies only to *active* LVDS operation (1.8 V signaling).
>
> **Do NOTHING.** Do not measure (the source is unambiguously SWOUT1), do not reprogram in the DT:
> setting BLDO2 to 1.8 V in the DT is **ineffective** - SWOUT1 holds the node at 3.3 V via the fitted R251,
> and BLDO2 would only sink against it. **Changeable only in hardware** (remove R251), not by software.

**How the 3.3 V gets into the register - the mechanism (for Sec. 4.6):**
Linux does **not** set the 3.3 V. `reg_bldo2` has **no** `regulator-init-microvolt` (0x in the whole board DTS) and **no** `min==max`, only the wide range `500000`-`3400000`; `pcie3v3-supply` only *enables* it. -> The value comes from **OTP**, Linux **inherits** it unchecked. Same mechanism as VDD2L/VCC-VDDQ -> **rule Sec. 4.6**: constraints must reflect the HW window so that an OTP value outside it is even noticed.

> **Side finding (same register dump): `BLDO1` is OFF** (enable `0x20` bit 6 = 0) -> `VCC-PE`/`VCC-PK`/`VCC-MCSI` **unpowered**, banks **PE/PK dead** (consistent with the camera off). **To enable the camera => turn BLDO1 on first** - the DT node alone is not enough.

### 8.1b RESOLVED: the IO-bank rails are 3.3 V (R251 fitted)

| Bank rail | Source p.4 | Source p.7 | **Resolution** |
|---|---|---|---|
| `VCC-PD` | BLDO2 | **VCC33-LCD via R251 (0 ohm, FITTED)** | **3.3 V from SWOUT1** |
| `VCC-PJ` | BLDO2 | VCC33-LCD | **3.3 V** (same node) |
| `VCC-PM` | CLDO1 | VCC-PL via RA13 (0 ohm) | check RA13 population analogously |

**R251 is fitted** ([Radxa placement map V1.10](https://dl.radxa.com/cubie/a7s/hw/radxa_cubie_a7s_components_placement_map_v1.10.pdf)). The whole BLDO2 node is thus bound **hard to SWOUT1 = DCDC1
-> 3.3 V**. **SWOUT1 is a load switch, not an LDO - it delivers real 3.3 V.** This was not a contradiction
but **redundancy**: the Allwinner reference sets BLDO2 = 3.3 V, Radxa *additionally* ties the same node to
SWOUT1. Both sources = 3.3 V, no conflict at the target.

> **This settles the logic level of the PCIe GPIOs PD20-PD23: 3.3 V.** (Banks PD/PJ abs-max 3.63 V -> ok.)
> The three earlier clues (0-ohm bridges, I2C pull-ups to `VCC33-LCD`, header nets `PJ24_3V3`) were
> **all correct** - now hard-backed instead of assumed.

**RA13 (VCC-PM) is the OPPOSITE case - DNP.** Unlike R251:
- `VCC-PM` is on **CLDO1 = 1.8 V** (p.4; register `0x2F`=`0x0D`, live ON) **and** via RA13 on `VCC-PL` = 3.3 V (p.7).
- If RA13 were fitted, that would be a **1.8/3.3 V short via 0 ohm**. Since CLDO1 is fixed at 1.8 V and the
  board runs, **RA13 must be unfitted (DNP)** -> **VCC-PM = 1.8 V, clean.**
- Confirmed by the placement map: RA13 is shown **gray** (= DNP), R251 **black** (= fitted).

> **The map encodes the intent exactly:** R251 fitted (both sources 3.3 V, no conflict), RA13 DNP
> (1.8 V vs 3.3 V would collide). Both IO-bank dual sources are thus resolved - without a multimeter.

### 8.2 The DDR-PARA strap contradicts itself in the schematic

The fitted resistors give GPIO = 0 and 382 mV -> **DDR PARA 2**. But in the same table **`DDR PARA 11`
is marked blue** (GPIO = 1, 608 mV / 5.1 K) - and blue is consistently the "selected" marker in this
schematic (for COMBO1 and BOARD-ID it matches the population). **This contradiction remains** - a
documentation curiosity (an editorial error in Allwinner's strap table, or Radxa populated differently
on purpose).

For a family kernel the strap is not the path to the RAM size anyway: the size is determined by DRAM
training in the boot loader (Sec. 3), independent of this strap table.

### 8.2b UFS: schematic and datasheet name the same rails **oppositely**

In a document whose purpose is the mapping, this cannot stand unresolved:

| Schematic net | Voltage | **DS pin name** | **DS meaning** |
|---|---|---|---|
| `VCC-UFS` | 2.5 V (DLDO6) | `VCC-UFS` | UFS-3.x **IO** |
| **`VCC-UFS-IO`** | **1.2 V** (DCDC8) | `VCC12-UFS` | **UFS-3.x CORE** |
| `VDD08-UFS` | 0.8 V (DCDC2) | `VDD-UFS` | UFS **controller** |

> **The net that carries "IO" in its name is, per the datasheet, the CORE.**
> `VCC-UFS-IO` carries **1.2 V** - and 1.2 V is, in the DS, the value of **`VCC12-UFS` = CORE**, not of the
> IO rail (which is 2.5 V). **The schematic net name is misleading.**
> Consistent with this, the schematic note at the DC8SET strap: *"1.2V FOR UFS 3.x / 1.8V FOR UFS 2.2"* -
> that is the **core** voltage of a UFS-3.x device.
>
> **Additional DS-internal inconsistency:** in the **ball table** there is **no pin named `VCC12-UFS`** -
> there is only `VCC-UFS` (ball **1V20**) and `VDD-UFS` (ball **1V22**). `VCC12-UFS` appears **only** in
> Tables 5-1/5-2. The DS is at odds with itself here. **Practically harmless (no UFS fitted)** -
> but anyone bringing UFS up must clear this first.

### 8.2c Curious but datasheet-backed: two rails **with no margin**

| Rail | **typ** (Table 5-2) | **abs-max** (Table 5-1) | Margin |
|---|---|---|---|
| `VCC12-UFS` (UFS CORE) | **1.2 V** | **1.2 V** | **0%** |
| `VCC-UFS` (UFS IO) | **2.5 V** | **2.5 V** | **0%** |

> For both, the **recommended operating value equals the absolute maximum**. There is **no room upward** -
> not one millivolt. Practically harmless, because **no UFS is fitted**. Noted here so nobody later thinks
> *"there is headroom"*.

### 8.3 CLARIFIED - `vdd_sys` is a DT fiction, not hardware

From the **DTS source** (not the board):
```
sun60iw2p1.dtsi:272 reg_vdd_sys: vdd-sys {
                        compatible = "regulator-fixed"; <-- not a PMIC rail!
                        regulator-min/max-microvolt = <900000>;
sun60i-a733-cubie-a7s.dts:1460 npu-supply = <&reg_dcdc2>;
```
- On the board there is **no fixed 900 mV regulator**. `vdd_sys` is a **`regulator-fixed` placeholder**
  in the SoC dtsi that `dmcfreq` uses as `vddcore`.
- The **real** VDD-SYS rail is **DCDC2** (adjustable, with remote sense `VDD-SYSFB`, schematic p.4/p.8).
- The DT declares DCDC2 only as **`npu-supply`** - although per schematic DCDC2 feeds **VDD-SYS, VDD-DRAM,
  VDD-VE, VDD-COMB0/1, VDD08-HDMI/UFS**, far more than the NPU.
- **-> a DT bug, not a hardware puzzle.** For a family kernel: model DCDC2 correctly as a system rail,
  do not treat the fictional `vdd_sys` as a real voltage source.

### 8.4 Formerly open minor points - both resolved

- **Bank PF is DUAL, not 1.8 V only.** The datasheet pin table lists `io_supply = VCC18-PF/VCC-IO` for
  **all** Port-F pins (compare Port B: only `VCC-IO`). Datasheet and schematic (two supply pins 1U5/1U6)
  thus agree -> **3.3 V and 1.8 V**, as needed for SD level switching.
  The earlier claim "PF = 1.8 V only" was an error in the DS extract.
- **The crystal is 26 MHz.** On the board photo the crystal next to the SoC legibly reads
  **`26.000 MHz`**. The "24.000 MHz" note in the old inventory was a misread.
  Schematic (X1 = 26 MHz), CK-SEL strap (SEL1=1/SEL0=0 -> "SOC=26M") and photo all **agree**.

### 8.5 DS-internal contradictions (not smoothed over)

- Ball size 0.3 mm (p.1/41) vs 0.25 mm (p.138);
  "L2 265 KB" (block diagram) vs "256 KB" (Sec. 3.1); "A56 core" (DS) vs "CPUL/A55" (UM);
  UM Fig. 18-8 contradicts Fig. 18-3 + footnotes [8]/[9] on the USB3 assignment (the latter are authoritative).

### 8.6 Pitfalls when reading the documents

- **IRQ conversion:** UM IRQ numbers are absolute GIC numbers incl. SGI+PPI -> **`GIC_SPI n = IRQ - 32`**
  (UART0: IRQ 34 -> SPI 2).
- **Name clash "COMBOPHY":** in the display chapter the MIPI-DSI/LVDS D-PHYs are named the same
  (`0x05506000`) - they have **nothing** to do with the SerDes combo PHYs (`0x06C80000`).
- **IO voltage** is set **solely by the rail voltage** - there is **no level-select pin**.

---

## Sources

All source documents are public. Direct links below.

| Source | What it provides |
|---|---|
| **[Radxa Cubie A7S schematic V1.10](https://dl.radxa.com/cubie/a7s/hw/radxa_cubie_a7s_schematic_v1.10.pdf)** (board RS504, 2026-02-03, 16 pp.) | board nets, population, straps |
| [A733 datasheet V0.93](https://dl.radxa.com/cubie/a7a/docs/hw/datasheet/A733_Datasheet_V0.93.pdf) | SoC pins, abs-max (Table 5-1), Recommended Operating Conditions (Table 5-2) |
| [A733 User Manual V0.92](https://gitlab.com/tina5.0_aiot/product/docs) | CCU/PLL reference, register bases, thermal sensors |
| [AXP318W datasheet V0.1 (draft)](https://gitlab.com/tina5.0_aiot/product/docs) | PMIC DVFS mechanics |
| Chip photos (population evidence) | own shots of the board (`img/`) |
| [Allwinner STD reference schematic V1.0](https://gitlab.com/tina5.0_aiot/product/docs) (2025-03-28, 34 pp.) | AXP318+LPDDR5+315BALL = same config as A7S. p.4 "POWER SEQUENCE" = rail-regulator-voltage, Allwinner's original intent. Resolves BLDO2 (Sec. 8.1). |
| AXP323 V1.1 datasheet + [`a733-rfc` driver](https://github.com/apritzel/linux/tree/a733-rfc) (Andre Przywara, ARM) | independent confirmation of the AXP318 register encoding |
| Micron + Rayson LPDDR5 datasheets | DRAM-die rails (VDD1/VDD2H/VDD2L/VDDQ, JEDEC nominals) |

### Missing sources - known gaps, not omissions

| Missing source | What we lack because of it |
|---|---|
| **LPDDR5 device datasheet** (manufacturer unknown - the chip marking does not name it, Sec. 1.1) | spec for **all DRAM-die rails**: `VDD1`, `VDD2H`, `VDD2L`, `VDDQ` - nominal, tolerance, abs-max. We have **only the schematic** for these. |
| **JEDEC JESD209-5 (LPDDR5)** | the normative reference for the same rails + timing. Not available. |
| **MAE0621A datasheet** | verification of the RGMII delay / PHYAD straps (Sec. 7.7) - currently only derived from the drawing. |

> As long as these are missing, statements about DRAM-die rails are **schematic-backed but not spec-backed.**
> That is a property of the source situation, not a lack of care.

**Schematic page index:** 1 cover, 2 index, 3 block, **4 POWER AXP318**, **5 LPDDR5**,
**6 analog & high-speed (straps, USB/eDP/PCIe, UFS, HDMI)**, **7 GPIO (pinmux)**, **8 SYS + CORE POWER**,
9 eMMC, 10 camera, 11 WiFi/BT, **12 Type-C (DP-Alt)**, 13 USBC0 + T-card + USB hub,
**14 header + PCIe**, **15 Ethernet**, 16 key/LED/fan/EEPROM.

---
*Method: extraction from datasheet/UM + visual tracing of all 16 schematic pages (400-600 dpi, crop
magnification). A running board served only as an aid for confirmation, never as truth. Where sources
contradict, the contradiction is stated in Sec. 8 rather than smoothed into a single claim.*
