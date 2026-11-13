# LM GS3 HMI Module engineering notes

What you need to know to connect, configure and write firmware for the module, revision R1A. Signal, I2C and LED assignments are in [IO.md](IO.md). Net names match the schematic.

## Connecting to the carrier

The module connects to the APEC carrier with one cable, on either of two connectors. They carry the same signals in parallel, so use only one:

* **J13**, a 2×10 2.54 mm box header (SMD), for an IDC ribbon cable to the carrier's J13. The pinout is the same as the carrier's J13. This is the one to use in a GS3.
* **J12**, an 18-pin EyeSPI FFC connector (Molex 505110-1892), for an EyeSPI cable to the carrier's J17. This is handier on the bench.

Pins 19 (VIN, about 12 V on the carrier) and 20 (+5V) of J13 aren't connected on the module.

## Power

| Rail | Source | Feeds |
| ---- | ------ | ----- |
| DISP_VDD (+3V3) | The carrier's +3V3, on J12 pin 1 and J13 pin 17 | Everything: the display logic, U1 (backlight), U2 (MCP23008), U4 and U5 (LP5813, which boost it for the LEDs), the button pull-ups and J3 |

DISP_GND is the only ground. Nothing on the module generates its own supply, apart from the boost converters inside U1, U4 and U5.

## Display (J2)

The display is a 2.79" 142×428 IPS TFT with an NV3007 controller, from [BuyDisplay](https://www.buydisplay.com/wide-angle-2-79-inch-142x428-color-ips-tft-display-nv3007-controller). Its 10-pin FFC plugs into J2 (Molex 52745-1097). The module only renames the carrier's EyeSPI signals, without buffering:

| J2 pin | Net | Function | From/to the carrier |
| ------ | --- | -------- | ------------------- |
| 1 | DISP_SDA | SPI data in | EYESPI_MOSI |
| 2 | DISP_SCL | SPI clock | EYESPI_SCK |
| 3 | DISP_RS | Data/command | EYESPI_DC |
| 4 | DISP_RSTB | Reset | EYESPI_RES |
| 5 | DISP_CS | Chip select | EYESPI_DISP_CS |
| 6 | DISP_TE | Tearing effect output | EYESPI_BUSY_TE |
| 7 | DISP_VDD | +3V3 | |
| 8 | DISP_LEDK | Backlight cathode | U1 FB, R1 |
| 9 | DISP_LEDA | Backlight anode | U1 output, through D1 |
| 10 | DISP_GND | Ground | |

The display has no data output, so EYESPI_MISO only goes to the J6 breakout.

## Backlight

U1 (TI LM3410) drives the display backlight as a SEPIC converter from +3V3: L1 and L2 (15 µH), C3 (2.2 µF coupling capacitor), D1 (BAT54) and C2 (4.7 µF output capacitor). A SEPIC can make an output voltage above or below its input, so the backlight's forward voltage doesn't have to match the 3.3 V supply.

* **Current:** R1 (2.49 Ω) sets the LED current to V_FB / R1 = 190 mV / 2.49 Ω ≈ 76 mA. To change it, change R1, and check the display's backlight rating first.
* **Dimming:** U1's DIM pin is EYESPI_BL (carrier GP1), pulled up by R2 (10K). The backlight is on at full current as soon as the module has power, unless the carrier drives GP1 low. PWM on GP1 dims it.

## Buttons

Six buttons, each with two tactile switches wired in parallel (Omron B3FS-1050P in the BOM; the cheaper E-Switch TL3301TF100QG fits the same `SW_Flexible` footprint), one on each side of the button's LED (19 mm apart). All switches and LEDs are on the bottom side of the board. Each button pulls its net to ground, against a 10K pull-up to +3V3, so the inputs are active low and the MCP23008's internal pull-ups aren't needed.

| Button | Silkscreen | Switches | Net | Pull-up | MCP23008 pin |
| ------ | ---------- | -------- | --- | ------- | ------------ |
| 1 | Decr | SW1, SW7 | BTN_1 | R3 | GP5 |
| 2 | Incr | SW2, SW8 | BTN_2 | R4 | GP4 |
| 3 | Sel | SW3, SW9 | BTN_3 | R5 | GP3 |
| 4 | Ret | SW4, SW10 | BTN_4 | R6 | GP2 |
| 5 | Fn | SW5, SW11 | BTN_5 | R7 | GP1 |
| 6 | Tea | SW6, SW12 | BTN_6 | R8 | GP0 |
| External 1 | BTN_EXT_1 | J4 pin 1 (pin 2 is GND) | BTN_EXT_1 | R9 | GP7 |
| External 2 | BTN_EXT_2 | J4 pin 3 (pin 4 is GND) | BTN_EXT_2 | R12 | GP6 |

U2 (MCP23008) sits on the I2C bus at 0x27 (A0–A2 tied to +3V3), with ~RESET tied to +3V3. Its INT output goes to EYESPI_INT (carrier GP21), so firmware can use interrupt-on-change instead of polling.

## Button LEDs

