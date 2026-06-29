# PITOT

Field Instrument #050. A testing, adjusting, and balancing (TAB) workbook for HVAC field work.

Single HTML file, no build step, no server. Opens directly from disk and persists to localStorage plus IndexedDB (for photos).

## Status

**v0.3.3 (Phase 3 second quick-win slice).** More small ergonomics fixes that come up the first time a field tech actually uses the tool:

- **Duplicate button on equipment grid cards.** Previously the duplicate action only appeared on the equipment detail view. Now every card in the grid has a Duplicate button in its footer (low-contrast, brightens on hover) for one-tap cloning without going into the detail page first.
- **Bulk add equipment** (new `+ Bulk add` button on the equipment toolbar). Lets the technician generate a sequence of equipment records in one shot: pick the type, set a tag prefix, separator, start number, count, optional zero-padding width, optional common location / serves / manufacturer / model, optional associated AHU (for terminals). The modal previews the first and last tags before creating, and warns on tag collisions with existing equipment (non-blocking). Capped at 200 per batch.
- **Last-active-tab restored on reload.** Open the punch list, close the browser, come back to PITOT, you land on the punch list again. The persisted tab name is sanity-checked against the valid set so a corrupt value can't lock the app into a non-existent view.
- **AHU detail shows associated terminal count.** Terminal detail shows which AHU it serves. Quick visual confirmation that the air-balance report's plumbing is wired up.
- **Tag uniqueness warning** when saving equipment. Detects collisions on whitespace-trimmed tags, reports the collision's type and location, and asks for confirmation. Non-blocking: if the duplicate is intentional, the technician can proceed.
- **Print button switches to the Report tab first.** No more printing the equipment grid by accident.

**v0.3.2** added the VAV traverse target picker, the `coilHeatBalance` defensive guards, the IndexedDB photo cleanup on delete, the equipment Duplicate button on the detail bar, and the traverse modal shape-switch robustness. **v0.3.1** added per-template equipment forms with section grouping. **v0.3.0** added the body-specific report style framing. All preserved here.

Schema is `pitot/workbook@3`. Older workbooks (`@1`, `@2`) are forward-migrated on load. New fields added in v0.3.1 do not require a schema bump because they live in `equipment.design` which is a free-form object; missing fields render as blank.

**These remain STYLE TEMPLATES, not official AABC/NEBB/TABB forms.** PITOT is independent software with no affiliation to any TAB certifying body. A certified submittal must use the certifying body's published forms and bear the signature of a credentialed supervisor.

## What this is

PITOT is the field workbook a TAB technician fills out, equipment by equipment, while balancing a commercial HVAC system. It captures design intent from the mechanical drawings, records initial / adjusted / final readings, compares the readings against design using configurable tolerance bands, and produces a printable TAB report.

It is named for the pitot tube, the iconic field instrument for HVAC duct traverse measurements (Henri Pitot, 1732).

## What this is not

PITOT is not an accredited TAB certification tool. The report style templates are LAYOUT TEMPLATES that follow the section ordering, terminology, and certification-statement conventions commonly seen in AABC, NEBB, and TABB submittals. They are NOT the certifying bodies' published forms, and PITOT is not affiliated with AABC, NEBB, TABB, SMACNA, or SMART.

PITOT does not validate that the field instruments used are themselves in calibration.

PITOT is not a controls verification tool. TAB verifies steady-state performance. Sequence-of-operation testing belongs in a commissioning record.

## Install and use

1. Download `pitot.html`.
2. Double-click to open in any modern browser, or serve it from any static host.
3. State persists to localStorage and IndexedDB.

No install, no account, no telemetry. Google Fonts is the only external resource.

## Equipment field model (v0.3.1)

Each equipment type's fields are now tagged with a `section` and a `type`. The pass/fail engine continues to operate on `tolerance` and `critical` flags; new section / type metadata only affects layout.

### AHU / RTU

- **Fan data**: Total airflow (CFM, critical), External static pressure, Fan RPM
- **Motor data**: Motor horsepower (nameplate), Motor voltage (nameplate), Motor phase (1 / 3, nameplate), Motor FLA (nameplate), Motor amps (measured per phase)
- **Drive data**: Fan sheave PD, Motor sheave PD, Belt size, Belt quantity (all nameplate / observed)
- **Filter data**: Filter type (e.g. MERV 13), Filter size, Filter quantity (all observed)
- **Outside air**: OA CFM (measured, air tolerance), OA percent
- **Air temperatures**: Mixed, Supply, Return (all temp tolerance)

