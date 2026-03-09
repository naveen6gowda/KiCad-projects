# How to Post Your PCB for Review on Reddit — Complete Guide

## Where to Post

| Subreddit | Best For | Rules |
|-----------|----------|-------|
| **r/PrintedCircuitBoard** (primary) | Schematic + PCB layout review | Strict formatting rules — read below carefully |
| **r/AskElectronics** | General circuit questions, sanity checks | More casual, good for early-stage questions |
| **r/embedded** | Firmware/MCU-specific questions | Good for CM5/EFR32MG24 integration questions |
| **r/homeassistant** | HA-specific product feedback | Good for validating feature requirements |

---

## r/PrintedCircuitBoard — Strict Rules (Your Posts WILL Be Deleted If Broken)

### ❌ DO NOT

- Post fuzzy/blurry images
- Post camera photos of a computer screen (screenshot only!)
- Post dark/black background schematics (use white/light background)
- Post without schematic (layout-only posts get poor feedback)
- Use non-standard image formats (PNG and JPG only)

### ✅ DO

- Use "Review Request" or similar clear title
- Export high-resolution PNG images from KiCad
- Include BOTH schematic AND layout
- Provide context in comments about what the board does
- Use light/white background for all exports

---

## Step-by-Step: Preparing Your Files

### Step 1: Export Schematic as PNG (White Background)

In KiCad Schematic Editor:

1. Go to **File → Export → Drawing to Image**
2. Set format to **PNG**
3. Set resolution to **300 DPI** (minimum)
4. Make sure background is **white** (not black)
5. If you have multiple schematic sheets, export each one separately
6. Name them: `CM5_Hub_Schematic_Sheet1_Power.png`, `CM5_Hub_Schematic_Sheet2_MCU.png`, etc.

### Step 2: Export PCB Layout Images

Export these views from KiCad PCB Editor (**File → Export → Image** or use **Plot**):

1. **Top Copper (F.Cu) + Components** — PNG, 300 DPI
2. **Bottom Copper (B.Cu) + Components** — PNG, 300 DPI
3. **All Layers combined** — PNG (useful for overall view)
4. **3D View** — Screenshot from KiCad 3D Viewer (Alt+3)
5. **Board outline with dimensions** — Show overall board size

**Tip**: In KiCad PCB Editor, you can also do **File → Plot** and choose SVG or PDF for individual layers.

### Step 3: Export the Board Stackup

Take a screenshot of your layer stackup configuration:

1. In KiCad PCB Editor → **Board Setup → Board Stackup**
2. Screenshot showing your 4-layer stackup:
   - L1: Signal + Components
   - L2: Ground Plane
   - L3: Power Plane
   - L4: Signal + Components

### Step 4: Prepare Your BOM Summary

Create a simple table of key components:

```
Key Components:
- MCU: EFR32MG24B220F1536IM48 (Zigbee/Thread radio, QFN-48)
- Antenna: Johanson 2450AT18x100 (2.4 GHz chip antenna)
- Sensor: BME280 (temperature/humidity/pressure)
- Connectors: 2× Hirose DF40HC(3.0)-100DS (CM5 socket)
- NVMe: M.2 M-Key connector (2230/2242)
- Power: 5V USB-C input, onboard 3.3V/1.8V regulators
- Ethernet: RTL8211F PHY (Gigabit)
```

### Step 5: Note Your Design Constraints & Concerns

Write down specific areas where you want feedback:

```
Specific areas I'd like reviewed:
1. EFR32MG24 RF section — antenna matching network & trace impedance
2. CM5 high-speed connector routing (DDR4, PCIe)
3. Power distribution — 5V→3.3V→1.8V regulator placement
4. NVMe M.2 PCIe Gen2/3 routing
5. Thermal isolation slot for BME280
6. High-voltage clearances (if relay board)
7. Ground plane continuity
8. Decoupling capacitor placement
```

---

## The Reddit Post — Template

### Title Format

```
[Review Request] CM5 Custom Carrier Board — Home Automation Hub with Onboard Zigbee/Thread (KiCad, 4-Layer)
```

### Post Body (Copy & Modify This Template)

