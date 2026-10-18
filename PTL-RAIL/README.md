# PTL-RAIL — modular IR TX/RX rail, rev A

KiCad 10 project for the Meesho put-to-light rail. One board carries five IR
emitter/receiver nodes, an ESP32-S3 controller and a CAN interface. Boards
daisy-chain through screw terminals.

| | |
|---|---|
| Board size | 400 × 40 mm, 4 mm corner radius (concentric with the corner holes) |
| Layers | 2 (F.Cu / B.Cu), 1.6 mm, GND pour on both layers with edge via fence |
| Nodes | 5, on an 84 mm pitch (centres at x = 32, 116, 200, 284, 368 mm from the left edge) |
| Controller | ESP32-S3-WROOM-1 (U3, onboard antenna, overhangs the top edge) |
| Bus | CAN, SN65HVD230 (U5) with PESD1CAN (CR1) and switchable 120 Ω termination (SW3 + R7) |
| Power | VIN (24 V nominal) → TPS54360B buck (U2, L1 15 µH, D1 SS36) → +5 V → TLV1117-33 (U1) → +3V3 |
| USB | J1 JST-PH 4-pin (5 V via D3 SS14, USBLC6-2 ESD on D+/D−) |

## Block overview

- **IR emitter row (top edge):** D7–D11 IR333C, 5 mm, mounted horizontally and firing out of the top edge. Each LED is switched by an AO3400A low-side MOSFET (Q1–Q5) with a 100 Ω gate resistor, a 100 kΩ gate pull-down and a 180 Ω LED resistor from +5 V.
- **IR receiver row (bottom edge):** U6–U10 TSOP382xx, lens facing out of the bottom edge. Each has a 100 Ω supply resistor and 4.7 µF on Vs (+3V3), plus a 100 Ω series resistor on the output.
- **Shrouds:** each emitter and each TSOP sits under a 3D-printed shroud (28 mm silk outline). Each shroud is held by two M3 screws at the node centre ±10 mm, which reduces reflections and narrows the field of view.
- **Status:** D4–D6 WS2812B chain on IO18; D2 power LED.
- **User inputs:** SW1 = RST (EN), SW2 = BOOT (IO0), SW4 = ACK_BUTTON (IO14, 10 kΩ pull-up R34, 100 nF C27).

## Connector pinouts

Pin labels are printed on both silkscreen sides.

**J3 — 10-way, TSOP OUT (towards the board below)**

| Pin | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Signal | RX1 | RX2 | RX3 | RX4 | RX5 | CAN_H | CAN_L | VIN | GND | GND |

**J4 — 10-way, TSOP IN (from the board above, into the ESP32)**

| Pin | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Signal | GND | GND | VIN | CAN_L | CAN_H | RX5 | RX4 | RX3 | RX2 | RX1 |

On J3, RXn is this board's TSOP output (TSOP_OUT_n). On J4, RXn is TSOP_OUT_n_ESP.

**J5 (left end) / J6 (right end) — 4-way CAN + power, mirrored**

| Pin | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| J5 | CAN_H | CAN_L | VIN | GND |
| J6 | GND | VIN | CAN_L | CAN_H |

**J1 — JST-PH 4-way USB:** 1 GND, 2 D+, 3 D−, 4 5 V (VBUS, via D3).

## ESP32-S3 pin map (U3)

| Function | GPIO | Module pin |
|---|---|---|
| TX_1 … TX_5 (emitter gates) | IO9 … IO13 | 17 … 21 |
| TSOP_OUT_1 … 5 _ESP (from J4) | IO4, IO5, IO6, IO7, IO8 | 4, 5, 6, 7, 12 |
| CAN_TX / CAN_RX | IO15 / IO16 | 8 / 9 |
| STATUS_LED (WS2812B DIN) | IO18 | 11 |
| ACK_BUTTON (SW4) | IO14 | 22 |
| USB D− / D+ | IO19 / IO20 | 13 / 14 |
| BOOT (SW2) | IO0 | 27 |
| EN (SW1) | EN | 3 |

All other GPIOs are unconnected. IO45 has a no-connect flag.

## Test points