### VAV box

- **Airflow data**: Minimum CFM (critical), Maximum CFM (critical), Inlet static pressure
- **Reheat data**: Reheat type (none / electric / hot water), Reheat capacity (BTU/h), Reheat water flow (GPM, water tolerance), Discharge air temp
- **Identification**: Box model, Controller address

### Pump

- **Pump performance**: Flow rate (GPM, critical), Head (ft wg)
- **Motor data**: Motor HP, Motor voltage, Motor phase, Motor FLA (nameplate), Motor amps (measured)
- **Pressure data**: Suction, Discharge

### Terminal (diffuser / grille)

- **Airflow data**: CFM (critical), Face velocity
- **Identification**: Neck size, K-factor, Area served (room number), Drawing grid reference

### Coil

- **Water side**: GPM (critical), Entering water temp, Leaving water temp
- **Air side**: Air flow across coil (CFM, air tolerance), Entering air temp, Leaving air temp
- **Capacity**: Coil type (cooling / heating / cooling+dehum), Heat transfer (BTU/h)

Nameplate / identifying fields (motor voltage, phase, drive data, filter data, etc.) are marked `excludeFromTest: true`. They are entered once on the equipment record and do NOT appear in the per-test reading form, because they don't change between TAB phases.

## Per-template section ordering

Each body template defines a section order per equipment type. Where the template has no preference (or uses `null`), PITOT renders sections in their natural order from the field registry.

**AABC** uses: fan, motor, drive, filter, oa, temp (for AHUs). Labels: "Fan performance", "Motor data", "Drive data", "Filter data", "Outside air data", "Temperature data".

**NEBB** uses the same ordering but with different labels: "Fan performance", "Motor electrical", "Drive components", "Filter section", "Outside air", "Air temperatures".

**TABB** uses the natural section labels.

**Draft** uses the natural section labels with no reordering.

The Report tab's equipment block prints each section as a labeled sub-table. Sections with no design or reading data are suppressed.

## Workflow

1. **Project tab**: project, personnel, schedule, reporting (style, firm, supervisor, dates, scope, architect, engineer), notes.
2. **Equipment tab**: add equipment. The Add/Edit modal now groups design fields by section (Fan data, Motor data, etc.) for the chosen type. Terminals can be assigned an associated AHU.
3. **Test entry**: opens a phase-tagged test record. Reading fields are also section-grouped and exclude nameplate / observed fields. Traverse / Hood / Photo buttons attach the relevant Phase 2 sub-records.
4. **Status auto-classifies**: pending, in-progress, balanced, out-of-spec, or punch.
5. **Punch list**: deficiencies blocking balance, with owner and priority.
6. **Pump equipment**: separate pump curve and system curve panel computes the operating point and renders an SVG plot.
7. **Coil tests**: automatic heat-balance display when water and air readings are both present.
8. **Report**: template-driven cover, scope of work, equipment-by-type with section sub-tables, air-balance verification, punch list, Statement of Certification, signature block. Print directly.
9. **Export**: JSON workbook (no photo binaries).

## Data model

Project block adds the v0.3.0 reporting fields. Equipment design is a free-form object; in v0.3.1 the well-known keys grew to match the field registry above. Old workbooks with fewer keys still work; those fields simply render as blank.

```
{
  schema: 'pitot/workbook@3',
  version: '0.3.1',
  project: { name, address, owner, mechanicalContractor, tabContractor,
             technician, technicianCertification, projectNumber,
             startDate, endDate, weather, notes,
             reportStyle, firmName, firmAddress, firmCertNumber,
             supervisorName, supervisorCertNumber,
             reportDate, reportRevision,
             projectArchitect, projectEngineer, scopeOfWork },
  equipment: [ {
    id, tag, type, location, serves, manufacturer, model, serial,
    design: { ... per-type fields per v0.3.1 ... },
    tests: [ { id, phase, date, technician, readings, instrumentUsed,
               instrumentCalDate, notes, photos, traverse?, hood? } ],
    tolerance: { airPercent, waterPercent, tempDegF },
    status, notes, photos, associatedAhu,
    pumpCurve?, systemCurve?
  } ],
  punchList: [ ... ]
}
```

Photos are in IndexedDB (`pitot.photos`) and not part of the JSON export.

## Tolerance defaults

- Air: 10 percent
- Water: 10 percent
- Temperature: 2 degF (absolute)

