# CM5_MINIMA_3 PCB Design Review

**Board**: CM5 MINIMA Rev 3.1 by Pierluigi Colangeli  
**Reviewer**: Naveen (PCB/Embedded review)  
**Date**: March 2026  
**Design Tool**: KiCad 9.0  
**Target Fab**: JLCPCB, 6-layer ENIG

---

## Executive Summary

I've done a thorough analysis of the KiCad project files (schematic + PCB). The design is quite ambitious for a first 6-layer board and shows good understanding of the CM5 ecosystem. However, there are several issues ranging from **critical** (will cause functional failures) to **advisory** (improvements for next rev). I've organized by severity.

---

## CRITICAL Issues (Must Fix Before Fab)

### 1. PCIe Differential Pair Length Mismatch — ~1.35mm

The PCIe TX, RX, and CLK differential pairs have intra-pair length mismatches of approximately 1.34–1.36mm (2.4–3.0%). For PCIe Gen2 at 5 GT/s, the maximum allowable intra-pair skew is typically **≤0.127mm (5 mils)**. Your current mismatches are roughly **10× over spec**.

**Measured values:**
| Pair | P length | N length | Mismatch |
|------|----------|----------|----------|
| PCIe TX | 45.21mm | 46.57mm | 1.358mm |
| PCIe RX | 44.02mm | 45.36mm | 1.340mm |
| PCIe CLK | 54.08mm | 55.44mm | 1.358mm |

**Fix**: Add serpentine length-matching tuning on the shorter trace of each pair. KiCad 9 has a built-in differential pair tuning tool (Route > Tune Diff Pair Length). Target <0.127mm mismatch within each pair.

### 2. Ethernet Pair Length Mismatches — TRD1 and TRD3

Two of four Ethernet pairs have significant intra-pair mismatches:

| Pair | P length | N length | Mismatch |
|------|----------|----------|----------|
| ETH TRD0 | 32.42mm | 32.19mm | 0.232mm ✓ |
| ETH TRD1 | 33.03mm | 34.67mm | **1.641mm** ✗ |
| ETH TRD2 | 38.64mm | 38.41mm | 0.232mm ✓ |
| ETH TRD3 | 40.70mm | 42.61mm | **1.917mm** ✗ |

For 1000BASE-T, intra-pair skew should be **<0.15mm**. TRD1 and TRD3 are about 10× over limit. TRD0 and TRD2 are also slightly over but may work.

**Fix**: Apply differential pair length tuning to all four Ethernet pairs. Also verify inter-pair length matching (pair-to-pair) is within 10mm for Gigabit operation.

### 3. Net Class Pattern Coverage Gaps

Your netclass patterns are:
- `90ohm`: `/*USB_*`, `/*PCIE*`, `/*USB2*`
- `100ohm`: `/*ETH_PI*`, `/*HDMI_P*.D*`, `/*DPHY0*`, `/*HDMI_P*.CK*`

**Issues identified:**
- **HDMI1 nets** (`/CM5/HDMI1_D0_P`, `/CM5/HDMI1_CK_N`, etc.) do **NOT** match the `/*HDMI_P*` pattern. These CM5 connector-side HDMI signals are routing at the Default class (0.178mm width, 0.25mm gap) instead of 100Ω differential.
- **USB_C.D_P / USB_C.D_N** (nets 91, 93) may not match `/*USB_*` depending on hierarchy path resolution.
- **USB3 nets** (`/CM5/USB3-0-TX_P`, etc.) also don't obviously match `/*USB_*` — verify these are being assigned to 90ohm.

**Fix**: Add patterns for the missing net groups or use explicit netclass assignments. Test by running DRC and checking which nets have impedance violations.

---

## HIGH Priority Issues (Likely to Cause Problems)

### 4. In2.Cu Ground Plane Integrity — 125 Signal Traces Cutting Through

In2.Cu is designated as a GND plane but has 125 signal trace segments routed through it, including:
- PCIE_PI control signals (PWR_EN, nRST, nWAKE, nCLKREQ)
- CC1/CC2 (USB-C configuration)
- VBUS_EN, PWM, LED signals
- SPI signals (SCLK, CS, MOSI, MISO) to the Zigbee module

