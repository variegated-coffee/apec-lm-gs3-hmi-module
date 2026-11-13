# Vendor 3D models

The PCB references these STEP models as `${KIPRJMOD}/3d-models/<file>`. Their vendors' terms don't allow redistribution, so they aren't in the repository. The design is complete without them; they're only needed for 3D views, renders and enclosure work.

To get full renders, download each model from its manufacturer (or the service named) and save it here under the file name in the first column. Rename the download if needed. Everything in this directory except this file is ignored by git.

| File | Part | Used by | Where to get it |
| ---- | ---- | ------- | --------------- |
| `0527451097.stp` | Molex 52745-1097, 10-circuit FFC connector | J2 (display) | Molex product page, or Ultra Librarian (with the part's download) |
| `5051101892.stp` | Molex 505110-1892, 18-circuit FFC connector | J12 (EyeSPI) | Molex product page, or Ultra Librarian (with the part's download) |

U1, U4, U5 and J3 have no 3D model. The RGB LEDs (D8–D16) reference `${KICAD9_3DMODEL_DIR}/LED_SMD.3dshapes/LED_Avago_PLCC6_3x2.8mm.wrl`, which KiCad 10 doesn't ship, so they don't show either. The display itself isn't modelled; the display bracket is, from `GS3 TFT Shim.step` in the repository root.