## Conventions followed

- Single HTML file, all CSS and JS inline, no build step, no npm
- Opens from `file://`, no backend, no server, no telemetry
- localStorage + IndexedDB only
- Google Fonts (Fraunces, Oswald, Inter, JetBrains Mono) only
- Dark mode default; print flips to light
- No em dashes anywhere
- Vintage scientific instrument aesthetic: graphite, brass, sky-blue, signal red, amber
- Mobile-responsive (768 / 540 / 380 breakpoints) with 40 px touch targets and 16 px input font on coarse pointers

## Validation performed

Static checks on the v0.3.3 release artifact:

- Line count: 5849 lines
- Em dash count: 0 across literal U+2014, `&mdash;`, `&#8212;`, and `\u2014` escapes
- CSS braces balanced: 367 / 367
- JS syntax: `node --check` passes
- HTML tag structure: matched, no mismatches, no unclosed

Logic tests (106 cases, all passed):

- Phase 1: pass/fail engine, status classification (16 cases)
- Phase 2: traverse, hood, coil heat balance, pump operating point, air balance, v1 to v2 migration (33 cases)
- Phase 3.0: report template registry, reading-label flip, certification statements, disclaimers, firm-header policy, draft fallback, v2 to v3 migration, full v1 to v3 chain (15 cases)
- Phase 3.1: section grouping, field type and excludeFromTest metadata, per-template ordering, label overrides, gracefully skipping unknown sections (19 cases)
- Phase 3.2: coilHeatBalance defensive guards against null inputs and missing design, duplicate-equipment data shape (10 cases)
- Phase 3.3: bulk tag generation across pad widths and prefixes, lastActiveTab fallback and invalid-value guarding, tag uniqueness detection (13 cases)

Not validated by the automated tests (browser-only):

- Visual rendering of section sub-tables in the report
- Print preview with section headings across page breaks
- Equipment-modal section grouping with the new text/select fields populating correctly
- Re-entering a test for an AHU that has all six sections filled (no overflow / layout issues)
- Photo capture and IndexedDB persistence

## Known issues / known limitations

- **Asymmetric tolerance not supported.** The pass/fail engine treats all bands as symmetric (+/-). Some specs use "no more than 110 percent of design but at least 95 percent." That's a Phase 4+ change.
- **Heat balance does not include latent loads.** Cooling coils with significant dehumidification will appear out of balance because the water side captures total capacity while the sensible-only air calc does not.
- **No multi-AHU air balance roll-up.** A terminal can only be associated with one AHU. Multi-zone systems and dual-duct setups need a different schema.

## Deferred to later in Phase 3

- Per-equipment-type explicit report sub-forms (e.g. NEBB Form 1 layouts at the field-level detail). Currently all bodies share the same section-grouped table layout. Body-specific form templates are a much bigger lift and require careful study of each body's published form layouts (which would also require licensed access to those forms).
- Multi-theme system matching the WATTMETER family
- PWA package for offline field use
- Photo OCR for nameplate auto-capture
- Multi-AHU air-balance roll-up

## Domain glossary

- **TAB**: Testing, Adjusting, and Balancing
- **AABC**: Associated Air Balance Council
- **NEBB**: National Environmental Balancing Bureau
- **TABB**: Testing, Adjusting and Balancing Bureau (SMACNA/SMART)
- **TBE**: Test and Balance Engineer (AABC supervisor credential)
- **CP**: Certified Professional (NEBB supervisor credential)
- **FLA**: Full Load Amps (motor nameplate)
- **RLA**: Running Load Amps (measured)
- **PD**: Pitch Diameter (sheave)
- **MERV**: Minimum Efficiency Reporting Value (filter rating)
- **CFM**: cubic feet per minute, volumetric airflow
- **GPM**: gallons per minute, volumetric water flow
- **ESP / TESP**: external / total external static pressure across an AHU
- **VP**: velocity pressure, in. wg
- **K-factor**: terminal or hood calibration factor
- **Statement of Certification**: the formal attestation paragraph in a TAB report

## License

GPL-3.0

Copyright 2026 M.B. Parks, Green Shoe Garage, Cumberland, Maryland.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see the GNU General Public License version 3 in plain text from the Free Software Foundation.

## Author

M.B. Parks (N1HNP), Green Shoe Garage, Cumberland, Maryland.

Part of the Field Instruments catalog. Make. Hack. Learn. Share. Repeat.
