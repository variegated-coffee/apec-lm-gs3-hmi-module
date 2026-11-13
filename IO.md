# LM GS3 HMI Module IO assignments

What firmware talks to on the module, revision R1A. Generated from the R1A schematic netlist; net names match the schematic. Carrier GPIOs are from the APEC carrier's `IO.md` (revision EV3).

## Connector signals (J12, J13)

J12 (EyeSPI FFC) and J13 (2×10 header) carry the same signals in parallel. Where the module renames a signal, the module net is the one in the schematic and PCB.

| EyeSPI signal | J12 pin | J13 pin | Carrier GPIO | Module net | Goes to |
| ------------- | ------- | ------- | ------------ | ---------- | ------- |
| Vcc | 1 | 17 | — | DISP_VDD | +3V3 for the whole module |
| BACKLIGHT | 2 | 18 | GP1 (PWM0B) | EYESPI_BL | U1 DIM (backlight on/off and PWM dimming). Pulled up by R2. |
| GND | 3 | 15 | — | DISP_GND | |
| SCK | 4 | 16 | GP2 | DISP_SCL | J2 pin 2, J6 pin 3 |
| MOSI | 5 | 13 | GP3 | DISP_SDA | J2 pin 1, J6 pin 4 |
| MISO | 6 | 14 | GP4 | EYESPI_MISO | J6 pin 5 only |
| DC | 7 | 11 | GP6 | DISP_RS | J2 pin 3 |
| RESET | 8 | 12 | GP7 | DISP_RSTB | J2 pin 4 |
| DISP_CS | 9 | 9 | GP12 | DISP_CS | J2 pin 5 |
| SD_CS | 10 | 10 | GP13 | EYESPI_SD_CS | J6 pin 6 only |
| MEM_CS | 11 | 7 | GP14 | EYESPI_MEM_CS | JP2 (LP5813 enable, optional) |
| TS_CS | 12 | 8 | GP15 | EYESPI_TS_CS | JP1 (LP5813 enable, default) |
| SCL | 13 | 5 | GP17 | QWIIC_SCL | I2C: U2, U4, U5, J3. Pulled up by R15. |
| SDA | 14 | 6 | GP16 | QWIIC_SDA | I2C. Pulled up by R13. |
| INT | 15 | 3 | GP21 | EYESPI_INT | U2 (MCP23008) INT |
| BUSY/TE | 16 | 4 | GP22 | DISP_TE | From J2 pin 6 (display tearing effect) |
| GPIO1 | 17 | 1 | GP18 | EYESPI_GP1 | J5 pin 3 only |
| GPIO2 | 18 | 2 | GP19 | EYESPI_GP2 | J5 pin 4 only |
| — | — | 19 | — | +12V | Not connected (VIN on the carrier) |
| — | — | 20 | — | +5V | Not connected |

## I2C devices

All on the carrier's I2C0.

| Address | Device | Notes |
| ------- | ------ | ----- |
| 0x27 | U2, MCP23008 I/O expander | A2–A0 tied to +3V3 |
| 0x50–0x53 | U4, LP5813A LED driver ("driver A") | Buttons 1–3 |
| 0x54–0x57 | U5, LP5813B LED driver ("driver B") | Buttons 4–6 |
| 0x6C–0x6F | U4 and U5 | LP5813 broadcast addresses |

## MCP23008 (U2)

Every input is active low: a press pulls the pin to ground against a 10K pull-up on the board, so leave the internal pull-ups (GPPU) off. INT goes to the carrier's GP21.

| Pin | Net | Function |
| --- | --- | -------- |
| GP0 | BTN_6 | Button 6, "Tea" (SW6, SW12) |
| GP1 | BTN_5 | Button 5, "Fn" (SW5, SW11) |
| GP2 | BTN_4 | Button 4, "Ret" (SW4, SW10) |
| GP3 | BTN_3 | Button 3, "Sel" (SW3, SW9) |
| GP4 | BTN_2 | Button 2, "Incr" (SW2, SW8) |
| GP5 | BTN_1 | Button 1, "Decr" (SW1, SW7) |
| GP6 | BTN_EXT_2 | External button 2, J4 pin 3 |
| GP7 | BTN_EXT_1 | External button 1, J4 pin 1 |
| INT | EYESPI_INT | Interrupt to the carrier (GP21) |

## LP5813 outputs (U4, U5)

Each LP5813's four outputs drive four RGB LEDs as a matrix. Both drivers are wired the same way:

| LED position | Driver A (U4) | Driver B (U5) | Anodes | Red cathode | Green cathode | Blue cathode |
| ------------ | ------------- | ------------- | ------ | ----------- | ------------- | ------------ |
| 1 | D8 (button 1) | D14 (button 4) | OUT0 | OUT2 | OUT1 | OUT3 |
| 2 | D10 (button 2) | D15 (button 5) | OUT1 | OUT3 | OUT2 | OUT0 |
| 3 | D12 (button 3) | D16 (button 6) | OUT2 | OUT0 | OUT3 | OUT1 |
| 4 | External, J4 even pins | External, J4 odd pins | OUT3 | OUT1 | OUT0 | OUT2 |

On J4, OUT0 is on pins 5/6, OUT1 on 7/8, OUT2 on 9/10 and OUT3 on 11/12 (odd pins driver B, even pins driver A).

Each output line is shared by several LEDs, so the schematic gives it more than one name, and KiCad picks one of them for the netlist:

| Output | Driver A net | Driver B net |
| ------ | ------------ | ------------ |
| OUT0 | DRVA_OUT0 (netlist: BTN1_LED_B_A) | DRVB_OUT0 (netlist: BTN4_LED_B_A) |
| OUT1 | DRVA_OUT1 (netlist: BTN1_LED_G_K) | DRVB_OUT1 (netlist: BTN4_LED_G_K) |
| OUT2 | DRVA_OUT2 (netlist: BTN1_LED_R_K) | DRVB_OUT2 (netlist: BTN4_LED_R_K) |
| OUT3 | DRVA_OUT3 (netlist: BTN1_LED_B_K) | DRVB_OUT3 (netlist: BTN4_LED_B_K) |

The EN pins are on EYESPI_TS_CS by default (JP1), with pull-ups R10 (U4) and R11 (U5); see [ENGINEERING-NOTES.md](ENGINEERING-NOTES.md#driver-enable-jp1-jp2).
