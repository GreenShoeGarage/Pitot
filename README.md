# PITOT

Field Instrument #050. A testing, adjusting, and balancing (TAB) workbook for HVAC field work.

Single HTML file, no build step, no server. Opens directly from disk and persists to localStorage plus IndexedDB (for photos).

## Status

**v0.2.0 (Phase 2).** Adds duct traverse calculator, flow-hood reading helper, pump operating point with system-curve plot, coil heat balance, air-balance verification across associated terminals, and photo attachments. Bigger touch targets and quick-entry friendly inputs on mobile.

Schema bumped to `pitot/workbook@2`. Workbooks saved under v0.1.0 (`pitot/workbook@1`) are forward-migrated automatically on load. The Phase 1 surface is preserved.

## What this is

PITOT is the field workbook a TAB technician fills out, equipment by equipment, while balancing a commercial HVAC system. It captures design intent from the mechanical drawings, records initial / adjusted / final readings, compares the readings against design using configurable tolerance bands, and produces a printable TAB report.

It is named for the pitot tube, the iconic field instrument for HVAC duct traverse measurements (Henri Pitot, 1732). Phase 2 makes good on that name by adding the traverse calculator the instrument is built around.

## What this is not

PITOT is not an accredited TAB certification tool. Formal AABC, NEBB, or TABB submissions must be performed under a certified TAB supervisor and follow the certifying body's prescribed report format. PITOT can serve as the workspace for drafting that report, but the certifying authority's branding, layout, and signature blocks must be added separately.

PITOT does not validate that the field instruments used are themselves in calibration. The technician enters the calibration date for each test; that field is documentation, not verification.

PITOT is not a controls verification tool. TAB verifies steady-state performance. Sequence-of-operation testing belongs in a commissioning record.

The pass/fail engine uses simple percentage-band tolerance (air and water) and an absolute degF band (temperature). Asymmetric tolerances, weighted criteria, and multi-variable acceptance are deferred.

## Install and use

1. Download `pitot.html`.
2. Double-click to open in any modern browser, or serve it from any static host.
3. State persists to localStorage and IndexedDB, both scoped to the page origin. Different hosts have different stores.

There is no install, no account, no telemetry. Network access is used only to load Google Fonts. JSON workbook export does not include photo binaries; photos are stored locally only.

## Workflow

1. **Project tab.** Enter project name, address, contractors, technician, certification, dates, outdoor conditions.
2. **Equipment tab.** Add equipment for each item on the mechanical drawings. Pick a type (AHU, VAV, pump, terminal, coil), enter the drawing tag, manufacturer, location, and design values from the schedule. Terminals can be assigned an associated AHU for air-balance verification.
3. **Test entry.** From an equipment card, add a test record. Mark the phase (initial, adjusted, or final), enter readings against each design field, record the instrument used and its calibration date. From a test record:
   - **Traverse** opens the duct traverse calculator (AHU and VAV)
   - **Hood** opens the flow-hood reading helper (terminals)
   - **Photos** attaches nameplates, panel access shots, or anything else worth keeping
4. **Status auto-classifies** based on what tests exist and whether the final-phase readings fall within tolerance:
   - **pending**: no tests recorded
   - **in-progress**: tests recorded but no final phase, or final phase missing readings on critical fields
   - **balanced**: final phase, all critical readings within tolerance
   - **out-of-spec**: final phase, one or more critical readings outside tolerance
   - **punch**: manually flagged for the punch list
5. **Punch list.** For items that cannot be balanced due to upstream deficiencies (missing dampers, undersized ductwork, controls not functioning), file a punch item with an owner.
6. **Pump equipment** has a pump-curve panel showing the manufacturer curve, the system curve, and the computed operating point. Click "Set up pump curve" on the equipment detail.
7. **Coil tests** automatically display a heat-balance sub-panel comparing water-side and air-sensible BTU/h when both sets of readings are present.
8. **Report.** The Report tab produces a printable summary: project header, executive summary, equipment grouped by type with readings, air-balance verification, punch list, signature block. Use the browser's print dialog. Letter-size paper, half-inch to six-tenths-inch margins.
9. **Export.** JSON export from the project tab gives you a portable copy of the workbook. Import is a full replace, with a confirmation prompt. Photos are not included in the export.

## Phase 2 features

### Duct traverse calculator