This creates **ground plane slots** that disrupt return current paths for the high-speed signals on F.Cu and B.Cu that reference In2.Cu as their ground plane.

**Fix**: 
- Move as many In2.Cu signals as possible to In3.Cu (power plane layer) where ground return continuity is less critical.
- For signals that must cross the ground plane, add **stitching vias** on both sides of the crossing to provide return current paths.
- Especially critical: the PCIe control signals should NOT cut under the PCIe differential pair routing area.

### 5. In3.Cu Power Plane Split — No GND Reference for B.Cu Signals

In3.Cu is split between M2_3V3 and +5V power zones with **no GND fill**. B.Cu signals (793 segments) that need to reference a ground plane can only see In4.Cu (through a 0.55mm core — very far). The nearest ground plane above is In2.Cu, but that's across a 0.13mm prepreg + 0.5mm core = 0.63mm total. 

For the high-speed signals on B.Cu (PCIe, Ethernet, HDMI, USB3 all route on B.Cu), this means:
- Poor impedance control (reference plane too far away)
- Increased crosstalk
- EMI issues

**Fix**: Consider changing the stackup to: **Sig-GND-Sig-Pwr-GND-Sig** or redesigning In3.Cu to include GND fills between the power islands, especially under high-speed routing areas on B.Cu.

### 6. Asymmetric Stackup — Potential Warpage

Your stackup is not symmetric about the center:
```
F.Cu    (0.035mm)
Prepreg (0.0994mm)  ← thinner
In1.Cu  (0.0152mm)
Core    (0.500mm)   ← thinner
In2.Cu  (0.0152mm)
Prepreg (0.130mm)   ← center prepreg
In3.Cu  (0.0152mm)
Core    (0.550mm)   ← thicker
In4.Cu  (0.0152mm)
Prepreg (0.0994mm)  ← thinner
B.Cu    (0.035mm)
```

The two cores are asymmetric (0.5mm vs 0.55mm), which can cause board warpage during reflow, especially problematic for the fine-pitch CM5 connectors and the BGA-area pads on the bottom.

**Fix**: Ask JLCPCB for their standard 6-layer stackup and match it exactly. Their JLC06161H-1080 stackup is symmetric and well-characterized for impedance.

### 7. Board Thickness Mismatch

Your stackup totals **1.5296mm** but you mention targeting 1.6mm. This 70μm difference will affect impedance calculations. Either adjust the stackup to hit 1.6mm or recalculate impedance for the actual thickness.

---

## MEDIUM Priority Issues (Should Address)

### 8. Zigbee/Thread Module (MGM240L) Placement and Antenna

The MGM240LD22VIF2 SiP module is at position (113.04, 42.61) on F.Cu. The nearest board edge appears to be at Y=36.36 (about 6.3mm from the module center to the edge). 

**Observations:**
- The MGM240L has an **integrated chip antenna** — the module datasheet specifies a keepout zone around the antenna area. Verify your placement respects the minimum keepout from the datasheet (typically 5-10mm clear of copper on all layers near the antenna end).
- I don't see a dedicated RF keepout rule area defined in the PCB around the antenna.
- There are **no ground plane cutouts** visible under the antenna area — the GND fills on In1.Cu and In2.Cu likely extend under the antenna, which can detune it.

**Fix**: 
- Add a rule area / keepout around the antenna end of the MGM240L that prohibits copper on ALL layers.
- Remove ground fills under the antenna area on all inner layers.
- Ensure the antenna faces the board edge with no copper or components between it and the edge.
- Add a row of ground stitching vias at the boundary of the keepout zone (RF fence).

### 9. Via Design — Single Size, All Through-Hole

All 539 vias are identical: 0.45mm pad, 0.3mm drill, through-hole F.Cu→B.Cu. While this simplifies manufacturing, it means:
- Every via on high-speed signals creates a **full-height stub** on unused layers, causing reflections at GHz frequencies (relevant for USB3, PCIe).
- The annular ring is exactly 0.075mm — right at JLCPCB's minimum. No margin for drill wander.

