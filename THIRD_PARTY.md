# Third-party content

Everything in this repository is licensed under the CERN Open Hardware Licence Version 2 – Permissive (CERN-OHL-P-2.0, see [LICENSE.txt](LICENSE.txt)), except the content listed below, which keeps its original licence.

## Library files included in this repository

| Files | Used by | Source | Licence | Changes made here |
| ----- | ------- | ------ | ------- | ----------------- |
| `BM04B-SRSS-TB_LF__SN_/` (`BM04B-SRSS-TB_LF__SN_.kicad_sym`, `JST_BM04B-SRSS-TB(LF)(SN).kicad_mod`) | J3 (JST BM04B-SRSS-TB Qwiic connector) | SnapEDA (now SnapMagic), <https://www.snapeda.com/> | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) with SnapEDA's Design Exception 1.0, subject to SnapMagic's [terms of service](https://www.snapeda.com/about/terms/) | None recorded. The download's STEP model and import guide aren't included. |
| `LP5813ADRRR/LP5813ADRRR.kicad_sym`, `LP5813ADRRR/IC_LP5813ADRRR.kicad_mod` | U4, U5 (TI LP5813) | SnapEDA | As above | None recorded. The import guide isn't included. |
| `FlexiblePushButton/SW_Flexible.kicad_mod` | SW1–SW12 (Omron B3FS-1050P, or E-Switch TL3301TF100QG) | Derived from `SW_SPST_Omron_B3FS-100xP` in the KiCad footprint library | [CC BY-SA 4.0 with the KiCad libraries exception](https://www.kicad.org/libraries/license/) | Pads lengthened outwards from 1.6 mm to 2.4 mm, and the courtyard widened to match. Same pad pitch. |

Each of these directories also contains a `LICENSE.md` with the same information. `LP5813ADRRR/DisplayShim.kicad_mod`, the outline of the display bracket (H5), shares a directory with the SnapEDA files but was made for this project, and is CERN-OHL-P-2.0 like the rest of the repository.

The Variegated Coffee logo library (`VariegatedCoffeeLogo.pretty/`) is copied from [variegated-coffee/kicad-footprints](https://github.com/variegated-coffee/kicad-footprints) (commit `1eca8d7`, the same copy the APEC carrier bundles). The EyeSPI connector symbol in `LM_GS3_HMI_Module.kicad_sym` is copied from the APEC carrier's `APEC_Carrier.kicad_sym`, so its default Footprint and LCSC fields still name the carrier's part; J12 overrides both. Both are CERN-OHL-P-2.0 like the rest of this repository.

## Content embedded in the design files

The schematic and PCB contain copies of symbols, footprints and 3D model references from these libraries. The library files themselves aren't included.

| Library | Used for | Licence |
| ------- | -------- | ------- |
| KiCad standard libraries | Most symbols and footprints | [CC BY-SA 4.0 with the KiCad libraries exception](https://www.kicad.org/libraries/license/): designs that use them, and files generated from them, aren't covered by the licence. |
| Alternate KiCad Library (AKL) by Dawid Cisło, <https://github.com/DawidCislo/Alternate-KiCad-Library> | R1 and D1 footprints | CC BY-SA 4.0 with the same exception as the KiCad libraries; no attribution is required in designs. |
| Ultra Librarian (Cadence), <https://www.ultralibrarian.com/> | U1 (TI LM3410, symbol and footprint), J2 footprint (Molex 52745-1097), J12 footprint (Molex 505110-1892) | Ultra Librarian's terms grant no right to redistribute the library files, so only the copies embedded in the schematic and PCB are included. Download the parts from Ultra Librarian if you need to edit them as library parts. |

## 3D models

The Molex 3D models for J2 and J12 aren't included, because their terms don't allow redistribution. The PCB looks for them in `3d-models/`; [3d-models/README.md](3d-models/README.md) lists each file and where to download it. Without them, those parts are missing from 3D views and renders, but the design is otherwise complete.

The KiCad standard 3D models are referenced through `${KICAD10_3DMODEL_DIR}`, except for the RGB LEDs (D8–D16), which still reference `${KICAD9_3DMODEL_DIR}/LED_SMD.3dshapes/LED_Avago_PLCC6_3x2.8mm.wrl`. KiCad 10 doesn't ship a model for that footprint, so the LEDs have no 3D model in KiCad 10.

`GS3 TFT Shim.step` and `GS3 TFT Shim.stl`, the display bracket, were made for this project and are CERN-OHL-P-2.0 like the rest of the repository.