Captures velocity-pressure readings across a grid and converts them to a total CFM. Supports rectangular ducts (with width, height, lines, and points per line) and round ducts (with diameter, number of diameters, and points per diameter). Position fractions follow either:

- **Log-Tchebycheff** (default): the ASHRAE-tabulated optimal positions
- **Equal-area**: center-of-area positions for n equal divisions

Optional air-temperature and barometric-pressure inputs apply a density correction relative to standard air (0.075 lb/ft^3, 70 degF, 29.92 in. Hg). The velocity formula is V = 1096.7 sqrt(VP / rho), which reduces to V = 4005 sqrt(VP) at standard conditions.

The computed total CFM is written back to the test's airflow reading automatically.

### Flow-hood reading helper

Averages multiple direct CFM readings from a balometer or capture hood, applies the K-factor and an optional back-pressure compensation multiplier, and writes the corrected total to the test's airflow reading.

### Pump operating point and system-curve plot

Stores manufacturer pump curve points (flow, head) and the system static head and design operating point on the equipment itself. Fits the system curve as H = static + k*Q^2 with k inferred from the design point, then walks pump-curve segments looking for the intersection by linear interpolation.

The plot is an inline SVG: pump curve solid, system curve dashed, design point as a brass diamond, operating point as an amber dot.

### Coil heat balance

For every coil test with both water and air readings, displays:

- Water-side BTU/h: GPM x 500 x dT
- Air-sensible BTU/h: CFM x 1.08 x dT
- Design BTU/h (if entered)
- Percent imbalance between water and air

Imbalance under 10 percent is flagged pass; over 10 percent is flagged fail. Latent loads are not modeled in Phase 2.

### Air-balance verification

Terminals can be tagged with an associated AHU. The report computes, for each AHU, the sum of associated terminal final-phase CFMs and compares against the AHU's final-phase total. Imbalance under 10 percent is flagged pass; over 10 percent is flagged fail.

### Photo attachments

Photos attach to equipment, to specific tests, or to both. They live in IndexedDB under the database name `pitot.photos`, separate from the workbook JSON in localStorage. Thumbnails render inline; full-quality blobs stay local. JSON export does not include binary photo data; only the metadata stubs travel with the workbook.

## Data model

State shape persisted to localStorage under the key `pitot.workbook`:

```
{
  schema: 'pitot/workbook@2',
  version: '0.2.0',
  project: { name, address, owner, mechanicalContractor, tabContractor,
             technician, technicianCertification, projectNumber,
             startDate, endDate, weather, notes },
  equipment: [ {
    id, tag, type, location, serves, manufacturer, model, serial,
    design: { type-appropriate fields },
    tests:  [ { id, phase, date, technician, readings, instrumentUsed,
                instrumentCalDate, notes, photos: [],
                traverse: {...}?, hood: {...}? } ],
    tolerance: { airPercent, waterPercent, tempDegF },
    status, notes, photos: [],
    associatedAhu,
    pumpCurve: [ {flow, head} ]?, systemCurve: {staticHead, designFlow, designHead}?
  } ],
  punchList: [ { id, equipmentId, issue, owner, priority,
                 dateRaised, dateResolved, resolution, status } ]
}
```

Photo binaries are stored separately in IndexedDB under the database name `pitot.photos` (object store `photos`). Each record carries `{ id, equipmentId, testId, caption, filename, mimeType, size, blob, added }`.

### Equipment types

- **AHU / RTU**: CFM (critical), external static pressure, motor HP, motor amps, fan RPM, mixed / supply / return air temperatures
- **VAV box**: minimum CFM (critical), maximum CFM (critical), reheat BTU/h, supply air temperature, inlet static
- **Pump**: GPM (critical), head feet, motor HP, motor amps, suction PSI, discharge PSI
- **Terminal (diffuser / grille)**: CFM (critical), neck size, face velocity, K-factor
- **Coil**: GPM (critical), entering / leaving water temperatures, entering / leaving air temperatures, BTU/h

### Schema migration

`pitot/workbook@1` workbooks are forward-migrated to `@2` on load. The migration adds `photos: []` on equipment and on each test, adds `associatedAhu: ''` on equipment, and adds `pumpCurve: []` and `systemCurve: {...}` on pumps. No existing fields are altered. Workbooks with a schema string PITOT does not recognize are rejected with a console warning, leaving the user with a fresh workbook.

