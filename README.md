# Shabir Ahmad — IFC Validation & Quantity Takeoff Portfolio

**BIM Automation Engineer**

[![Website](https://img.shields.io/badge/shabirbim.com-000000?style=flat&logo=google-chrome&logoColor=white)](https://shabirbim.com)
[![Email](https://img.shields.io/badge/engrshabir@shabirbim.com-EA4335?style=flat&logo=gmail&logoColor=white)](mailto:engrshabir@shabirbim.com)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/shabir-bim)

---

This repository is the source for [shabirbim.com](https://shabirbim.com) — a portfolio of IFC validation and structural quantity takeoff case studies. Seven IFC files processed, validated, and documented with full quantity traceability across four authoring tools and two IFC schemas.

The work demonstrates one thing: **IFC data is semantically unreliable without validation**. Errors are not missing data problems. They are detection and interpretation failures that propagate silently into BOQs, cost plans, and procurement decisions.

StructBOQ — the engine behind this work — processes IFC files from any of four authoring tools, extracts quantities at element level, assigns confidence scores, flags semantic anomalies, and generates PDF validation reports and Excel BOQs automatically. The engine is private. The outputs and methodology are fully documented here.

---

## Case Studies

Seven files. Each processed through the same validation pipeline — quantity extraction, source auditing, element-level confidence scoring, Bonsai/BlenderBIM cross-validation, and PDF + Excel report generation.

| # | File | Software | Schema | Concrete | Key Finding |
|---|------|----------|--------|----------|-------------|
| 01 | AC20-FZK-Haus | ArchiCAD | IFC4 | 131.67 m³ | Pset property `Slope = 1718.873` misread as beam depth → 37.65 m derived dimension blocked |
| 02 | SampleStructuralModel.ifc | Revit 2025 | IFC4 | 232.18 m³ | 17 steel pipe piles and 15 concrete pile caps — identical IFC class, zero material assignment, separated by name pattern only |
| 03 | IfcReinforcingBar.ifc | Allplan 2014 | IFC2X3 | 2.61 m³ | Initial steel: 204 kg. Correct: 404.83 kg. Deviation 98%. Data was present — extraction path was wrong |
| 04 | SampleBimModelWebinar.ifc | Revit 2025 | IFC4 | 2,290.91 m³ | 828.272 m³ composite slab excluded — insulation and concrete combined with no material fractions defined |
| 05 | SampleBuilding.ifc | Revit 2024 | IFC4 | 2,760.58 m³ | Class-only extraction returns 9 foundation elements and 56.88 m³. Correct: 90 elements and 361.62 m³ |
| 06 | Allplan TIEFGARAGE | Allplan 2017 | IFC2X3 | 62.703 m³ | 31.371 m³ of masonry, plasterboard, and insulation removed after reading `Gewerk` field in `IfcComplexProperty` — a field most parsers never reach |
| 07 | CPS-026/270 | Revit 2021 | IFC2X3 | 122.27 m³ | Real project. First delivery failed. Steel error 767% — 20,906 kg reported, 2,409.84 kg correct. 1,252 bars fully modeled and unread |

Full case studies with findings, screenshots, and downloadable PDF + Excel reports:
[**shabirbim.com/case-studies.html**](https://shabirbim.com/case-studies.html)

---

## Validation Methodology

Every concrete element validated against raw IFC data using Bonsai / BlenderBIM before any output is published. Three-stage process:

**1 — Extraction**
Quantity priority: IFC `BaseQuantities` → Pset properties → geometric fallback. Source recorded per element. No silent fallback.

**2 — Classification**
Multi-layer detection using `PredefinedType`, element naming conventions, material keywords, and authoring software-specific signals — `Gewerk` for Allplan, `BaseQuantities` absence for ArchiCAD library objects, `BASESLAB` for Revit footings. Software detected from IFC header at load time.

**3 — Cross-validation**
Every published concrete volume independently verified in Bonsai. Quantity delta documented per file. Discrepancies above floating-point tolerance investigated and resolved before publication.

---

## Cross-Software Coverage

| Software | Schema | Status |
|----------|--------|--------|
| Autodesk Revit 2021–2025 | IFC2X3, IFC4 | ✅ Validated — 4 files |
| ARCHICAD 22 | IFC4 | ✅ Validated — 1 file |
| Allplan 2014, 2017 | IFC2X3 | ✅ Validated — 2 files |
| Tekla Structures | IFC2X3, IFC4 | 🔲 Not yet tested |

---

## Selected Technical Findings

Real extraction failures caught across the seven files. Each one is documented with exact numbers in the case studies.

**Non-dimensional property as geometry input (File 01)**
`Pset_BeamCommon.Slope = 1718.873` on 42 timber rafters. Read naively as a dimensional input, this produces a beam depth of 37.65 m. Actual rafter depth: 0.16 m. The non-dimensional property blocklist — `Slope`, `FireRating`, `ThermalTransmittance`, `LoadBearing` — prevents this class of error permanently.

**Unit deviation without metadata flag (File 03)**
Allplan exports `CrossSectionArea` in cm². IFC schema defines the field in m². No flag in the file indicates the deviation. Read as m², a 10mm rebar weighs 6,162 kg/m. Correct: 0.616 kg/m. Error factor: 10,000×.

**GrossVolume corruption in IFC2X3 export (File 07)**
`BASESLAB` elements: `GrossVolume` 432.0 m³, `NetVolume` 0.432 m³. Factor of 1000. Spandrel beams: approximately 260× mismatch. No warning is issued. The corrupted value looks plausible. `NetVolume` used exclusively throughout.

**Multi-layer slab misread (File 07)**
Ground floor build-up: 100mm concrete, 50mm sand blinding, 300mm earth subgrade. IFC `NetVolume` includes all layers: 60.2249 m³. Structural concrete layer only: 13.383 m³. Cost impact at €120/m³: €5,621 from one slab. One unchecked assumption.

**Gewerk classification field (File 06)**
Stored in `IfcComplexProperty` Object Layer Attributes — not in standard Psets. Seven trade values distinguish structural concrete from masonry, plasterboard, insulation, waterproofing, stone finish. Without reading this field: 50% volume overstatement. With it: 62.703 m³ validated to 4 decimal places.

---

## Output Format

Every case study produces two downloadable outputs:

**Validation Report — PDF**
Project summary, file data confidence block, element-level confidence scoring, flagged elements with severity and issue reason, reinforcement schedule, model quality verdict.

**BOQ Report — Excel**
Element-level breakdown with name, material, flag status, data source, dimensions, volume, steel estimate, and cost. About tab with generation metadata. Non-concrete elements documented as excluded with reason.

> Cost figures in all reports use default rates (€120/m³ concrete, €1.2/kg steel). These are indicative only — not market rates.

---

## Tech Stack

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![IfcOpenShell](https://img.shields.io/badge/IfcOpenShell-grey?style=flat)
![Bonsai / BlenderBIM](https://img.shields.io/badge/Bonsai%20%2F%20BlenderBIM-E87D0D?style=flat&logo=blender&logoColor=white)
![openpyxl](https://img.shields.io/badge/openpyxl-217346?style=flat)
![ReportLab](https://img.shields.io/badge/ReportLab-CC0000?style=flat)

---

## Contact

Send an IFC file for a free validation report — delivered within 24 hours.

[**shabirbim.com/submit-ifc.html**](https://shabirbim.com/submit-ifc.html)
[engrshabir@shabirbim.com](mailto:engrshabir@shabirbim.com)

Structural engineering consultancies, BIM coordinators, and technical directors working with IFC workflows are welcome to get in touch directly.

---

*Shabir Ahmad — admitted to Bauhaus University Weimar, Digital Engineering track, 2026.*
