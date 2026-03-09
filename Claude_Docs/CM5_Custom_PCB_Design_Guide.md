# Complete Guide: Custom CM5 Carrier Board PCB Design
## Using Raspberry Pi CM5IO Reference + Waveshare CM5-NANO-B as Inspiration
### KiCad on Windows — Beginner-Friendly Edition

---

## TABLE OF CONTENTS

1. [What You're Building](#1-what-youre-building)
2. [Downloads & Tool Setup (Windows)](#2-downloads--tool-setup-windows)
3. [Understanding the Reference Designs](#3-understanding-the-reference-designs)
4. [KiCad Crash Course for Beginners](#4-kicad-crash-course-for-beginners)
5. [Starting Your Custom Project](#5-starting-your-custom-project)
6. [Schematic Design — Sheet by Sheet](#6-schematic-design--sheet-by-sheet)
7. [PCB Layout Strategy for Compactness](#7-pcb-layout-strategy-for-compactness)
8. [High-Speed Design Rules](#8-high-speed-design-rules)
9. [Design Rule Check & Validation](#9-design-rule-check--validation)
10. [Manufacturing Output](#10-manufacturing-output)
11. [Assembly & Testing](#11-assembly--testing)
12. [Common Mistakes & Troubleshooting](#12-common-mistakes--troubleshooting)
13. [Resource Links & Learning Path](#13-resource-links--learning-path)

---

## 1. WHAT YOU'RE BUILDING

You're designing a **custom carrier board** for the Raspberry Pi Compute Module 5 that combines:

| Feature | Details |
|---------|---------|
| **CM5 Module Mount** | 2x Hirose DF40C-100DS-0.4V(51) connectors |
| **Silicon Labs EFR32MG24** | Zigbee 3.0 / Thread / Matter radio (SPI interface) |
| **BME280 Sensor** | Temperature, humidity, pressure (I2C interface) |
| **NVMe SSD** | M.2 2230 slot via PCIe Gen2 x1 |
| **Single HDMI** | Keep HDMI0 only, remove HDMI1 |
| **USB** | 1x USB 2.0 or USB 3.0 for debug/provisioning |
| **Power** | USB-C 5V input |

**What we're removing** (compared to CM5IO reference board):
- ❌ Second HDMI port (HDMI1)
- ❌ DSI display connector
- ❌ CSI camera connector  
- ❌ Full-size SD card slot (CM5 has eMMC)
- ❌ Composite video / 3.5mm audio
- ❌ Extra USB ports
- ❌ PCIe FFC connector (direct M.2 instead)

**Target board size**: ~55mm × 40mm (same footprint as CM5 module itself, like the Waveshare CM5-NANO-B)

---

## 2. DOWNLOADS & TOOL SETUP (WINDOWS)

### 2.1 Install KiCad 8.x on Windows

1. Go to **https://www.kicad.org/download/windows/**
2. Download the latest **KiCad 8.x** stable installer (64-bit)
3. Run the installer → select "Full Installation" (includes all libraries)
4. Default install path: `C:\Program Files\KiCad\8.0\`
5. Launch KiCad from Start Menu

### 2.2 Install Essential KiCad Plugins

Open KiCad → **Plugin and Content Manager** (button on main screen):

| Plugin | Purpose |
|--------|---------|
| **Interactive HTML BOM** | Generates interactive BOM for assembly |
| **KiCad JLCPCB Tools** | Auto-generates JLCPCB fab/assembly files |
| **Teardrops** | Improves pad-to-trace connections (reliability) |

### 2.3 Download Reference Design Files

These are your most critical resources. Download ALL of these before starting:

#### A) Raspberry Pi CM5 IO Board KiCad Files (YOUR PRIMARY REFERENCE)

**Download from Raspberry Pi Product Information Portal:**
- URL: `https://pip.raspberrypi.com/categories/1098-design-files`
- File: **"CM5 IO Board, revision 2, KiCAD files"** (~3.4 MB ZIP)
- Also download: **"rpi-cm5io Kicad"** (~3.4 MB ZIP)

These files are created in **KiCad 8** and contain:
- Complete schematic (hierarchical sheets)
- Full PCB layout with all layers
- Custom symbol library (`CM5IO.kicad_sym`)
- Custom footprint library
- 3D models directory (`CM5IO.3shapes`)

**Extract to**: `C:\Users\YourName\Documents\KiCad\CM5IO-Reference\`

#### B) Waveshare CM5-NANO-B Schematic (YOUR SIZE REFERENCE)

- URL: `https://www.waveshare.com/wiki/CM5-NANO-B`
- Scroll to "Resources" section → download:
  - **CM5-NANO-B Schematic diagram** (PDF)
  - **CM5-NANO-B 3D** (STEP file)
- NOTE: Waveshare provides **PDF schematics only** (not editable KiCad files), so use this as a visual reference for how they achieved the compact CM5-sized form factor

**Extract to**: `C:\Users\YourName\Documents\KiCad\CM5-NANO-B-Reference\`

#### C) Shawn Hymel's CM5 Carrier Template (OPTIONAL BUT HELPFUL)

- URL: `https://github.com/ShawnHymel/rpi-cm4-carrier-template`
- Click **Code → Download ZIP**
- This is a CM4 template but the connector footprints and hierarchical sheet structure are useful learning material
- Move the `hardware` folder into your KiCad base folder: `C:\Users\YourName\Documents\KiCad\`

#### D) CM5 MINIMA Open-Source Project (EXCELLENT REAL-WORLD REFERENCE)

- URL: `https://github.com/piecol/CM5_MINIMA_REV2`
- This is a **working, certified open-source** compact CM5 carrier board designed in KiCad
- Has HDMI, USB-C PD, Ethernet, sensor, and radio — very similar to your project
- Download the full repo and study the KiCad files

#### E) Datasheets You Need

| Document | Where to Get It |
|----------|-----------------|
| CM5 Datasheet | `https://datasheets.raspberrypi.com/cm5/cm5-datasheet.pdf` |
| CM5IO Datasheet | `https://datasheets.raspberrypi.com/cm5/cm5io-datasheet.pdf` |
| CM5 3D Model (STEP) | `https://datasheets.raspberrypi.org/cm5/CM5-step.zip` |
| EFR32MG24 Datasheet | Silicon Labs website → search "EFR32MG24B220F1536IM48" |
| EFR32MG24 Reference Design | Silicon Labs Simplicity Studio → Hardware Examples |
| BME280 Datasheet | Bosch Sensortec website |
| Hirose DF40C-100DS Datasheet | Hirose or DigiKey |

### 2.4 Create Your Working Folder Structure

On Windows, create this structure:
```
C:\Users\YourName\Documents\KiCad\
├── CM5IO-Reference\          ← Extracted RPi CM5IO KiCad files
├── CM5-NANO-B-Reference\     ← Waveshare schematic PDF + 3D
├── CM5-MINIMA-Reference\     ← CM5 MINIMA REV2 repo
├── Datasheets\               ← All PDFs organized here
│   ├── cm5-datasheet.pdf
│   ├── cm5io-datasheet.pdf
│   ├── EFR32MG24.pdf
│   └── BME280.pdf
└── Projects\
    └── cm5-ha-controller\    ← YOUR NEW PROJECT (created in Step 5)
```

---

## 3. UNDERSTANDING THE REFERENCE DESIGNS

### 3.1 Study the CM5IO Board First (Critical Step!)

Before you design anything, spend 2-3 hours studying the reference:

1. Open `CM5IO.kicad_pro` in KiCad
2. Open the **Schematic Editor** — it uses hierarchical sheets:
   - Root sheet with CM5 connector pinout
   - Sub-sheets for HDMI, USB, PCIe, GPIO, Power, etc.
3. Open the **PCB Editor** — study:
   - How differential pairs are routed (HDMI, USB, PCIe)
   - Ground plane continuity
   - Decoupling cap placement
   - Connector positioning

**Take notes on:**
- Which pins map to HDMI0 vs HDMI1 (you're keeping 0, removing 1)
- Which pins map to PCIe (for your NVMe)
- Which GPIO pins are available for SPI/I2C (for your EFR32 and BME280)
- How power distribution works (5V input → CM5)

### 3.2 Study the Waveshare CM5-NANO-B Schematic

The CM5-NANO-B PDF schematic shows you:
- How Waveshare achieved CM5-sized form factor (~55×40mm)
- Their power management approach (simpler than CM5IO)
- Mini-HDMI connector usage (saves ~50% space vs full-size)
- PCIe Gen2 x1 connector layout
- Ethernet PHY integration in compact space
- Where they placed the Hirose connectors relative to edge connectors

**Key insight from CM5-NANO-B**: It uses a 16-pin PCIe Gen2/3 x1 connector (not full M.2), which you can replace with an actual M.2 2230 slot. This will extend the board slightly but gives native NVMe support.

### 3.3 What to Copy vs What to Customize

| From CM5IO Reference | Action |
|----------------------|--------|
| CM5 Hirose connector schematic/pinout | **COPY EXACTLY** — do not modify |
| 5V power input + protection | **COPY** (simplify if using USB-C only) |
| HDMI0 circuit (AC coupling, ESD) | **COPY EXACTLY** |
| PCIe lane connections | **COPY** but route to M.2 slot instead of FFC |
| USB 2.0/3.0 circuit | **COPY** one port |
| HDMI1 circuit | **DELETE** |
| DSI/CSI connectors | **DELETE** |
| SD card slot | **DELETE** |
| GPIO header | **MODIFY** — keep SPI/I2C pins for your chips |

| New Additions | Source |
|---------------|--------|
| Silicon Labs EFR32MG24 | Design from Silicon Labs reference schematics |
| BME280 sensor | Design from Bosch datasheet |
| M.2 2230 NVMe slot | Design from M.2 specification + CM5IO PCIe pinout |

---

## 4. KICAD CRASH COURSE FOR BEGINNERS

### 4.1 KiCad Project Structure (Windows)

When you create a KiCad project, it creates these files:
```
cm5-ha-controller\
├── cm5-ha-controller.kicad_pro    ← Project file (open this)
├── cm5-ha-controller.kicad_sch    ← Root schematic
├── cm5-ha-controller.kicad_pcb    ← PCB layout
├── libraries\                      ← Your custom libraries
│   ├── cm5-ha-controller.kicad_sym ← Custom symbols
│   └── cm5-ha-controller.pretty\   ← Custom footprints
└── 3dmodels\                       ← 3D models
```

### 4.2 Essential KiCad Shortcuts (Learn These!)

**Schematic Editor:**
| Key | Action |
|-----|--------|
| `A` | Add/place component (symbol) |
| `P` | Place power symbol (GND, VCC, etc.) |
| `W` | Draw wire |
| `L` | Place net label |
| `Ctrl+L` | Place global label |
| `E` | Edit component properties |
| `M` | Move component |
| `R` | Rotate component |
| `C` | Copy component |
| `Delete` | Delete selected |
| `Ctrl+Z` | Undo |
| `Ctrl+S` | Save |

**PCB Editor:**
| Key | Action |
|-----|--------|
| `X` | Route track |
| `/` | Toggle single/differential pair mode |
| `V` | Place via (while routing) |
| `D` | Drag component |
| `B` | Fill/refill all zones (copper pours) |
| `F.Cu` / `B.Cu` | Switch active layer |
| `I` | Select net inspector |
| `Ctrl+Shift+D` | Run DRC |
| `Space` | Reset local coordinates |

### 4.3 Understanding KiCad's Schematic → PCB Workflow

```
Step 1: Draw Schematic
    ↓
Step 2: Run ERC (Electrical Rules Check)
    ↓
Step 3: Assign Footprints to Symbols
    (Tools → Assign Footprints)
    ↓
Step 4: Update PCB from Schematic
    (Tools → Update PCB from Schematic)
    ↓
Step 5: Place Components on PCB
    ↓
Step 6: Route Traces
    ↓
Step 7: Add Copper Zones (Ground/Power pours)
    ↓
Step 8: Run DRC (Design Rules Check)
    ↓
Step 9: Generate Manufacturing Files (Gerber, Drill)
```

---

## 5. STARTING YOUR CUSTOM PROJECT

### 5.1 Create New KiCad Project

1. Open KiCad
2. **File → New Project**
3. Navigate to `C:\Users\YourName\Documents\KiCad\Projects\`
4. Name: `cm5-ha-controller`
5. Click Save

### 5.2 Import CM5IO Libraries Into Your Project

This is the most important setup step. You need the CM5 connector symbols and footprints from the reference design.

**Step 1: Copy library files**
1. From the extracted CM5IO reference folder, find:
   - `CM5IO.kicad_sym` (symbol library)
   - `CM5IO.pretty/` folder (footprint library)
   - `CM5IO.3shapes/` folder (3D models)
2. Copy these into your project's `libraries\` folder:
```
cm5-ha-controller\
└── libraries\
    ├── CM5IO.kicad_sym
    ├── CM5IO.pretty\
    └── CM5IO.3shapes\
```

**Step 2: Add symbol library to KiCad**
1. In Schematic Editor: **Preferences → Manage Symbol Libraries**
2. Click the **Project Specific Libraries** tab
3. Click **Add existing library to table** (folder icon)
4. Browse to `libraries\CM5IO.kicad_sym`
5. Nickname: `CM5IO`
6. Click OK

**Step 3: Add footprint library to KiCad**
1. In PCB Editor: **Preferences → Manage Footprint Libraries**
2. Click the **Project Specific Libraries** tab
3. Click **Add existing library to table**
4. Browse to `libraries\CM5IO.pretty`
5. Nickname: `CM5IO`
6. Click OK

**Step 4: Download additional symbols/footprints you'll need**

Go to **SnapEDA** (snapeda.com) or **Ultra Librarian** and download KiCad format files for:

| Component | What to Download |
|-----------|-----------------|
| EFR32MG24B220F1536IM48 | Symbol + QFN48 footprint |
| BME280 | Symbol + LGA-8 footprint |
| M.2 2230 connector (M-key) | Symbol + footprint |
| Mini-HDMI receptacle | Symbol + footprint |
| USB-C receptacle (16-pin) | Symbol + footprint |
| AP2112K-3.3TRG1 (LDO) | Symbol + SOT-23-5 footprint |
| 2450AT18x100 (chip antenna) | Footprint from Johanson datasheet |

**For each download:**
1. Extract the `.kicad_sym` file → copy to `libraries\`
2. Extract the `.kicad_mod` file → copy to `libraries\cm5-ha-controller.pretty\`
3. Add to KiCad via Manage Libraries (same process as above)

### 5.3 Set Up Hierarchical Schematic Sheets

Hierarchical sheets keep complex designs organized. Your project should have these sheets:

**In the Root Schematic:**
1. Right-click on empty space → **Add Hierarchical Sheet**
2. Create these sheets one by one:

```
Root (cm5-ha-controller.kicad_sch)
│
├── CM5_Connectors.kicad_sch       ← CM5 Hirose connectors + all pin labels
│
├── Power_Supply.kicad_sch          ← USB-C input, 3.3V/1.8V regulators, decoupling
│
├── HDMI0_Output.kicad_sch          ← Single HDMI port (copy from CM5IO reference)
│
├── USB_Port.kicad_sch              ← USB 2.0 or 3.0 port
│
├── NVMe_PCIe.kicad_sch             ← M.2 2230 slot with PCIe routing
│
├── Zigbee_EFR32.kicad_sch          ← Silicon Labs EFR32MG24 + RF matching + antenna
│
├── BME280_Sensor.kicad_sch         ← Environmental sensor + I2C
│
└── Misc_GPIO_LEDs.kicad_sch        ← Status LEDs, buttons, debug header
```

**How to create a hierarchical sheet in KiCad:**
1. In root schematic, press `S` or go to **Place → Hierarchical Sheet**
2. Draw a rectangle
3. In the dialog: set Sheet name (e.g., "CM5_Connectors") and File name (e.g., "CM5_Connectors.kicad_sch")
4. Double-click the sheet rectangle to open and edit it
5. Use **Global Labels** (`Ctrl+L`) to pass signals between sheets (e.g., `SPI0_MOSI`, `I2C1_SDA`, `HDMI0_TX0P`)

---

## 6. SCHEMATIC DESIGN — SHEET BY SHEET

### 6.1 Sheet: CM5_Connectors (Copy from CM5IO Reference)

This is the foundation. Open the CM5IO reference schematic and **copy the two Hirose connector symbols with ALL pin labels exactly**.

**Two connectors:**
- **J1**: Hirose DF40C-100DS-0.4V(51) — primarily high-speed signals
- **J2**: Hirose DF40C-100DS-0.4V(51) — primarily GPIO, SD, power

**Critical signals you'll use:**

From J1 (verify exact pin numbers against CM5 datasheet):
```
PCIe:
  PCIe_TX_P / PCIe_TX_N       → to M.2 NVMe
  PCIe_RX_P / PCIe_RX_N       → from M.2 NVMe
  PCIe_REFCLK_P / PCIe_REFCLK_N → to M.2 NVMe
  
HDMI0:
  HDMI0_TX0_P / HDMI0_TX0_N   → to HDMI connector
  HDMI0_TX1_P / HDMI0_TX1_N   → to HDMI connector
  HDMI0_TX2_P / HDMI0_TX2_N   → to HDMI connector
  HDMI0_CLK_P / HDMI0_CLK_N   → to HDMI connector
  HDMI0_SDA, HDMI0_SCL        → DDC (I2C for EDID)
  HDMI0_CEC, HDMI0_HPD        → Control signals
  
USB:
  USB_DP / USB_DN              → to USB connector
  (USB3 signals if using USB 3.0)

Power (ACTIVE — connect ALL of these):
  5V pins (multiple)           → from your 5V supply
  GND pins (multiple)          → to ground plane
```

From J2:
```
GPIO for your custom chips:
  GPIO2 (I2C1_SDA)            → to BME280 SDA
  GPIO3 (I2C1_SCL)            → to BME280 SCL
  GPIO7 (SPI0_CE1)            → to EFR32 CS
  GPIO9 (SPI0_MISO)           → from EFR32 MISO
  GPIO10 (SPI0_MOSI)          → to EFR32 MOSI
  GPIO11 (SPI0_SCLK)          → to EFR32 SCLK
  GPIO17                       → to EFR32 nRESET
  GPIO27                       → from EFR32 IRQ
  
Signals to leave UNCONNECTED:
  HDMI1_* (all HDMI1 pins)    → NC (mark as No Connect ✕)
  DSI/CSI signals              → NC
  SD_* signals                 → NC (using eMMC on CM5)
```

**CRITICAL RULES:**
- Connect EVERY GND pin (mark with GND power symbol)
- Connect EVERY 5V/VBUS pin (mark with +5V power symbol)
- Place a "No Connect" flag (press `Q` in KiCad) on every unused pin — this prevents ERC errors

### 6.2 Sheet: Power_Supply

**Input: USB-C connector providing 5V**

```
Design:

USB-C (5V in) ──[Polyfuse/PTC 3A]──┬── +5V rail (to CM5, USB VBUS)
                                     │
                                     ├── [10µF + 100nF ceramic] → GND
                                     │
                                     ├── AP2112K-3.3 LDO ──→ +3.3V rail
                                     │   └── [10µF in + 10µF out + 100nF out] 
                                     │
                                     └── AP2112K-1.8 LDO ──→ +1.8V rail (if needed by EFR32)
                                         └── [10µF in + 10µF out + 100nF out]
```

**KiCad schematic steps:**
1. Place USB-C connector symbol
2. Place AP2112K-3.3 symbol (SOT-23-5)
3. Wire VIN to +5V through polyfuse
4. Wire VOUT to +3.3V net
5. Wire EN to VIN (always enabled)
6. Add input/output capacitors
7. Add power flags: `PWR_FLAG` on +5V and GND nets (prevents ERC warnings)

**Bulk capacitance near CM5:**
- 2× 100µF electrolytic on +5V (CM5 can draw 2-3A peaks)
- 4× 10µF ceramic on +5V
- 100nF ceramic on every IC power pin (placed in respective sheets)

### 6.3 Sheet: HDMI0_Output (Copy from CM5IO)

**Copy this section directly from the CM5IO reference schematic.** It contains:

1. **AC coupling capacitors** (100nF, 0402) on each TMDS differential pair
2. **ESD protection** (TPD12S016 or similar) — CRITICAL for HDMI certification
3. **HDMI connector** pinout
4. **5V supply** to HDMI (through current-limiting switch)
5. **DDC I2C** (SDA/SCL with pullups)
6. **CEC** and **HPD** (Hot Plug Detect) connections

**For compactness: Use Mini-HDMI** connector instead of full-size HDMI (saves ~15mm width).

Wire mapping is the same — just use a mini-HDMI footprint.

### 6.4 Sheet: NVMe_PCIe

**M.2 2230 M-key connector connections:**

```
From CM5 (via Hirose connector):
  PCIe_TX_P  ──[100nF AC cap]──→  M.2 Pin 41 (PETp0)
  PCIe_TX_N  ──[100nF AC cap]──→  M.2 Pin 43 (PETn0)
  PCIe_RX_P  ←──────────────────  M.2 Pin 47 (PERp0)
  PCIe_RX_N  ←──────────────────  M.2 Pin 49 (PERn0)
  PCIe_CLK_P ──[100nF AC cap]──→  M.2 Pin 53 (REFCLKp0)
  PCIe_CLK_N ──[100nF AC cap]──→  M.2 Pin 55 (REFCLKn0)

Control signals:
  PCIe_nRST  ──[10kΩ pullup]───→  M.2 Pin 77 (PERST#)
  PCIe_CLKREQ ←─[10kΩ pullup]──  M.2 Pin 79 (CLKREQ#)
  PCIe_nWAKE  ←─[10kΩ pullup]──  M.2 Pin 81 (PEWAKE#)

Power:
  +3.3V ──→ M.2 3.3V pins (pins 2, 4, 35, 37, 39, 41)
  GND   ──→ All M.2 GND pins
```

**AC coupling caps**: Place 100nF 0402 caps as close to the CM5 connector as possible (within 5mm).

### 6.5 Sheet: Zigbee_EFR32 (Silicon Labs EFR32MG24)

**Recommended part**: EFR32MG24B220F1536IM48 (QFN-48, 6×6mm)

```
Power:
  +3.3V ──→ All AVDD pins (with 100nF each)
  +3.3V ──→ All DVDD pins (with 100nF each)
  +3.3V ──→ IOVDD pins (with 100nF each)
  DECOUPLE pin ── 100nF ── GND
  VREGVDD pin ── 1µF ── GND
  
  TOTAL: Place 100nF on EVERY power pin + 10µF bulk near chip

SPI to CM5:
  CM5 GPIO7  (SPI0_CE1)  ──→ EFR32 pin CS
  CM5 GPIO9  (SPI0_MISO) ←── EFR32 pin MISO  
  CM5 GPIO10 (SPI0_MOSI) ──→ EFR32 pin MOSI
  CM5 GPIO11 (SPI0_SCLK) ──→ EFR32 pin SCLK
  CM5 GPIO17 ──[10kΩ pullup]── EFR32 pin RESETn
  CM5 GPIO27 ←── EFR32 pin IRQ (configure as PTI or GPIO interrupt)

Crystals:
  HFXO: 38.4 MHz crystal ── 2× load capacitors (per datasheet)
         Connect to HFXO_XI and HFXO_XO pins
  LFXO: 32.768 kHz crystal ── 2× load capacitors
         Connect to LFXO_XI and LFXO_XO pins

RF Output:
  RF_2G4 pin ──[matching network]── Chip Antenna (Johanson 2450AT18x100)
                                     OR
                                     U.FL connector (for external antenna option)
```

**RF Matching Network:**
```
RF_2G4 ──[L_series]──┬──[C_series]── Antenna Feed
                      │
                   [C_shunt]
                      │
                     GND
```
Use the exact component values from the **Silicon Labs EFR32MG24 reference design** (available in Simplicity Studio or application note AN928). The values depend on your PCB stackup.

**Antenna choice for compactness:**
1. **Chip antenna** (recommended): Johanson 2450AT18x100 — tiny (1.6×0.8mm), ~$0.30
2. **PCB trace antenna**: Free, but needs tuning and takes ~15×10mm area
3. Provide a **U.FL pad** as an alternate connection point for customers needing extended range

### 6.6 Sheet: BME280_Sensor

**Bosch BME280 — LGA-8 package (2.5 × 2.5 × 0.93mm)**

```
+3.3V ──┬── Pin 1 (VDD) ──[100nF]── GND
        │
        ├── Pin 6 (VDDIO) ──[100nF]── GND
        │
        └── Pin 2 (CSB) ── +3.3V   [Disables SPI, enables I2C mode]

I2C connections:
  CM5 GPIO2 (I2C1_SDA) ──[4.7kΩ pullup to 3.3V]── Pin 3 (SDI/SDA)
  CM5 GPIO3 (I2C1_SCL) ──[4.7kΩ pullup to 3.3V]── Pin 4 (SCK/SCL)

Address selection:
  Pin 5 (SDO) ── GND     → Address 0x76
  Pin 5 (SDO) ── +3.3V   → Address 0x77

Ground:
  Pin 7, Pin 8 ── GND
```

**IMPORTANT — Thermal Isolation:**
The BME280 measures temperature. If it's near the CM5 (which runs at 60-80°C), your readings will be way off. Design considerations:
- Place BME280 at the **board edge**, as far from CM5 as possible
- Route a **thermal isolation slot** in the PCB (2mm wide slot cut) between the BME280 area and the rest of the board, leaving only 2-3 thin bridges for mechanical connection
- In KiCad PCB Editor: draw the slot on the **Edge.Cuts** layer

### 6.7 Sheet: Misc_GPIO_LEDs

```
Status LEDs (optional but recommended):
  +3.3V ── [330Ω] ── LED_Power (Green) ── GPIO (active low drive)
  +3.3V ── [330Ω] ── LED_Zigbee (Blue)  ── GPIO (active low drive)

Buttons:
  nRPI_BOOT ── [Tactile Button] ── GND   (with 10kΩ pullup to 3.3V)
    → Pressing during power-on enters USB boot/flash mode
  
  RUN/RESET ── [Tactile Button] ── GND   (directly from CM5 pin)

Debug UART header (optional, 4-pin 2.54mm):
  Pin 1: +3.3V
  Pin 2: GPIO14 (UART0_TX)
  Pin 3: GPIO15 (UART0_RX)
  Pin 4: GND
```

---

## 7. PCB LAYOUT STRATEGY FOR COMPACTNESS

### 7.1 Board Stackup: 4-Layer PCB (Minimum for This Design)

```
Layer 1 (F.Cu):   Signal + Components (top side)
Layer 2 (In1.Cu): SOLID GND plane (DO NOT SPLIT)
Layer 3 (In2.Cu): Power plane (3.3V / 5V zones) + some signals
Layer 4 (B.Cu):   Signal + Components (bottom side)

Total thickness: 1.6mm (standard)
```

**Why 4 layers are essential:**
- PCIe and HDMI require controlled impedance, which needs a continuous reference plane
- A solid ground plane on Layer 2 provides excellent return paths for all high-speed signals
- Cost: Only ~$5-10 more than 2-layer at JLCPCB for small quantities

### 7.2 Impedance Targets

Set these up in KiCad: **File → Board Setup → Design Rules → Net Classes**

| Net Class | Impedance | Trace Width* | Spacing* | Usage |
|-----------|-----------|-------------|----------|-------|
| Default | 50Ω SE | 0.25mm | 0.15mm | General signals |
| PCIe_diff | 85Ω diff | 0.20mm | 0.15mm gap | PCIe TX/RX pairs |
| HDMI_diff | 100Ω diff | 0.15mm | 0.20mm gap | HDMI TMDS pairs |
| USB2_diff | 90Ω diff | 0.18mm | 0.15mm gap | USB 2.0 D+/D- |
| Power | N/A | 0.50mm+ | 0.20mm | 5V, 3.3V traces |
| RF | 50Ω SE | ~0.30mm | 0.30mm | EFR32 to antenna |

*Exact widths depend on your fabrication stackup. **Use JLCPCB's impedance calculator**: https://jlcpcb.com/pcb-impedance-calculator — enter your 4-layer stackup and it will calculate the trace widths needed for each impedance target.

### 7.3 Setting Up Net Classes in KiCad

1. Open PCB Editor → **File → Board Setup → Design Rules → Net Classes**
2. Create net classes as listed above
3. Assign nets to classes:
   - Select all `PCIe_TX*`, `PCIe_RX*`, `PCIe_CLK*` nets → assign to `PCIe_diff`
   - Select all `HDMI0_TX*`, `HDMI0_CLK*` nets → assign to `HDMI_diff`
   - Select `USB_DP`, `USB_DN` → assign to `USB2_diff`
   - Select `+5V`, `+3.3V` → assign to `Power`

### 7.4 Target Board Size & Component Placement

**Target: 55mm × 40mm** (matches CM5 module footprint)

If this is too tight with the M.2 slot, extend to **70mm × 50mm** — still very compact.

**Component Placement Map (Top View):**

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  ┌────────────────────────────────────────────┐      │
│  │     CM5 MODULE (mounts on top via Hirose)  │      │
│  │                                            │      │
│  │  [Hirose J1]              [Hirose J2]      │      │
│  └────────────────────────────────────────────┘      │
│                                                      │
│ [Mini-HDMI]   [M.2 2230 NVMe]          [USB-C PWR]  │
│  (left)        (bottom side, under CM5)  (right)     │
│                                                      │
│ [nBoot]  [EFR32MG24]  [Antenna]   [BME280]           │
│  btn     (bottom)     (near edge)  (edge, isolated)  │
│                                                      │
│ [LED] [LED]  [UART debug header]  [Reset btn]        │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Key placement strategies:**

1. **CM5 connectors**: Place first — everything else revolves around these. Position exactly per mechanical drawing. The two Hirose connectors are your anchor points.

2. **M.2 NVMe on BOTTOM side**: Mount the M.2 connector on the bottom of the PCB. The SSD hangs underneath the board when the CM5 is mounted on top. This saves massive top-side space. Check clearance against CM5 3D model.

3. **Mini-HDMI**: Edge-mount on left or right side, positioned to minimize HDMI trace length from J1 pins.

4. **USB-C power**: Edge-mount on opposite side from HDMI.

5. **EFR32MG24**: Place on bottom side if possible. Keep the antenna at the board edge with clear ground plane requirements.

6. **BME280**: Board edge, far from CM5 heat source, with thermal isolation slot.

7. **Decoupling caps**: Within 2mm of their IC power pins, on the SAME LAYER as the IC.

### 7.5 Routing Order (Critical!)

Route in this exact sequence — most critical signals first:

```
1. PCIe differential pairs (TX, RX, REFCLK)
   → Tightest impedance requirements
   → Must be length-matched within each pair (±5 mils / 0.127mm)
   → Place AC coupling caps within 5mm of CM5 connector

2. HDMI differential pairs (TX0, TX1, TX2, CLK)
   → Must be length-matched within each pair
   → Match all four pairs to within ±50 mils of each other

3. USB differential pair (D+/D-)
   → Length match within pair

4. SPI bus (EFR32 ↔ CM5)
   → Keep total length under 50mm (2 inches)
   → SCLK is the critical signal

5. I2C bus (BME280 ↔ CM5)
   → Less critical, can be longer
   → Keep under 100mm

6. Power traces (5V, 3.3V)
   → Use wide traces (0.5mm+) or copper pours

7. Control signals, LEDs, buttons
   → Least critical, route last
```

### 7.6 Ground Plane Rules

- **Layer 2 (In1.Cu)**: Fill entire layer with GND pour. **NEVER split this plane, NEVER route signals on it.**
- **Layer 3 (In2.Cu)**: Create power zones (5V, 3.3V) with GND fill in remaining areas
- **Top and Bottom**: Add GND copper pours after all routing. Use **stitching vias** every 3-5mm to connect top/bottom GND to the inner ground plane.
- **Under EFR32 antenna**: Create a **keep-out zone on ALL copper layers** directly beneath the antenna element. The ground plane should extend up to the antenna feed point but not under it.

### 7.7 Space-Saving Techniques

| Technique | Savings | How to Implement |
|-----------|---------|-----------------|
| 0402 passives | ~40% vs 0603 | Select 0402 footprints for caps/resistors |
| Both-side assembly | ~30% area | Place small passives/ICs on bottom side |
| M.2 on bottom | Huge | SSD hangs under board, doesn't consume top area |
| Mini-HDMI vs full | ~50% connector | Smaller footprint, same functionality |
| Chip antenna | ~150mm² | vs PCB antenna which needs dedicated space |
| Edge-mount connectors | Board area | Connectors flush with board edge |
| Via-in-pad | Better routing | Allows traces to escape BGA/QFN pads directly (requires JLCPCB's via-in-pad service or via tenting) |

---

## 8. HIGH-SPEED DESIGN RULES

### 8.1 PCIe Gen2 Layout Rules

- **Differential impedance**: 85Ω ±10%
- **Length matching**: Within each pair (P/N), match to ±5 mils (0.127mm)
- **Between pairs**: TX and RX don't need to match each other
- **AC coupling caps** (100nF 0402): Place within 5mm of CM5 connector
- **Max trace length**: <6 inches (150mm) — easy on a compact board
- **Keep-out**: No other traces within 3× trace width of PCIe pairs
- **Reference plane**: Must have continuous GND below on Layer 2
- **No vias in the differential pair path** if possible; if you must via, place two vias (one for P, one for N) with same length

### 8.2 HDMI Layout Rules

- **Differential impedance**: 100Ω ±10%
- **Length matching within pair**: ±5 mils
- **Length matching between pairs**: All four TMDS pairs within ±50 mils of each other
- **AC coupling caps**: 100nF, as close to CM5 connector as possible
- **Guard traces**: Optional GND traces alongside HDMI pairs for crosstalk reduction
- **ESD protection**: Place TVS diodes or TPD chip as close to HDMI connector as possible

### 8.3 USB 2.0 Layout Rules

- **Differential impedance**: 90Ω ±10%
- **Length matching**: Within pair, ±5 mils
- **Trace length**: Keep under 150mm
- **Series resistors**: 22Ω on D+ and D- (near the CM5 connector)

### 8.4 RF (EFR32 to Antenna) Layout Rules

- **50Ω single-ended** trace from EFR32 RF pin to matching network to antenna
- **Matching network**: Place components directly adjacent to EFR32 RF pin
- **Antenna ground plane**: Clear ground plane extends from board edge to ~1mm from antenna element. NO copper under the antenna element itself.
- **No traces under antenna**: Keep-out zone on ALL layers under the antenna
- **RF trace**: Keep as short as possible, ideally <10mm

### 8.5 Using KiCad's Length Matching Tool

1. In PCB Editor, route your differential pair using `X` then `/` for diff-pair mode
2. After routing, select both traces of a pair
3. **Route → Tune Skew of a Differential Pair** (or press `9`)
4. KiCad will show the length mismatch and let you add serpentine tuning
5. Add serpentine to the shorter trace until matched

---

## 9. DESIGN RULE CHECK & VALIDATION

### 9.1 Electrical Rules Check (ERC) — In Schematic

1. Open Schematic Editor
2. **Inspect → Electrical Rules Checker**
3. Run ERC
4. Fix ALL errors:
   - Missing power flags → add `PWR_FLAG` symbol
   - Unconnected pins → add No Connect flags (`Q`) or wire them
   - Multiple power connections → verify intentional

### 9.2 Design Rules Check (DRC) — In PCB

1. Open PCB Editor
2. **Inspect → Design Rules Checker** (or `Ctrl+Shift+D`)
3. Run DRC
4. Fix ALL errors:
   - Clearance violations → increase spacing or adjust track width
   - Unconnected nets → check if you missed a route
   - Via-near-pad violations → adjust via placement
   - Track width violations → match to net class rules

### 9.3 Pre-Fab Checklist

Run through this before generating manufacturing files:

- [ ] All CM5 power pins (5V, GND) connected — verify against datasheet
- [ ] All unused CM5 pins marked with No Connect
- [ ] Differential pairs length-matched (use **Inspect → Board Statistics**)
- [ ] PCIe AC coupling caps within 5mm of CM5 connector
- [ ] HDMI AC coupling caps and ESD protection placed
- [ ] Every IC has 100nF decoupling cap within 2mm
- [ ] Bulk capacitors on 5V rail (100µF+ near CM5)
- [ ] EFR32 antenna area has keep-out zone on all layers
- [ ] BME280 has thermal isolation slot
- [ ] No signal traces crossing ground plane splits
- [ ] Mounting holes present (M2.5, aligned with CM5 if needed)
- [ ] Board outline complete on Edge.Cuts layer
- [ ] Silkscreen: component designators, connector labels, board name/version
- [ ] Fiducials: 2× 1mm fiducials at diagonal corners (for pick-and-place)

### 9.4 3D Viewer Check

**View → 3D Viewer** in PCB Editor:

Check for:
- CM5 module clearance above bottom-side components
- M.2 SSD clearance if mounted on bottom
- Connector accessibility from board edges
- No physical collisions between tall components
- Overall aesthetics and layout sanity

---

## 10. MANUFACTURING OUTPUT

### 10.1 Generate Gerber Files

**File → Plot** in PCB Editor

Select these layers:

| Layer | Include | Purpose |
|-------|---------|---------|
| F.Cu | ✅ | Front copper |
| In1.Cu | ✅ | Inner layer 1 (GND) |
| In2.Cu | ✅ | Inner layer 2 (Power) |
| B.Cu | ✅ | Back copper |
| F.SilkS | ✅ | Front silkscreen |
| B.SilkS | ✅ | Back silkscreen |
| F.Mask | ✅ | Front solder mask |
| B.Mask | ✅ | Back solder mask |
| Edge.Cuts | ✅ | Board outline |
| F.Paste | ✅ | Front solder paste (for stencil) |
| B.Paste | ✅ | Back solder paste |

Settings:
- Plot format: **Gerber**
- Output directory: `./gerbers/`
- Check "Use Protel filename extensions" (for JLCPCB compatibility)

Click **Plot** then **Generate Drill Files**:
- Drill file format: **Excellon**
- Map file format: Gerber
- Drill origin: Absolute
- Click **Generate Drill File**

### 10.2 Generate BOM and CPL for Assembly

If using **JLCPCB assembly service**:

1. Install the **JLCPCB Tools** plugin (from Plugin Manager)
2. In PCB Editor: **Tools → Generate JLCPCB Fabrication Files**
3. This generates:
   - BOM (Bill of Materials) in JLCPCB format
   - CPL (Component Placement List) with X/Y/rotation

Alternatively, manually:
- **File → Fabrication Outputs → BOM** for bill of materials
- **File → Fabrication Outputs → Component Placement** for pick-and-place data

### 10.3 Fabrication Specifications

Order with these specs:

| Parameter | Value | Notes |
|-----------|-------|-------|
| Layers | **4** | Required for impedance control |
| Board thickness | **1.6mm** | Standard |
| Dimensions | 55×40mm or 70×50mm | Your target size |
| Surface finish | **ENIG** | Required for fine-pitch Hirose (0.4mm) |
| Min trace/space | 0.1mm / 0.1mm | Most fabs support this |
| Min via drill | 0.2mm (laser) or 0.3mm (mechanical) | |
| Impedance control | **Yes** | Specify in order notes |
| Copper weight | 1oz outer, 0.5oz inner | Standard for 4-layer |
| Solder mask | Any color | Green is cheapest |
| PCBA (assembly) | Optional | Use JLCPCB SMT service |

### 10.4 Where to Order (Fab Houses)

| Fab House | Best For | Approx. Cost (5 boards, 4-layer) |
|-----------|---------|-----------------------------------|
| **JLCPCB** | Cheapest, best for prototypes | $15-30 + shipping from China |
| **PCBWay** | Good impedance control | $30-50 |
| **AISLER** | EU-based (fast for Germany!) | €40-60 |
| **Eurocircuits** | EU, high quality | €60-100 |
| **OSH Park** | US-based, nice purple boards | $50-80 |

**Recommendation for you (based in Germany)**: Start with **JLCPCB** for first prototype (cheapest, 1-week delivery). If you need faster turnaround, use **AISLER** (ships from within EU, no customs delays).

---

## 11. ASSEMBLY & TESTING

### 11.1 Tools Needed

| Tool | Purpose | Approx. Cost |
|------|---------|-------------|
| Soldering iron (fine tip) | Through-hole, larger SMD | €30-80 |
| Hot air rework station | QFN (EFR32), LGA (BME280) | €50-100 |
| Solder paste + stencil | Reflow for fine-pitch parts | €15 (order stencil with PCBs) |
| Flux (no-clean) | Helps solder flow | €10 |
| Multimeter | Continuity, voltage checks | €20-50 |
| Tweezers (fine tip) | Placing 0402 components | €10 |
| Magnifying glass/microscope | Inspecting solder joints | €20-100 |
| Hot plate or reflow oven | Best for batch assembly | €50-200 (optional) |

### 11.2 Assembly Sequence

**If assembling yourself:**

1. **Apply solder paste** using stencil (order from JLCPCB with your board)
2. **Place bottom-side components first** (if double-sided)
3. **Reflow bottom side** (hot plate or oven at 240°C peak)
4. Flip board, apply paste to top side
5. **Place top-side components**:
   - Hirose connectors first (most critical alignment)
   - EFR32MG24 (QFN-48, needs perfect alignment)
   - BME280 (LGA-8, tiny)
   - Passives (caps, resistors)
   - Connectors (HDMI, USB-C, M.2)
6. **Reflow top side**
7. **Hand-solder** through-hole parts (buttons, headers)
8. **Inspect** all joints under magnification

**Alternative: Use JLCPCB's assembly service** — they place and solder SMD components for you. Much easier for QFN and 0402 parts. Cost: ~$10-30 per board depending on component count.

### 11.3 Testing Sequence

**STEP 1: Visual Inspection**
- Check for solder bridges, especially on Hirose connectors (0.4mm pitch)
- Verify component orientation (capacitor polarity, IC dot/pin 1)

**STEP 2: Power Test (NO CM5 mounted!)**
```
1. Connect 5V via USB-C
2. Measure with multimeter:
   - +5V rail: should read 5.0V ±5%
   - +3.3V rail: should read 3.30V ±5%
   - +1.8V rail (if present): 1.80V ±5%
3. Check current draw: should be <50mA without CM5
4. If current is high → SHORT CIRCUIT → disconnect immediately!
```

**STEP 3: Mount CM5 + Boot Test**
```
1. Carefully align CM5 onto Hirose connectors
2. Press firmly until it clicks/seats
3. Connect HDMI monitor + USB keyboard
4. Apply 5V power
5. CM5 should boot (green LED flashes on CM5 module)
6. If no boot → check nBOOT pin isn't grounded
```

**STEP 4: NVMe SSD Test**
```bash
# Insert NVMe SSD into M.2 slot, then boot
sudo lspci                    # Should show NVMe controller
sudo nvme list                # Should list your SSD
sudo dd if=/dev/nvme0n1 of=/dev/null bs=1M count=100  # Read test
```

**STEP 5: Zigbee Radio Test**
```bash
# Check SPI interface
ls /dev/spidev0.*              # Should show spidev0.1

# For ZHA/Zigbee2MQTT, you'll flash the EFR32 first
# Use Silicon Labs Simplicity Commander to flash NCP firmware
```

**STEP 6: BME280 Test**
```bash
# Enable I2C
sudo raspi-config  # → Interface Options → I2C → Enable

# Detect sensor
sudo i2cdetect -y 1
# Should show device at 0x76 (or 0x77)

# Quick Python test
pip install bme280 smbus2
python3 -c "
import smbus2, bme280
bus = smbus2.SMBus(1)
cal = bme280.load_calibration_params(bus, 0x76)
data = bme280.sample(bus, 0x76, cal)
print(f'Temp: {data.temperature:.1f}°C')
print(f'Humidity: {data.humidity:.1f}%')
print(f'Pressure: {data.pressure:.1f} hPa')
"
```

---

## 12. COMMON MISTAKES & TROUBLESHOOTING

### 12.1 Schematic Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Missing GND pin on CM5 connector | Intermittent failures, overheating | Double-check every pin against datasheet |
| Wrong I2C pullup value | Communication failures | Use 4.7kΩ for 3.3V I2C |
| Missing decoupling caps | IC instability, noise | 100nF on every power pin |
| Not connecting all 5V pins | CM5 brown-out under load | Connect every 5V pin on both connectors |
| CSB on BME280 left floating | SPI/I2C mode undefined | Tie to 3.3V for I2C mode |

### 12.2 PCB Layout Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Broken ground plane under high-speed | Signal integrity failure | Never route on Layer 2 (GND) |
| Differential pairs not matched | Data errors, no link | Use KiCad's length matching tool |
| AC coupling caps too far from CM5 | PCIe won't train | Place within 5mm of connector |
| Copper under chip antenna | Detuned antenna, poor range | Create keep-out zone on ALL layers |
| BME280 near CM5 heat source | Temperature reads 10°C high | Thermal isolation slot + edge placement |
| Insufficient bulk caps on 5V | CM5 resets under load | 2× 100µF + 4× 10µF near connector |
| USB-C wired incorrectly | No power or intermittent | Check CC resistors (5.1kΩ to GND on each CC pin) |

### 12.3 Manufacturing Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| Wrong surface finish (HASL vs ENIG) | Can't solder 0.4mm pitch Hirose | Always specify ENIG for fine-pitch |
| No impedance control ordered | Incorrect trace impedance | Add impedance note in fab order |
| Missing solder paste layer | Can't make stencil | Include F.Paste / B.Paste layers |
| Missing fiducials | Pick-and-place alignment issues | Add 2× 1mm fiducials at corners |

---

## 13. RESOURCE LINKS & LEARNING PATH

### 13.1 Essential Downloads Checklist

| Resource | URL |
|----------|-----|
| **KiCad 8 (Windows)** | https://www.kicad.org/download/windows/ |
| **CM5 IO Board KiCad Files** | https://pip.raspberrypi.com/categories/1098-design-files |
| **CM5 Datasheet** | https://datasheets.raspberrypi.com/cm5/cm5-datasheet.pdf |
| **CM5IO Datasheet** | https://datasheets.raspberrypi.com/cm5/cm5io-datasheet.pdf |
| **CM5 3D Model** | https://datasheets.raspberrypi.org/cm5/CM5-step.zip |
| **Waveshare CM5-NANO-B Wiki** | https://www.waveshare.com/wiki/CM5-NANO-B |
| **CM5 MINIMA (open-source carrier)** | https://github.com/piecol/CM5_MINIMA_REV2 |
| **Shawn Hymel CM4 Template** | https://github.com/ShawnHymel/rpi-cm4-carrier-template |
| **VIBE2051 CM5 Carrier** | https://github.com/VIBE2051/RPI-CM5-Carrier-Board |
| **MakerForge CM5 Guide** | https://www.makerforge.tech/posts/cm5-carrier-basics/ |
| **CM5 Reverse Engineering** | https://github.com/schlae/cm5-reveng |
| **JLCPCB Impedance Calc** | https://jlcpcb.com/pcb-impedance-calculator |
| **SnapEDA (component libs)** | https://www.snapeda.com |

### 13.2 Recommended Learning Path

Since you're new to PCB design, follow this sequence:

**Week 1-2: Learn KiCad Basics**
1. Watch **"KiCad 8 Complete Tutorial"** on YouTube (Chris Gammell or Phil's Lab)
2. Watch **Phil's Lab #27**: "PCB Design for BEGINNERS" — covers impedance, stackups, DRC
3. Complete KiCad's built-in tutorial project (Help → Getting Started)
4. Practice: make a simple LED + resistor + button board

**Week 3: Study Reference Designs**
5. Open CM5IO reference in KiCad → trace every connection on the schematic
6. Study the PCB layout → understand how differential pairs are routed
7. Open CM5 MINIMA REV2 → study how they achieved compact size
8. Read the Waveshare CM5-NANO-B schematic PDF → understand their design choices

**Week 4-5: Design Your Board**
9. Create your project following this guide
10. Start with CM5 connectors + power (most critical)
11. Add HDMI, USB, NVMe (copy from reference)
12. Add EFR32 and BME280 (your custom additions)
13. Run ERC, fix all errors

**Week 5-6: PCB Layout**
14. Component placement (spend 70% of layout time here!)
15. Route high-speed pairs first
16. Route remaining signals
17. Add copper pours, stitching vias
18. Run DRC, fix all errors
19. 3D check for clearances

**Week 7: Review & Manufacture**
20. Post your design for peer review:
    - Reddit: r/PrintedCircuitBoard
    - EEVforum: PCB Design subforum
    - MakerForge GitHub Discussions
21. Address feedback, generate Gerbers
22. Order boards from JLCPCB/AISLER

**Week 8+: Assemble & Test**
23. Assemble board (or use JLCPCB assembly)
24. Test following the sequence in Section 11
25. Iterate on Rev 2 based on what you learn

### 13.3 YouTube Channels for PCB Design Learning

| Channel | What They Cover |
|---------|----------------|
| **Phil's Lab** | Impedance control, high-speed design, KiCad tutorials |
| **Robert Feranec** | PCB layout reviews, signal integrity |
| **Chris Gammell (Contextual Electronics)** | KiCad project-based learning |
| **Shawn Hymel** | CM4/CM5 carrier board design series |
| **EEVblog** | General electronics, PCB design tips |

### 13.4 Community Resources for Help

| Community | URL |
|-----------|-----|
| r/PrintedCircuitBoard | https://reddit.com/r/PrintedCircuitBoard |
| KiCad Forum | https://forum.kicad.info |
| Raspberry Pi Forums | https://forums.raspberrypi.com |
| MakerForge Discussions | https://github.com/makerforgetech/modular-biped/discussions |
| EEVforum | https://www.eevblog.com/forum/ |

---

## QUICK REFERENCE: BILL OF MATERIALS

| Component | Part Number | Package | Qty | ~Cost |
|-----------|-------------|---------|-----|-------|
| CM5 Connector | Hirose DF40C-100DS-0.4V(51) | SMD | 2 | $2.50 ea |
| EFR32MG24 | EFR32MG24B220F1536IM48 | QFN-48 | 1 | $3.50 |
| BME280 | BME280 (Bosch) | LGA-8 | 1 | $2.50 |
| M.2 Connector | 2230 M-key socket | SMD | 1 | $0.80 |
| Mini-HDMI | Mini-HDMI receptacle | SMD | 1 | $0.50 |
| USB-C Power | USB-C 16-pin | SMD | 1 | $0.30 |
| 3.3V LDO | AP2112K-3.3TRG1 | SOT-23-5 | 1 | $0.20 |
| 38.4 MHz Crystal | For EFR32 HFXO | 3.2×2.5mm | 1 | $0.30 |
| 32.768 kHz Crystal | For EFR32 LFXO | 2.0×1.2mm | 1 | $0.20 |
| Chip Antenna | Johanson 2450AT18x100 | 0402 | 1 | $0.30 |
| Decoupling caps | 100nF, 0402 | 0402 | ~30 | $1.50 |
| Bulk caps | 10µF, 0805 | 0805 | ~6 | $0.60 |
| Bulk caps | 100µF electrolytic | 6.3×5.4mm | 2 | $0.40 |
| Resistors | Various, 0402 | 0402 | ~15 | $0.50 |
| LEDs | Green, Blue, 0402 | 0402 | 2 | $0.20 |
| Tactile switches | nBOOT, Reset | 4×4mm | 2 | $0.20 |
| **4-Layer PCB** | 55×40mm or 70×50mm, ENIG | — | 5 | $15-30 |
| **Total per board** | *(excluding CM5 module & NVMe SSD)* | | | **~$15-20** |

---

*Document prepared for Naveen's CM5-based Home Assistant Controller project.*
*This is Rev 1 of the design guide — expect updates as the design evolves.*