```markdown
Hi everyone,

I'm designing a custom Raspberry Pi CM5 carrier board for a home automation hub
product. This is my first 4-layer PCB with high-speed interfaces and I'd really
appreciate a review before I send it for fabrication.

## What the board does

Custom carrier board for Raspberry Pi Compute Module 5, designed as a dedicated
Home Assistant hub. Key differentiator: onboard Zigbee 3.0/Thread radio
(EFR32MG24) — no USB dongle needed.

## Key specs

- **Compute**: Raspberry Pi CM5 (8GB/64GB eMMC)
- **Wireless**: EFR32MG24B220F1536IM48 with chip antenna (Zigbee 3.0/Thread/BLE)
- **Storage**: M.2 M-Key NVMe (2230/2242)
- **Network**: Gigabit Ethernet (RTL8211F PHY)
- **Sensors**: BME280 (temp/humidity/pressure)
- **Power**: USB-C 5V/5A input, onboard LDOs for 3.3V and 1.8V
- **PCB**: 4-layer, ENIG finish, 1.6mm thickness
- **Size**: ~85 × 56mm (credit card form factor)

## Layer stackup

- L1: Signal + Components (F.Cu)
- L2: Ground Plane (GND)
- L3: Power Plane (3.3V / 1.8V split)
- L4: Signal + Components (B.Cu)

Impedance controlled: 50Ω single-ended, 90Ω differential (PCIe), 100Ω differential (Ethernet/DDR)

## Reference designs used

- Raspberry Pi CM5 IO Board (official KiCad files)
- Silicon Labs BRD4187C (EFR32MG24 reference)
- Waveshare CM5-NANO-B (compact CM5 carrier)

## Design tools

- KiCad 8.x
- Schematic + PCB files available on request

## Specific concerns / areas I'd like reviewed

1. **RF section**: EFR32MG24 antenna matching network — chip antenna at board edge, 
   is my matching network layout correct? Ground plane clearance under antenna?
2. **CM5 high-speed routing**: DDR4 and PCIe differential pairs — length matching, 
   via transitions
3. **Power integrity**: Decoupling cap placement for CM5 and EFR32MG24
4. **NVMe routing**: PCIe Gen2 x1 to M.2 connector — trace impedance
5. **Ethernet**: RTL8211F to RJ45 routing, magnetics placement
6. **BME280 placement**: Thermal isolation slot — is it sufficient?
7. **Ground plane**: Any obvious splits or breaks I should worry about?
8. **General DFM**: Anything that would cause issues in manufacturing?

## Images attached

1. Full schematic (multiple sheets)
2. Top copper + components
3. Bottom copper + components
4. Ground plane (layer 2)
5. Power plane (layer 3)
6. 3D render
7. Board dimensions

## Manufacturing target

Planning to fab at LionCircuits (India) or JLCPCB, 4-layer ENIG.

Thanks in advance for any feedback!
```

---

## How to Upload Images on Reddit

### Option A: Reddit Image Post (Recommended)

1. Go to r/PrintedCircuitBoard
2. Click "Create Post"
3. Select **"Images"** tab
4. Upload all your PNG images (schematic + layout + 3D)
5. Add the post body text in the post or as the first comment
6. Reddit allows up to **20 images** per image post

### Option B: Imgur Album + Text Post

1. Go to imgur.com → "New Post"
2. Upload all images
3. Copy the album link
4. Create a **Text Post** on Reddit
5. Paste the imgur album link in the body
6. This gives you more control over text formatting

### Option C: Share KiCad Files (Best for Detailed Review)

1. Upload your KiCad project to **GitHub** or **GitLab** (public repo)
2. Include the repo link in your Reddit post
3. Reviewers can actually open your files and zoom in
4. This gets the BEST quality feedback

---

## Bonus: Also Post on These Forums

### EEVblog Forum (www.eevblog.com/forum)

- Section: "PCB Design / EMC"
- Very knowledgeable community, especially for RF and power
- Slower responses but higher quality feedback

### All About Circuits Forum (forum.allaboutcircuits.com)

- Section: "PCB Design"
- Good for detailed schematic reviews
- Active community

### KiCad Forum (forum.kicad.info)

- Good for KiCad-specific questions
- Help with footprints, libraries, DRC issues

---

## Timeline / When to Post

| Stage | What to Post | Where |
|-------|-------------|-------|
| **Schematic only** (before layout) | Schematic PNGs + component list | r/PrintedCircuitBoard, r/AskElectronics |
| **Layout draft** (before DRC) | Schematic + layout PNGs | r/PrintedCircuitBoard |
| **Final layout** (after DRC, before fab) | Full package: schematic + layout + 3D + stackup | r/PrintedCircuitBoard + EEVblog |
| **Post-fabrication** | Photos of assembled board | r/PrintedCircuitBoard (show-off post) |

**Recommendation**: Post at the **schematic-only stage first**. Fix issues. Then post again at the **layout stage**. Two rounds of review catches more problems.

---

## What Good Feedback Looks Like (What to Expect)

Common feedback you'll likely receive on your board:

- "Move decoupling caps closer to pin X"
- "Your ground plane has a split under the RF trace — fix this"
- "Add more stitching vias around the antenna keepout zone"
- "Your PCIe differential pair length mismatch is too high"
- "Increase clearance between high-voltage and low-voltage sections"
- "Add thermal relief on the ground pad of your QFN"
- "Your power trace is too narrow for the expected current"
- "Consider adding test points for debugging"

**Don't take feedback personally** — the community is direct but helpful. Thank reviewers and follow up on their suggestions.

---

## Quick Checklist Before Posting

- [ ] Schematic exported as PNG (white background, 300 DPI, readable text)
- [ ] Each schematic sheet is a separate image
- [ ] PCB top copper layer exported as PNG
- [ ] PCB bottom copper layer exported as PNG
- [ ] Ground plane layer exported
- [ ] 3D render screenshot
- [ ] Board dimensions visible
- [ ] Layer stackup described
- [ ] BOM summary of key components
- [ ] Reference designs mentioned
- [ ] Specific concerns/questions listed (5-8 items)
- [ ] Post title includes: [Review Request] + board name + layer count + tool
- [ ] All images are clear, high-resolution, light background
- [ ] KiCad project optionally shared via GitHub
