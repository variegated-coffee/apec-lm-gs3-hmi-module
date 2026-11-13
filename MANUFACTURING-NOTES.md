# LM GS3 HMI Module manufacturing notes

How to fabricate, assemble and bring up the module, revision R1A. R1A hasn't been built yet; EV1 has. Values below come from the R1A design files (KiCad 10). The production files are set up for JLCPCB, but any fab and assembler with the same capabilities will do.

For pinouts and configuration, see [ENGINEERING-NOTES.md](ENGINEERING-NOTES.md).

## PCB fabrication

| Parameter | Value |
| --------- | ----- |
| Layers | 2 |
| Size | 126.5 × 71.5 mm |
| Thickness | 1.6 mm |
| Material | FR-4 |
| Copper weight | 35 µm (1 oz) on both layers |
| Surface finish | Not specified in the design; choose ENIG or lead-free HASL |
| Default net class | 0.2 mm track, 0.19 mm clearance, 0.6 mm via with 0.3 mm drill |
| PWR_DIST net class (DISP_VDD, DISP_GND) | 0.4 mm track, 0.2 mm clearance |
| Smallest vias | 0.4 mm with 0.2 mm drill: ten DISP_GND vias at U4 and U5. EV1 was made with the same vias. |
| Copper to board edge | 0.5 mm minimum |
| Impedance control | Not specified in the design |

## Assembly

* **Both sides have parts.** The switches (SW1–SW12) and RGB LEDs (D8, D10, D12, D14, D15, D16) are on the bottom side; everything else is on the top.
* **Assembly service parts:** the SMD parts carry LCSC part numbers in the schematic. The production files are generated with the Fabrication Toolkit plugin for KiCad, which writes them to `production/` (Gerber zip, BOM, placement file, IPC netlist). Its settings are in `fabrication-toolkit-options.json`. The plugin takes LCSC numbers from the footprints on the PCB, so sync them from the schematic first (see the checklist below).
* **Switches:** the schematic specifies the Omron B3FS-1050P (LCSC C231326) for SW1–SW12. It works, but it's expensive. The E-Switch TL3301TF100QG is a cheaper alternative that fits the same footprint; to use it, change the switches' LCSC number before ordering, or fit it yourself.
* **No LCSC number:** J4, the 2×6 header for external buttons and LEDs. Source and fit it yourself, or add an LCSC number before ordering. It's only needed if you use external buttons or LEDs.
* **Not fitted (DNP):** J5 and J6, the breakout headers.
* **Not parts:** JP1 and JP2 are solder jumpers (JP1 is bridged in copper, JP2 is open), H5 is the outline of the display bracket, and H1–H4 and H9–H17 are unplated mounting holes.
* **The display** isn't on the BOM. Buy it from [BuyDisplay](https://www.buydisplay.com/wide-angle-2-79-inch-142x428-color-ips-tft-display-nv3007-controller) and plug its FFC into J2.
* **Rotations:** check the placement preview for rotation errors before confirming the order.

## Display bracket

The display bracket positions the display on the module. Print [`GS3 TFT Shim.stl`](GS3%20TFT%20Shim.stl) (the file name still says "shim"). `GS3 TFT Shim.step` is the same part, used as the 3D model of its outline, H5, on the bottom side of the PCB.

It has been printed at JLC3DP with MJF in PA12-HP nylon, black, which turned out very well.

## Before ordering

1. **Sync part numbers:** in the PCB editor, run Tools → Update PCB from Schematic with "Update footprint fields" enabled, so the LCSC numbers on the footprints match the schematic.
2. **DRC:** refill all zones and run DRC. Review every error; the design should have no unconnected items. R1A's DRC reports these errors, all of which are in the design as it stands:
   * Clearance between U1's pads (0.19 mm against 0.2 mm). That's the pitch of the LM3410's vendor footprint.
   * J2: copper polygons inside the vendor footprint touch its mounting pads P1/P2 and share their mask opening.
   * J12: a malformed courtyard in the vendor footprint.
   * The ten 0.4/0.2 mm vias at U4 and U5, which are below the board setup's 0.5/0.3 mm minimum. Check that your fab can make them.
   * Starved thermal reliefs (one spoke) on pin 2 of J5, J6 and C9.

   Warnings that a footprint's library isn't found are expected for the embedded-only parts listed in [THIRD_PARTY.md](THIRD_PARTY.md).
3. **Outputs:** generate the production files and check them in a Gerber viewer before uploading.

## Bring-up

1. **Check for shorts.** Before connecting anything, measure between +3V3 and GND (for example at J3 pins 2 and 1).
2. **Connect to the carrier** with one cable, not both: on the bench, an EyeSPI FFC cable on J12 is usually easiest; in the machine, use an IDC ribbon cable on J13. Power the carrier from USB for bench work.
3. **Backlight.** U1 starts at full current as soon as +3V3 is present, unless the carrier holds GP1 low. With the display connected, the backlight should come on straight away.
4. **I2C.** Scan the carrier's I2C0. Expect 0x27 (MCP23008) and 0x50–0x57 (the two LP5813s), as well as the carrier's own devices (the DS3231 at 0x68).
5. **Buttons.** Read the MCP23008's GPIO register: each button reads 0 while pressed. Check both switches of every button.
6. **LEDs.** Keep EYESPI_TS_CS (carrier GP15) high or as an input, so the LED drivers are enabled, then light each colour of each LED. [IO.md](IO.md) has the matrix.
7. **Display.** Initialise the NV3007 over SPI and check the picture and the tearing-effect signal on GP22.
