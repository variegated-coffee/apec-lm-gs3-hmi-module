# Changelog

## EV1 to R1A

* Changed the button switches from the Omron B3FS-1002P (LCSC C87038) to the Omron B3FS-1050P (LCSC C231326), on a new footprint, `SW_Flexible`: the KiCad B3FS footprint with longer pads, which also takes the cheaper E-Switch TL3301TF100QG
* Changed the MCP23008 (U2) I2C address from 0x20 to 0x27, by tying A0–A2 to +3V3 instead of GND
* Added two external button inputs on the MCP23008's previously unused GP6 and GP7 (BTN_EXT_2, BTN_EXT_1), with 10K pull-ups R12 and R9
* Added J4, a 2×6 header for the two external buttons and for the fourth LED position of each LP5813 (one external RGB LED per driver)
* The LP5813 EN pins, previously tied to +3V3 through R10 and R11, can now be switched by the carrier: by EYESPI_TS_CS through JP1 (bridged by default), or by EYESPI_MEM_CS through JP2 (open by default)
* BOM changes: the RGB LEDs (D8–D16, LCSC C19531997) and J13 (LCSC C22387750) are now included for assembly, and the J5 and J6 breakout headers are now DNP