| TP | Net | Footprint |
|---|---|---|
| TP1 / TP2 | USBC_D+ / USBC_D− (connector side) | Pad Ø1.5 mm |
| TP3 / TP4 | USB_D+ / USB_D− (ESP32 side) | Pad Ø1.5 mm |
| TP5 … TP9 | TX_1 … TX_5 | Loop Ø1.8 mm, drill 1.0 mm |
| TP10 … TP14 | TSOP_OUT_1 … TSOP_OUT_5 | Loop Ø1.8 mm, drill 1.0 mm |
| TP15 / TP16 / TP17 | VIN / +5V / +3V3 | Loop Ø1.8 mm, drill 1.0 mm |
| TP18 / TP19 | GND | Loop Ø1.8 mm, drill 1.0 mm |

## Mounting

- **Rail fixing:** H1–H4 (corners, M3 with plated pad, 4 mm from each edge), plus H25–H28 mid-board (M3 with plated pad).
- **Shroud fixing:** H5–H24 are NPTH Ø3.2 mm. They sit at each node centre ±10 mm, at y = 5.0 mm (LED side) and 35.0 mm (TSOP side) from the top edge.
- **Antenna:** the ESP32 antenna overhangs the top edge. Keep the rack upright and any metal clear of it.

## Fabrication notes

- 2-layer FR-4, 1.6 mm. Copper weight and surface finish are not specified in the design; 1 oz and HASL-LF or ENIG are suitable.
- Design rules: min track 0.2 mm, min clearance 0.2 mm, via 0.6 mm / 0.3 mm drill, min hole 0.3 mm, copper-to-edge 0.5 mm, hole-to-hole 0.25 mm.
- Some footprints (the U3 thermal pad vias) use 0.2 mm holes, below the 0.3 mm design minimum. That DRC check is set to ignore, so confirm your fab supports it.
- **Panel:** 2-up, 400 × 127 mm. Board 2 is rotated 180° so both antenna overhangs land on the 20 mm top/bottom rails. Boards are joined by 7 mouse-bite tabs per long edge (Ø0.5 mm, 0.8 mm pitch, 3 mm wide), placed clear of the LEDs, TSOPs, connectors and antenna. The rails carry 3 fiducials and 3 tooling holes.

## Assembly notes

- **Top side:** ESP32, power stage, CAN, WS2812s, switches, connectors, IR LEDs and TSOPs.
- **Bottom side:** all node passives and MOSFETs (R9–R33, C22–C26, Q1–Q5).
- **Through-hole:** IR LEDs, TSOPs, screw terminals, J1, SW3 and the loop test points.

## Production files (release assets)

| File | Content |
|---|---|
| `PTL-RAIL_revA_BOM.csv` | BOM grouped by Value + Footprint (test points and holes excluded) |
| `PTL-RAIL_revA_gerbers_drill_panel-2up.zip` | Gerbers, Excellon drill (PTH/NPTH) and drill maps for the 2-up panel, plus `assembly/` panel pick-and-place |
| `PTL-RAIL_revA_ibom.html` | Interactive HTML BOM |
| `PTL-RAIL_revA_netlist.ipc` | IPC-D-356 netlist for bare-board test |
| `PTL-RAIL_revA_pick-and-place.csv` | Single-board pick-and-place, both sides, mm |
| `PTL-RAIL_revA_refdes.csv` | Reference designator list (value, footprint, side, X/Y, rotation) |
| `PTL-RAIL_revA_schematic.pdf` | Schematic, all sheets |
| `PTL-RAIL_revA_kicad-source.zip` | KiCad project, schematics, PCB, local libraries and the panel PCB |

## Known open issues (rev A)

1. **No 24 V input protection.** VIN goes straight to C1/C3 and U2: no fuse, reverse-polarity protection, TVS or bulk capacitor.
2. **CAN_L sits next to VIN** on J3 (7/8), J4 (3/4), J5 (2/3) and J6 (2/3). The SN65HVD230 bus pins are rated −4 … +16 V, so a 24 V short destroys the transceiver. Either separate the pins with GND or use a ±58/70 V part (e.g. TCAN1042, MCP2562FD).
3. **J3/J4 pin order is reversed** (J3.1 = RX1, J4.10 = RX1). Check the harness wiring between boards.
4. **J4 inputs are unprotected.** IO4–IO8 connect straight to an off-board connector with no series resistor, pull-up, RC filter or ESD protection.
5. **WS2812B data level.** It is driven at 3.3 V while the LEDs run on 5 V (V_IH = 0.7 × VDD), which is marginal. Add a level shifter or a diode drop on the LED supply.
6. **No capacitor voltage ratings.** C1/C3 (on 24 V) need ≥ 50 V. No MPNs are assigned yet.
7. **U3 overlaps the node-3 emitter shroud and hole H19** (DRC exclusion set). The shroud screw lands near the module corner.