Each button has an RGB LED (ASMT-YTC2-0AA02, PLCC6) between its two switches. Two LP5813 LED drivers run them, each from +3V3 through its own boost converter:

| Driver | I2C address | Broadcast | LEDs | Boost parts |
| ------ | ----------- | --------- | ---- | ----------- |
| U4 (LP5813A, "driver A") | 0x50–0x53 | 0x6C–0x6F | D8, D10, D12 (buttons 1–3), and one external LED on J4 | L4 (1 µH), C12 (22 µF), C4 (10 µF input) |
| U5 (LP5813B, "driver B") | 0x54–0x57 | 0x6C–0x6F | D14, D15, D16 (buttons 4–6), and one external LED on J4 | L3 (1 µH), C11 (22 µF), C9 (10 µF input) |

Each LP5813 answers on four consecutive addresses, and both answer on the broadcast range. The two drivers' SYNC pins are tied together (DRV_SYNC).

The LEDs are wired as a matrix across each driver's four outputs: the three anodes of an LED share one OUT pin, and its red, green and blue cathodes go to the other three. Each driver has four LED positions, three on the board and one on J4. [IO.md](IO.md) lists the OUT pin for every anode and cathode.

### Driver enable (JP1, JP2)

The LP5813 EN pins are pulled up to +3V3 (R10 for U4, R11 for U5), and two 3-pad solder jumpers connect them to one of the carrier's spare chip selects, so firmware can switch the drivers off:

| Jumper | Default | Centre pad | Side pads |
| ------ | ------- | ---------- | --------- |
| JP1 | All three pads bridged | EYESPI_TS_CS (carrier GP15) | Pad 3 to U4 EN ("A"), pad 1 to U5 EN ("B") |
| JP2 | Open | EYESPI_MEM_CS (carrier GP14) | Pad 3 to U4 EN ("A"), pad 1 to U5 EN ("B") |

By default, TS_CS enables both drivers: keep it high, or leave it as an input, and the pull-ups keep the drivers on. Driving it low switches them off. To move a driver to MEM_CS, cut JP1 between the centre pad and that driver's side, and bridge JP2 on the same side.

## I2C

The module uses the carrier's I2C0 (QWIIC_SDA on GP16, QWIIC_SCL on GP17), from J12 or J13. U2, U4 and U5 are on it, and J3 (a JST SH Qwiic connector: GND, +3V3, SDA, SCL) passes the bus on.

R13 and R15 (10K) pull SDA and SCL up to +3V3 on the module. The carrier has its own 10K pull-ups, so with the module connected the bus sees about 5K, which is still well within the I2C specification at 3.3 V. Devices on J3 need no more pull-ups.

None of the module's addresses (0x27, 0x50–0x57, 0x6C–0x6F) clash with the carrier's DS3231 RTC at 0x68.

## Other connectors

| Connector | Fitted | Pins |
| --------- | ------ | ---- |
| J3 | Yes | Qwiic: 1 GND, 2 +3V3, 3 SDA, 4 SCL |
| J4 | Yes (no LCSC number; see the manufacturing notes) | 2×6 2.54 mm box header (SMD) for two external buttons and two external RGB LEDs. Odd pins are driver B (U5), even pins are driver A (U4). Pins 1/3 BTN_EXT_1/BTN_EXT_2, 2/4 GND, 5/6 OUT0 (green cathode), 7/8 OUT1 (red cathode), 9/10 OUT2 (blue cathode), 11/12 OUT3 (common anode). |
| J5 | No (DNP) | Breakout: 1 +3V3, 2 GND, 3 EYESPI_GP1 (carrier GP18, silkscreen "GP1 (TX)"), 4 EYESPI_GP2 (carrier GP19, "GP2 (RX)") |
| J6 | No (DNP) | Breakout: 1 +3V3, 2 GND, 3 SCK, 4 MOSI, 5 MISO, 6 EYESPI_SD_CS (carrier GP13) |

## Solder jumpers

| Jumper | Default | Function |
| ------ | ------- | -------- |
| JP1 | Bridged (all three pads) | LP5813 enable from EYESPI_TS_CS. See [Driver enable](#driver-enable-jp1-jp2). |
| JP2 | Open | LP5813 enable from EYESPI_MEM_CS |

## Mechanical

* The board is 126.5 × 71.5 mm, 2 layers, 1.6 mm thick.
* **Display bracket:** the display is positioned by a 3D-printed bracket. [`GS3 TFT Shim.stl`](GS3%20TFT%20Shim.stl) is the file to print; `GS3 TFT Shim.step` is the same part, used as the 3D model of H5 (the bracket's outline, `DisplayShim`, on the bottom side), so it shows in KiCad's 3D view. The files keep "shim" in their names. See the manufacturing notes for printing.
* **Mounting holes** (unplated): H1–H4, H9 and H10 are 2.5 mm, H11–H14 are 3 mm, and H15–H17 are 4 mm.
* **Button stems:** a note next to the board outline gives the button stem used with the switches: 3 mm stem, square top 2.5 × 2.5 mm, 7 mm total height.
