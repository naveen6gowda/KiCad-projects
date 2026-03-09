# CM5_MINIMA_3 — Step-by-Step Fix Guide

**Board**: CM5 MINIMA Rev 3.1  
**Tool**: KiCad 9.0  
**Date**: March 2026

This guide is organized in the order you should perform the fixes. Some steps depend on earlier ones, so follow the sequence.

---

## Phase 1: Stackup & Impedance Foundation

Everything else depends on getting the stackup right first. If the stackup changes, all trace widths and impedance values change with it.

### Step 1.1 — Contact JLCPCB for the Actual Stackup

**Why**: Your current stackup (1.5296mm total, asymmetric cores of 0.5mm/0.55mm) is not a standard JLCPCB 6-layer offering. If you order without impedance control, they will substitute their standard stackup, and all your impedance calculations become invalid.

**What to do:**

1. Go to [JLCPCB Impedance Calculator](https://jlcpcb.com/impedance) and select "6 layers".
2. Select the stackup closest to your needs. For a 1.6mm board, the typical option is:
   - **JLC06161H-7628** — symmetric 6-layer, 1.6mm total
3. Note down the exact layer thicknesses, dielectric constants (εr), and copper weights they provide.
4. Alternatively, submit a pre-order inquiry asking JLCPCB to confirm they can fabricate your exact custom stackup (the 2116 RC58% prepreg you specified). Custom stackups cost more but give you precise impedance control.

**If using JLCPCB standard stackup JLC06161H-7628:**

```
F.Cu      35μm   (1oz)
Prepreg   0.2104mm  7628×1  εr≈4.6
In1.Cu    17.5μm (0.5oz)
Core      0.36mm    εr≈4.6
In2.Cu    17.5μm (0.5oz)
Prepreg   0.1088mm  2116×1  εr≈4.25
In3.Cu    17.5μm (0.5oz)
Core      0.36mm    εr≈4.6
In4.Cu    17.5μm (0.5oz)
Prepreg   0.2104mm  7628×1  εr≈4.6
B.Cu      35μm   (1oz)
Total:    ~1.6mm (SYMMETRIC — no warpage concern)
```

### Step 1.2 — Update KiCad Board Setup with Confirmed Stackup

**In KiCad:**

1. Open `CM5_MINIMA_3.kicad_pcb`
2. Go to **File → Board Setup → Physical Stackup**
3. Update each layer's thickness, material, and εr to match the JLCPCB-confirmed values
4. Make sure the total thickness shows ~1.6mm
5. Verify symmetry: top prepreg = bottom prepreg, top core = bottom core

### Step 1.3 — Recalculate Impedance-Controlled Trace Widths

**Why**: Your current trace widths (0.13mm for 100Ω, 0.14mm/0.154mm gap for 90Ω) were calculated for a 99.4μm prepreg. If the prepreg changes (e.g., to 210μm for JLC standard), the widths must change.

**What to do:**

1. Use the **JLCPCB impedance calculator** or **Saturn PCB Toolkit** (free) or **KiCad's built-in calculator** (Calculators → Transmission Lines)
2. For the confirmed stackup, calculate:

| Target | Type | Parameters to Calculate |
|--------|------|------------------------|
| 50Ω single-ended | Microstrip (F.Cu ref In1.Cu) | trace width |
| 90Ω differential | Edge-coupled microstrip (F.Cu ref In1.Cu) | trace width + gap |
| 100Ω differential | Edge-coupled microstrip (F.Cu ref In1.Cu) | trace width + gap |
| 50Ω single-ended | Stripline (In2.Cu between cores) | trace width |
| 90Ω differential | Edge-coupled stripline (B.Cu ref In4.Cu) | trace width + gap |

3. In KiCad, go to **File → Board Setup → Net Classes** and update:

| Net Class | Update `Track Width` | Update `Diff Pair Width` | Update `Diff Pair Gap` |
|-----------|---------------------|--------------------------|------------------------|
| 50ohm | (new calculated value) | — | — |
| 90ohm | (new calculated value) | (new calculated value) | (new calculated value) |
| 100ohm | (new calculated value) | (new calculated value) | (new calculated value) |

4. After updating, you'll need to re-route all impedance-controlled traces to the new widths (see Phase 3).

---

## Phase 2: Fix Net Class Assignments

**Why**: Many high-speed differential nets are currently falling through to the Default net class (0.178mm width, 0.25mm gap) instead of their correct impedance class. This means they're routed at the wrong impedance.

### Step 2.1 — Fix HDMI1 Net Assignments

**Current problem**: The 12 HDMI1_* nets from the CM5 connector module don't match the pattern `/*HDMI_P*`. They use the naming format `/CM5/HDMI1_xxx` instead of `/CM5/HDMI_PI.xxx`.

**In KiCad:**

1. Go to **File → Board Setup → Net Classes → Netclass Assignments** tab (at the bottom)
2. You need to add patterns or explicit assignments. Choose one approach:

**Option A — Add new patterns** (recommended):

Click the **+** button under "Netclass Patterns" and add:

| Pattern | Net Class |
|---------|-----------|
| `/*HDMI1_*.D*` | 100ohm |
| `/*HDMI1_*.CK*` | 100ohm |
| `/*HDMI1*.CK*` | 100ohm |

This catches `/CM5/HDMI1_D0_P`, `/CM5/HDMI1_D0_N`, `/CM5/HDMI1_CK_P`, `/CM5/HDMI1_CK_N`, etc.

**Option B — Use the Schematic Editor** (cleaner long-term):

1. Open the schematic sheet **CM5.kicad_sch** (or wherever the HDMI connector is)
2. Select each HDMI differential net label
3. In Properties, set the **Net Class** field to `100ohm`
4. Update PCB from schematic (**Tools → Update PCB from Schematic**)

### Step 2.2 — Fix USB3 Net Assignments

**Current problem**: USB3 nets use the format `/CM5/USB3-0-TX_P` which doesn't match `/*USB_*`. The dash in "USB3-0" breaks the pattern match.

**Add these patterns:**

| Pattern | Net Class |
|---------|-----------|
| `/*USB3*` | 90ohm |

This catches all USB3-0 and USB3-1 differential pairs.

### Step 2.3 — Fix DPHY1 Net Assignments

**Current problem**: DPHY1 (second camera/display port) nets don't match `/*DPHY0*` — they start with `DPHY1`.

**Add this pattern:**

| Pattern | Net Class |
|---------|-----------|
| `/*DPHY1*` | 100ohm |

### Step 2.4 — Verify All Assignments

After adding the patterns:

1. Go to **Inspect → Board Statistics** or **Inspect → Design Rule Check**
2. Run DRC — look for any "Net class" related warnings
3. In the **Net Inspector** panel (View → Panels → Net Inspector), filter by net class and verify:
   - All HDMI1_D* and HDMI1_CK* nets show as `100ohm`
   - All USB3-* nets show as `90ohm`
   - All DPHY1_* nets show as `100ohm`
   - All PCIE_* nets show as `90ohm`
   - All ETH_PI.* nets show as `100ohm`

### Step 2.5 — Re-route Affected Traces

After assigning the correct net classes, the existing traces will have wrong widths. You need to re-route them:

1. Select each affected net in the PCB editor
2. Right-click → **Select → All Tracks in Net**
3. Delete the selected tracks
4. Re-route using the **Route Differential Pair** tool (shortcut: **D** then **press 6** for diff pair mode)
5. KiCad will automatically use the correct width/gap from the net class

---

## Phase 3: Fix Differential Pair Length Matching

This is the most time-consuming step but the most critical for board function.

### Step 3.1 — PCIe Differential Pairs (Currently ~1.35mm mismatch, need <0.127mm)

**Current routing analysis:**

| Pair | Mostly on | Via transitions | Length P | Length N | Mismatch |
|------|-----------|----------------|----------|----------|----------|
| TX | F.Cu (43mm) + B.Cu (2.2mm) | 2 vias each | 45.21mm | 46.57mm | 1.36mm |
| RX | F.Cu (42mm) + B.Cu (2.2mm) | 2 vias each | 44.02mm | 45.36mm | 1.34mm |
| CLK | B.Cu (51mm) + F.Cu (3mm) | 2 vias each | 54.08mm | 55.44mm | 1.36mm |

**Step-by-step fix:**

1. **Open the PCB and enable length display:**
   - View → Panels → **Net Inspector** (shows total net lengths)
   - Or use **Inspect → Board Statistics → Track Length** report

2. **Select the PCIe TX pair for tuning:**
   - Click on the TX_P trace (net `/CM5/PCIE_PI.TX_P`)
   - Press **U** to select the full trace
   - Note the length displayed in the status bar

3. **Use the Differential Pair Tuning tool:**
   - Go to **Route → Tune Differential Pair Length** (or press shortcut — check your key bindings)
   - Click on the **shorter** trace of the pair (TX_P in this case, since it's 45.21mm vs 46.57mm)
   - KiCad will show a preview of the serpentine/accordion pattern
   - Adjust the amplitude and spacing with keyboard:
     - **1/2** keys: decrease/increase amplitude
     - **3/4** keys: decrease/increase spacing
     - **</>** keys: change the target length
   - Set the target so that P matches N within 0.127mm (5 mils)
   - Click to place the tuning pattern

4. **Place the tuning serpentine in a suitable location:**
   - Best location: on the **F.Cu** section of the trace, where there's the most room
   - Avoid placing tuning near via transitions or near the M.2 connector pads
   - The tuning pattern should be on a **straight section** of the trace
   - Keep the tuning amplitude small (< 3× trace width) to minimize impedance discontinuity

5. **Repeat for TX_N, RX pair, and CLK pair:**

   For each pair:
   - Identify which trace is shorter
   - Add serpentine to the shorter trace to match the longer one
   - Verify final mismatch < 0.127mm

6. **Also check inter-pair length matching:**
   - PCIe spec allows up to ~12.5mm difference between TX and RX pair lengths
   - CLK pair should be shorter than or equal to the data pairs
   - Current: TX≈46mm, RX≈45mm, CLK≈55mm
   - **CONCERN**: CLK is ~10mm longer than data pairs — check if this meets CM5 timing requirements. If the CM5 datasheet specifies CLK-to-data length matching, you may need to shorten CLK or lengthen TX/RX.

### Step 3.2 — Ethernet Differential Pairs

**Current status:**

| Pair | Length P | Length N | Mismatch | Status |
|------|----------|----------|----------|--------|
| TRD0 | 32.42mm | 32.19mm | 0.232mm | Needs minor fix |
| TRD1 | 33.03mm | 34.67mm | 1.641mm | **Needs major fix** |
| TRD2 | 38.64mm | 38.41mm | 0.232mm | Needs minor fix |
| TRD3 | 40.70mm | 42.61mm | 1.917mm | **Needs major fix** |

All four pairs route primarily on B.Cu (~30-40mm) with short F.Cu sections (~2mm) and 2 vias each. The via transitions are at (86.15, 91.90) on the CM5 side and near (100.0, 68-75) on the RJ45 side.

**Step-by-step fix:**

1. **TRD1 (1.641mm mismatch):**
   - The P trace (net 13, `/CM5/ETH_PI.TRD1_P`) is 33.03mm, N trace (net 102, `/CM5/ETH_PI.TRD1_N`) is 34.67mm
   - Use **Route → Tune Differential Pair Length**
   - Click on the TRD1_P trace (the shorter one)
   - Add serpentine on the B.Cu section between the two vias
   - Target: match N trace length within 0.15mm

2. **TRD3 (1.917mm mismatch):**
   - P trace (net 11) is 40.70mm, N trace (net 103) is 42.61mm
   - Add serpentine to TRD3_P to match TRD3_N
   - This needs ~1.9mm of additional length, so use a larger serpentine amplitude

3. **TRD0 and TRD2 (0.232mm mismatch):**
   - These are close but still over the 0.15mm target
   - Add small serpentine tuning to the shorter trace of each pair
   - TRD0: add ~0.08mm to P trace; TRD2: add ~0.08mm to P trace

4. **Inter-pair length matching for Gigabit:**
   - For 1000BASE-T, the four pairs should be within ~10mm of each other
   - Current range: 32mm (TRD0) to 43mm (TRD3) = ~11mm spread
   - This is borderline. Consider adding length tuning to TRD0 and TRD1 to bring them closer to TRD2/TRD3 lengths.
   - Alternatively, the CM5's built-in PHY may have sufficient receive-side skew compensation, but it's safer to length-match.

### Step 3.3 — HDMI Pairs (After Re-routing with Correct Net Class)

After fixing the net class assignments in Phase 2, you'll be re-routing the HDMI traces. During that re-routing:

1. Route each data lane (D0, D1, D2) and clock (CK) as differential pairs
2. Intra-pair length match each to < 0.2mm
3. Inter-lane length match all four pairs to within ~2mm of each other
4. The HDMI_PI nets (to the HDMI connector J501) and the HDMI1 nets (from the CM5 module) are different ends of the same signals — ensure continuity through the board-to-board connector routing

---

## Phase 4: Ground Plane Integrity

### Step 4.1 — Clean Up In2.Cu (Ground Plane)

**Current problem**: 125 signal trace segments are routed on In2.Cu, which is supposed to be a solid ground plane. These traces create slots in the ground fill that disrupt return current paths.

**What to do:**

1. **Switch to In2.Cu** in the layer selector
2. **Identify all signal traces** (everything that's not GND):
   - Select a non-GND trace → right-click → **Select → All Tracks in Net on This Layer**
   - Note which nets are present (you identified: VBUS_EN, PCIE control, CC1/CC2, PWM, SPI, LEDs, etc.)

3. **For each signal, try to reroute to a different layer:**

| Signal Group | Currently on In2.Cu | Move to | Notes |
|-------------|---------------------|---------|-------|
| PCIE_PI.PWR_EN, nRST, nWAKE, nCLKREQ | X=[93-118], Y=[63-93] | In3.Cu or F.Cu | These cross right under PCIe diff pairs on F.Cu — CRITICAL to move |
| CC1, CC2 | X=[85-125], Y=[84-102] | F.Cu via dedicated routing | USB-C configuration — low speed, can go anywhere |
| VBUS_EN | X=[81-97], Y=[81-101] | F.Cu or In3.Cu | Power control — not speed-critical |
| SCLK_GPIO21 | X=[111-131], Y=[55-86] | In3.Cu | SPI clock to Zigbee — medium speed |
| nLED_SPEED_100_1000 | X=[99-131], Y=[61-85] | In3.Cu or F.Cu | LED indicator — slow |
| PWM, BL_GPIO24, DC_GPI25 | Various | In3.Cu | GPIO — low speed |
| +3V3_PI | X=[106-129], Y=[98-104] | In3.Cu (power plane) | Power — belongs on power layer |
| nRPIBOOT, EEPROM_nWP | Small area | F.Cu or B.Cu | Rare use signals |
| ETH_nLED_ACTIVITY | X=[105-128], Y=[76-84] | In3.Cu | LED — low speed |

4. **Step-by-step for each signal:**
   - Select all segments of that net on In2.Cu
   - Delete them
   - Re-route on the target layer using additional vias as needed
   - **Priority order**: Move the PCIe control signals FIRST (they sit right under the PCIe data traces)

5. **After moving signals, re-fill the ground zones:**
   - Select the GND zone on In2.Cu
   - Press **B** to refill all zones
   - Visually inspect that In2.Cu now has a mostly solid ground fill with minimal interruptions

6. **If some signals absolutely MUST cross In2.Cu:**
   - Route them perpendicular to the high-speed traces above/below (minimize the slot length)
   - Add a **stitching via pair** on both sides of the crossing:
     - Place two GND vias, one on each side of the signal trace, spaced ~1mm apart
     - This creates a local return current bridge across the slot

### Step 4.2 — Add GND Reference to In3.Cu

**Current problem**: In3.Cu is entirely power zones (M2_3V3 + 5V) with no GND fill. B.Cu signals (793 traces) have no nearby ground reference — the closest ground plane is In4.Cu, which is 0.55mm core + 0.1mm prepreg = 0.65mm away.

**What to do:**

1. **Identify which B.Cu areas need ground reference the most:**
   - PCIe pairs route on B.Cu: X=[91-120], Y=[62-85]
   - Ethernet pairs route on B.Cu: X=[86-100], Y=[68-92]
   - CM5 high-speed area: around (127, 112) on B.Cu

2. **Edit the In3.Cu power zones to leave room for GND:**
   - Open the properties of each power zone on In3.Cu
   - Shrink the M2_3V3 zones to cover only the M.2 connector area (X=108-130, Y=60-96)
   - Shrink the +5V zone to cover only the power delivery path

3. **Add a GND zone on In3.Cu:**
   - Draw a new zone on In3.Cu, assign it to GND
   - Fill the areas NOT covered by power zones, particularly:
     - Under the PCIe routing area
     - Under the Ethernet routing area
     - Under the CM5 module area
   - Set the zone priority **lower** than the power zones so it fills in the gaps

4. **Refill all zones** (press **B**) and verify:
   - M2_3V3 still reaches the M.2 connector
   - +5V still reaches the buck converter area
   - GND fills in the gaps and provides reference under B.Cu high-speed signals

### Step 4.3 — Add Ground Stitching Vias

**Why**: Even with clean ground planes, the ground reference needs to be connected between layers, especially near high-speed signals and at zone boundaries.

**What to do:**

1. **Along the perimeter of the board** (every ~5mm):
   - Place GND vias around the board edge
   - Use the same via size (0.45/0.3mm) or slightly larger

2. **Near every differential pair via transition:**
   - For each via pair where a diff pair changes layers (you have 2 vias per pair for PCIe and Ethernet)
   - Add 2-4 GND stitching vias within 1mm of the signal vias
   - Place them flanking the signal vias, not between the P and N traces

3. **Around the Zigbee module** (see Phase 5)

4. **Between power zone islands on In3.Cu:**
   - Where the +5V and M2_3V3 zones meet, add GND vias to connect the GND zones on In1.Cu, In2.Cu, and In4.Cu

**In KiCad:**
- Select **Place → Via** (shortcut: **V**)
- Set the net to GND before placing
- Or use **Edit → Fill All Zones** after placing to ensure they connect

---

## Phase 5: RF Section — Zigbee/Thread Module

### Step 5.1 — Verify MGM240L Antenna Orientation

**Current placement**: U2 at (113.04, 42.61) on F.Cu, board edge at Y=36.36 (6.25mm from module center to edge).

**Check the MGM240LD22VIF2 datasheet** (Silicon Labs document):

1. Confirm which end of the module is the antenna end (typically the end opposite the pads)
2. The pads extend from Y≈42 to Y≈53 (local coordinates suggest pads run from the center downward/rightward)
3. The antenna end should face the nearest board edge

### Step 5.2 — Create Antenna Keepout Zone

**Why**: There are currently **16 vias at Y=38.86** (a row right between the module and the board edge!) plus another **12 vias at Y=41.36** — these are directly in the antenna's near-field region and will severely detune it.

**What to do:**

1. **Find the MGM240L module's antenna keepout specification** in the datasheet:
   - Typically: no copper (traces, pours, or vias) within ~5mm of the antenna element on ANY layer
   - No ground plane under the antenna area on ANY layer

2. **Create a Rule Area in KiCad:**
   - Go to **Place → Rule Area**
   - Draw a rectangle around the antenna zone. Based on the module at (113.04, 42.61) with antenna facing towards Y=36.36:
     - Approximate keepout box: X=[105, 121], Y=[36.36, 42]
     - Adjust based on datasheet recommendation
   - In the Rule Area properties:
     - Check **All copper layers** (F.Cu, In1.Cu, In2.Cu, In3.Cu, In4.Cu, B.Cu)
     - Check: **No tracks**, **No vias**, **No copper fill**
     - Give it a descriptive name: "MGM240L Antenna Keepout"

3. **Delete the existing vias in the keepout zone:**
   - The row of ~16 vias at Y=38.86 (X=101-124) MUST be removed
   - The vias at Y=41.36 in the keepout zone must also be removed
   - These appear to be ground stitching vias — they'll need to be relocated outside the keepout

4. **Modify ground zones on inner layers:**
   - Edit the GND zones on In1.Cu, In2.Cu, In4.Cu
   - Add the keepout area as an exclusion (the rule area should handle this automatically)
   - Refill zones — verify no copper fill appears under the antenna

5. **Add RF fence vias around the keepout boundary:**
   - Place GND vias at ~2mm spacing along three sides of the keepout zone (the fourth side is the board edge)
   - These create an RF shield that contains the antenna radiation pattern
   - Do NOT place vias on the board-edge side (that's where the radiation goes)

### Step 5.3 — Check SPI Routing to Zigbee Module

The MGM240L connects to the CM5 via SPI (SCLK, MOSI, MISO, CS) plus control signals (RST, IRQ, BOOT). These are currently routed through multiple layers including In1.Cu, In2.Cu, and In3.Cu:

- SCLK_GPIO21: 8 segments on In2.Cu (X=[111-131], Y=[55-86])
- CS_GPIO18: 13 segments on In3.Cu
- MOSI_GPIO20: 5 segments on In3.Cu
- MISO_GPIO19: 8 segments on In1.Cu

**Check these routes for:**
1. Do they cross under the antenna keepout zone? If so, reroute around it.
2. Are SCLK traces length-matched with MOSI/MISO? For SPI at moderate speeds (10-20 MHz), this isn't critical, but keep them roughly similar.
3. Add ground stitching vias near via transitions for these signals.

---

## Phase 6: Power Integrity

### Step 6.1 — Add B.Cu Decoupling Capacitors

**Current problem**: The CM5 module is on B.Cu at (127.25, 112.61) but ALL decoupling caps are on F.Cu, minimum 7.2mm away. The via inductance plus trace inductance degrades decoupling effectiveness.

**What to do:**

1. **Add 3-4 new 100nF 0402 caps on B.Cu** near the CM5 power pins:
   - One for each power rail: +5V, +3V3_PI, +1V8
   - Place them as close as possible to the CM5 connector power pins
   - Target placement: within 2mm of the power pins on B.Cu

2. **Connect each cap with short, wide traces:**
   - Power side: connect to the nearest power pin or via
   - GND side: connect to the nearest GND pad or via
   - Use trace widths ≥ 0.3mm for power connections

3. **Alternatively, use via-in-pad** on existing F.Cu caps:
   - If adding B.Cu components is difficult (may conflict with CM5 module keepout), ensure the existing F.Cu caps have vias directly in or adjacent to their pads, providing the shortest possible path through the board to the B.Cu power pins

### Step 6.2 — Widen Power Traces on Inner Layers

**Current problem**: +3V3_PI uses 0.15mm traces on inner layers (½oz copper = 15.2μm). At this width and thickness, the current capacity is approximately 0.3A with a 10°C temperature rise. The CM5 can draw up to 2A on the 3.3V rail.

**What to do:**

1. **Identify thin power segments:**
   - In KiCad, click on the +3V3_PI net, right-click → **Select → All Tracks in Net**
   - Look for segments with width < 0.3mm on inner layers
   - Same for +5V net

2. **Widen them:**
   - For +3V3_PI on inner layers: minimum 0.5mm width (can carry ~1A on ½oz)
   - For +5V: minimum 0.8mm width (needs to handle up to 5A from USB-C)
   - Where possible, use zone fills instead of traces for power distribution

3. **Check the buck converter output path:**
   - U702 (AP3441) output → L701 (inductor) → output caps → distribution
   - The LX (switching node) traces at 0.3-0.7mm are fine for the short distances
   - But the output distribution to the M.2 connector and CM5 should be wide or zone-based

### Step 6.3 — Verify Bulk Capacitance Near M.2

**Current**: Only C710 (10μF) and C712 (100nF) near the M.2 connector at (113.60, 65.74).

**Add:**
1. One 47μF or 100μF capacitor (1206 or tantalum package) near the M.2 3.3V power input
2. Place it on F.Cu within 5mm of the M.2 connector's power pins
3. This handles the transient current spikes when the NVMe drive wakes from low-power states

---

## Phase 7: Via Improvements

### Step 7.1 — Increase Via Annular Ring

**Current**: All vias are 0.45mm pad / 0.3mm drill = 0.075mm annular ring (exactly JLCPCB minimum).

**What to do:**

1. Go to **File → Board Setup → Design Rules → Vias**
2. Change the via size from 0.45mm to **0.5mm** (keeping 0.3mm drill)
3. This gives 0.1mm annular ring — much better manufacturing margin
4. **Note**: This means every via in the design gets slightly larger. Check for clearance violations after the change by running DRC.

5. **Alternatively, use a selective approach:**
   - Keep 0.45mm vias for dense areas (under CM5 module, between fine-pitch pads)
   - Use 0.5mm or 0.6mm vias for power connections and ground stitching

### Step 7.2 — Add Return Current Vias at Every Layer Transition

For every via pair where a differential pair transitions between layers, you need nearby ground vias:

**PCIe TX/RX (transition around X=93, Y=84-85 and X=120, Y=60-62):**
1. At each via pair, place 2 GND vias within 0.5-1mm
2. Place them to the sides, not between P and N

**Ethernet (transitions at X=86 and X=100):**
1. Same principle — 2 GND vias near each pair's transition vias
2. Since all 8 Ethernet vias cluster near X=100, Y=68-75, you can share some ground vias between pairs

**PCIe CLK (transitions around X=118-120):**
1. Add 2 GND vias flanking the CLK pair's via transitions

---

## Phase 8: Solder Mask & DFM Cleanup

### Step 8.1 — Increase Solder Mask Expansion

**In KiCad:**

1. Go to **File → Board Setup → Design Rules → Solder Mask/Paste**
2. Change **Pad to Mask Clearance** from 0mm to **0.051mm** (2 mils)
3. This adds a 51μm expansion around each pad for solder mask registration tolerance

### Step 8.2 — Verify DRC Clean

After all changes:

1. Run **Inspect → Design Rule Check** (DRC)
2. Address ALL errors:
   - Clearance violations (may appear after via size increase)
   - Unconnected nets
   - Track width violations (after net class changes)
   - Zone fill issues
3. Address warnings where reasonable:
   - Silk-to-pad clearance
   - Courtyard overlaps

### Step 8.3 — Add Additional Test Points

Add SMD test point pads (1mm round pads) for the following signals:

| Signal | Net Name | Suggested Location |
|--------|----------|--------------------|
| SPI SCLK | /CM5/SCLK_GPIO21 | Near U2 Zigbee module |
| SPI MOSI | /CM5/MOSI_GPIO20 | Near U2 |
| SPI MISO | /CM5/MISO_GPIO19 | Near U2 |
| SPI CS | /CM5/CS_GPIO18 | Near U2 |
| I2C SDA | /IO/SDA | Near J502 I2C connector |
| I2C SCL | /IO/SCL | Near J502 |
| 1.8V | +1V8 | Near CM5 module |
| M2 3.3V | M2_3V3 | Near M.2 connector |

---

## Phase 9: Final Verification Checklist

Before exporting Gerbers:

1. **Run DRC** — zero errors
2. **Run ERC on schematic** — zero errors
3. **Refill all zones** (press B) — check for isolated copper islands
4. **Check impedance** — verify trace widths match calculated values for confirmed stackup
5. **Verify length matching:**
   - PCIe intra-pair: < 0.127mm (5 mils)
   - Ethernet intra-pair: < 0.15mm
   - HDMI intra-pair: < 0.2mm
6. **Visual inspection per layer:**
   - F.Cu: clean routing, no unexpected copper
   - In1.Cu: solid GND fill, minimal interruptions
   - In2.Cu: solid GND fill (after cleanup), no signal slots under high-speed areas
   - In3.Cu: power zones + GND fill in gaps
   - In4.Cu: solid GND fill
   - B.Cu: clean routing, decoupling caps present
7. **Antenna keepout**: no copper on any layer in the keepout zone
8. **Export Gerbers** with impedance control notes:
   - Include the stackup specification as a manufacturing note
   - Specify "Impedance Controlled" in the JLCPCB order
   - Upload the stackup document with your order

---

## Quick Reference: KiCad Shortcuts

| Action | Shortcut |
|--------|----------|
| Route differential pair | D, then 6 |
| Tune diff pair length | (check Route menu) |
| Place via | V |
| Fill all zones | B |
| Run DRC | (Inspect menu) |
| Select all tracks in net | Right-click → Select → All Tracks in Net |
| Switch active layer | Page Up / Page Down or + / - |
| Net Inspector panel | View → Panels → Net Inspector |

---

*Generated from analysis of CM5_MINIMA_3.kicad_pcb (Rev 2), CM5_MINIMA_3.kicad_sch (Rev 3.1), and CM5_MINIMA_3.kicad_pro*
