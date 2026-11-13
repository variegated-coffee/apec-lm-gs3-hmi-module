# Changelog

## EV1 to R1A

* Changed the button switches from the Omron B3FS-1002P (LCSC C87038) to the Omron B3FS-1050P (LCSC C231326), on a new footprint, `SW_Flexible`: the KiCad B3FS footprint with longer pads, which also takes the cheaper E-Switch TL3301TF100QG
* Changed the MCP23008 (U2) I2C address from 0x20 to 0x27, by tying A0–A2 to +3V3 instead of GND
* Added two external button inputs on the MCP23008's previously unused GP6 and GP7 (BTN_EXT_2, BTN_EXT_1), with 10K pull-ups R12 and R9
* Added J4, a 2×6 header for the two external buttons and for the fourth LED position of each LP5813 (one external RGB LED per driver)
* The LP5813 EN pins, previously tied to +3V3 through R10 and R11, can now be switched by the carrier: by EYESPI_TS_CS through JP1 (bridged by default), or by EYESPI_MEM_CS through JP2 (open by default)
* BOM changes: the RGB LEDs (D8–D16, LCSC C19531997) and J13 (LCSC C22387750) are now included for assembly, and the J5 and J6 breakout headers are now DNP
* Added silkscreen for the board name and revision, J4's pins and the LED driver enable jumpers, updated the MCP23008 address, and hid the reference designators that overlapped other silkscreen (SW1–SW12, R9, R12, JP1, JP2, J4)
* Rerouting for the above
* Repository cleanup for the open source release (no electrical changes)
  * The Variegated Coffee logo library is now bundled in the repository, instead of being loaded from a path outside it
  * The EyeSPI connector symbol (J12) moved to a new project library, `LM_GS3_HMI_Module.kicad_sym`, copied from the APEC carrier
  * Removed unused libraries, and the library files of parts whose licences don't allow redistribution (they stay embedded in the schematic and PCB)
  * Vendor 3D models are no longer in the repository; `3d-models/README.md` lists where to get them
  * Added README, licence, third-party notice, and engineering, IO and manufacturing notes

### Firmware changes R1A needs

* Address the MCP23008 at 0x27 instead of 0x20.
* Read GP6 and GP7 of the MCP23008 as external buttons 2 and 1 (active low, like the others).
* EYESPI_TS_CS (carrier GP15) now enables both LED drivers. Keep it high or as an input; driving it low switches the LEDs off.