## Tolerance defaults

- Air: 10 percent
- Water: 10 percent
- Temperature: 2 degF (absolute band, not percentage)

Tolerance is per-equipment; edit the values on any equipment card to adjust.

## Conventions followed

- Single HTML file, all CSS and JS inline, no build step, no npm
- Opens from `file://`, no backend, no server, no telemetry
- State persists to localStorage and IndexedDB only
- Google Fonts (Fraunces, Oswald, Inter, JetBrains Mono) are the only external resources
- Dark mode default; light mode triggered automatically by print
- No em dashes anywhere in code, comments, or prose
- Vintage scientific instrument aesthetic: graphite background, brass accents, cool sky-blue for balanced, signal red for out-of-spec, amber for pending and in-progress
- Mobile-responsive at 768 / 540 / 380 breakpoints
- Mobile and coarse-pointer devices get larger touch targets (40 px minimum) and 16 px input font to prevent iOS auto-zoom

## Validation performed

Static checks on the v0.2.0 release artifact:

- Line count: 4631 lines
- Em dash count: 0 (literal U+2014, `&mdash;`, `&#8212;`, and `\u2014` escapes all zero)
- CSS braces balanced: 325 opening, 325 closing
- JS syntax: `node --check` passes on the extracted script
- HTML tag structure: matched, stack empty at EOF

Logic tests (49 cases, all passed):

- `passFailField` across in-band, at-the-edge, just-over, and far-out cases for air, water, and temperature buckets, plus null handling
- `classifyStatus` across all five status states, including the punch override, missing-critical edge, and VAV multi-critical case
- `vpToVelocity` at standard density and edge cases
- `airDensity` at standard and altitude
- `ductArea` rectangular and round
- `equalAreaPositions` and `logTchebycheffRectangular` against the standard tables
- `traverseCFM` arithmetic-mean averaging, including with invalid values mixed in
- `hoodCFM` raw average, K and BP application, empty inputs
- `coilWaterBtuh`, `coilAirSensibleBtuh`, and full `coilHeatBalance` including the percent-imbalance computation
- `pumpOperatingPoint` linear-interpolation intersection across a known curve, plus not-enough-points and invalid-system guards
- `airBalanceReport` returns an array (data-dependent, exercised structurally)
- Schema migration v1 to v2 (schema bumps, equipment preserved, Phase 2 fields initialized) and rejection of unknown schema versions

Not validated through automated tests, and requires browser interaction to verify:

- Visual rendering of the new sub-panels, photo strips, and pump plot
- Photo capture and IndexedDB persistence (no native indexedDB in headless harness)
- Click and tap handlers on traverse grid cells, hood readings list, pump curve editor rows
- Print preview of the air-balance section and pump plot
- localStorage migration across a real page reload from a v0.1.0 workbook

These should be exercised on a real device before fielding on a job.

## Deferred to Phase 3

- Multi-theme system matching the WATTMETER family
- Full TAB report as a separate printable HTML document
- Body-specific worksheet formats (AABC, NEBB, TABB conventions)
- PWA package for offline field use
- BACnet object name mapping
- Photo OCR for nameplate auto-capture
- Multi-AHU air balance roll-up across building zones

## Domain glossary

- **TAB**: Testing, Adjusting, and Balancing
- **CFM**: cubic feet per minute, volumetric airflow
- **GPM**: gallons per minute, volumetric water flow
- **ESP / TESP**: external / total external static pressure across the AHU
- **Duct traverse**: pitot-tube readings across a duct cross-section, totaled into a single CFM
- **K-factor**: terminal-specific (or hood-specific) calibration relating reading to actual CFM
- **VP**: velocity pressure, in. wg, measured at a traverse point
- **Initial / adjusted / final**: the three phases of TAB measurement
- **Punch list**: items blocked from balancing by upstream deficiencies, assigned to a responsible party
- **Operating point**: the intersection of the pump performance curve and the loop system curve

## License

GPL-3.0

Copyright 2026 M.B. Parks, Green Shoe Garage, Cumberland, Maryland.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see the GNU General Public License version 3 in plain text from the Free Software Foundation.

## Author

M.B. Parks (N1HNP), Green Shoe Garage, Cumberland, Maryland.

Part of the Field Instruments catalog. Make. Hack. Learn. Share. Repeat.