**Recommendations:**
- Increase via pad to 0.5mm (0.1mm annular ring) for better manufacturing yield.
- For the next revision, consider using **back-drilled vias** or requesting JLCPCB's via-in-pad capability for critical high-speed transitions.

### 10. Decoupling Cap Placement for CM5 Module

The CM5 module (GPIO + HSS connectors) is on **B.Cu** at position (127.25, 112.61), but all decoupling capacitors are on **F.Cu**, with the closest being 7.2mm away. For effective decoupling:
- The caps need a short, low-inductance path to the power pins.
- Having caps on the opposite side of the board from the CM5 module introduces via inductance for every power connection.

**Recommendation**: Place at least some 100nF decoupling caps on B.Cu, directly adjacent to the CM5 power pins. Use via-in-pad or place vias immediately next to the cap pads.

### 11. Power Trace Width Inconsistency

The +3V3_PI net (net 53) uses trace widths ranging from 0.15mm to 1.0mm across different layers. Similarly, +5V (net 96) ranges from 0.2mm to 0.8mm. For a 5A USB-C input:
- 0.15mm trace on inner layers (0.0152mm copper = ½oz) can carry roughly 0.3A before exceeding a 10°C rise.
- The CM5 can draw 2-3A on 5V alone.

**Fix**: Ensure all power traces are adequately sized for their current. Use the zone fills for power distribution rather than thin traces. The 0.15mm segments on the 3V3 and 5V nets need widening or replacing with zone connections.

---

## LOW Priority / Advisory

### 12. Solder Mask Settings

Pad-to-mask clearance is set to 0mm with solder mask-to-copper clearance of only 0.005mm. JLCPCB recommends a minimum of 0.05mm mask expansion for reliable solder mask registration. With 0mm, any registration shift could cause solder mask to overlap pads, particularly problematic for 0402 components.

**Recommendation**: Set pad_to_mask_clearance to at least 0.05mm (50μm).

### 13. Test Point Accessibility

You have 6 test points (VBUS, +5V_PI, +3V3_PI, and 3× GND). For a home automation hub that may need field debugging, consider adding test points for:
- SPI bus to Zigbee module (SCLK, MOSI, MISO, CS)
- I2C bus (SDA, SCL)
- UART (if available from CM5)
- 1.8V rail
- PCIe reset/wake signals

### 14. Missing Bulk Decoupling Near M.2 Connector

The M.2 NVMe connector at (113.60, 65.74) has M2_3V3 power zones but I only see C710 (10μF) and C712 (100nF) nearby. NVMe drives can have significant transient current draws. Consider adding a 47-100μF bulk cap near the M.2 connector.

### 15. Edge Cuts Geometry

The board outline includes points ranging from X=75 to X=274.6 and Y=20.1 to Y=118.6, giving a rough footprint of 199.5 × 98.5mm. However, the actual PCB components are concentrated in a much smaller area (roughly X=78-133, Y=36-117). The extended outline to X=274 appears to include the M.2 drive mounting area. Verify this matches your enclosure design.

---

## Summary of Recommended Actions

**Before fab (must fix):**
1. Length-match all PCIe differential pairs to <0.127mm intra-pair skew
2. Length-match all Ethernet differential pairs to <0.15mm intra-pair skew
3. Fix netclass pattern assignments for HDMI1 and USB nets
4. Verify stackup with JLCPCB's standard 6-layer offering

**Before fab (strongly recommended):**
5. Reduce signal routing on In2.Cu ground plane
6. Add GND reference regions on In3.Cu under B.Cu high-speed routing
7. Add proper antenna keepout zones for MGM240L on all copper layers
8. Increase via annular ring to 0.1mm minimum

**For next revision:**
9. Add B.Cu decoupling caps near CM5 module
10. Increase power trace widths on inner layers
11. Add more test points for debug
12. Consider symmetric stackup
