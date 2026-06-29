# PITOT

Field Instrument #050. A testing, adjusting, and balancing (TAB) workbook for HVAC field work.

Single HTML file, no build step, no server. Opens directly from disk and persists to localStorage plus IndexedDB (for photos).

## Status

**v0.5.0 (maintenance / polish).** No new features. The Phase 4 surface area added a lot of components fast; this release walks back through it for the rough edges:

- **Print mode hardened.** All Phase 4 components that don't belong in a printed report (`.bulk-action-bar`, `.toast`, `.csv-import`, `.csv-steps`, `.ocr-host`, `.ocr-snippets`) explicitly hidden in `@media print`. Theme palette overrides now defeat with `html[data-theme]` print rules, so printing from any theme always yields the light-palette report layout rather than a colored-background print.
- **Keyboard support.** Global `Escape` closes the open modal. Visible focus rings (`:focus-visible`) added to all buttons and form controls in the brass palette, so keyboard navigation no longer disappears into the page on tab focus.
- **Toast positioning at narrow viewports.** Toasts pin to both sides (with margin) and rise above the bulk action bar at viewports ≤540px, so a long save-failed message no longer overflows the screen edge or hides behind the action bar.
- **Workbook clear protected by typed confirmation.** With existing equipment present, "Clear workbook" now requires typing `CLEAR` after the confirm dialog. Empty workbooks still take a single confirm. Clear also cleans up the IndexedDB photos for the cleared records (previously orphaned).
- **Save quota error explained.** A `QuotaExceededError` from localStorage now surfaces actionable guidance (export to JSON, clear old equipment) rather than the raw exception message.
- **`defaultTolerance()` helper.** Single source of truth for the tolerance defaults; consolidates 15+ inline duplicates across production paths (equipment modal, bulk add, CSV import, migrate). Test fixtures still use the legacy 3-field inline shape on purpose to verify migration backfills properly.

No schema bump. No data migration. Existing workbooks open and behave identically.

**v0.4.3** added CSV export, elevation correction, mobile camera capture. **v0.4.2** added asymmetric tolerance, latent loads, bulk edit. All preserved.

**These remain STYLE TEMPLATES, not official AABC/NEBB/TABB forms.** PITOT is independent software with no affiliation to any TAB certifying body. A certified submittal must use the certifying body's published forms and bear the signature of a credentialed supervisor.

## What this is

PITOT is the field workbook a TAB technician fills out, equipment by equipment, while balancing a commercial HVAC system. It captures design intent from the mechanical drawings, records initial / adjusted / final readings, compares the readings against design using configurable tolerance bands, and produces a printable TAB report.

It is named for the pitot tube, the iconic field instrument for HVAC duct traverse measurements (Henri Pitot, 1732).

## What this is not

PITOT is not an accredited TAB certification tool. The report style templates are LAYOUT TEMPLATES that follow the section ordering, terminology, and certification-statement conventions commonly seen in AABC, NEBB, and TABB submittals. They are NOT the certifying bodies' published forms, and PITOT is not affiliated with AABC, NEBB, TABB, SMACNA, or SMART.

PITOT is not a real-time controls verification tool. TAB verifies steady-state performance.

PITOT does not validate field-instrument calibration. The technician records the calibration date as documentation.

## Install and use

1. Download `pitot.html`.
2. Open directly from disk (file://) or serve it over HTTPS.
3. State persists to localStorage (workbook) and IndexedDB (photos).

To install as a homescreen app on a phone or tablet, open PITOT in a browser, then use "Add to Home Screen" from the browser menu. The inline manifest gives PITOT a proper title, icon, and standalone display mode.

No install workflow on disk, no account, no telemetry. Google Fonts is the only external resource.

## CSV import

The mechanical engineer or contractor usually provides an equipment schedule in Excel. PITOT's `+ Import CSV` button on the equipment toolbar opens a three-step modal:

1. **Input**: pick equipment type (one type per import; run the modal again for each), choose whether the first row is a header, paste the CSV text directly or upload a .csv/.tsv file. Tab-separated is auto-detected if the first row contains more tabs than commas.

2. **Mapping**: PITOT shows every detected column with a sample value from the first row, and a dropdown of valid PITOT field targets (type-aware: only fields appropriate to the chosen equipment type are listed). Suggested mappings appear automatically based on common synonyms: "Mark" suggests `tag`, "Mfr" suggests `manufacturer`, "Design CFM" suggests `cfm`, "Max CFM" suggests `cfmMax` (for VAV), and so on. Skip any column by selecting "(skip)".

3. **Preview**: shows up to 5 rows as they would be created, plus aggregate counts and warnings: tag collisions with existing equipment, rows missing a tag value, unresolved AHU references (for terminal imports). Commit to push the records into the workbook; cancel to revise the mapping.

Numeric design fields tolerate units mixed in with values: a cell containing "1500 cfm" parses to 1500. Text fields keep the whole string verbatim.

Terminal imports support an "AHU" column that maps to `associatedAhu`. PITOT resolves the AHU by tag at import time; unresolved references are flagged in the warnings list (the terminal still imports, just without the association).

Tag collisions don't block import. You can have duplicate tags in the workbook if you want to (a renovation project might have AHU-1 covering both old and new units in different phases). The preview just makes the collision visible.

## Themes

PITOT ships six themes. Cycle through them with the moon-and-sun icon in the header. The active theme persists across sessions.

- **Brass & Graphite** (default). Dark graphite background with warm brass accents. The classic look.
- **Inkwell.** High-contrast monochrome ink-on-near-black. Good for projector demos, screenshots in printed proposals, low-light conditions where the brass accents read as too warm.
- **Blueprint.** Deep cyanotype blue background with cream accents. A nod to the old reprographic blueprint look. Reads clearly in bright field conditions.
- **Foundry.** Warm copper and ash. Heavier than the default brass; suits a metalworking aesthetic.
- **Field Notes.** Kraft paper background, black-ink accents, brass highlights. Memo-pad aesthetic.
- **Locomotive.** Coal-black, signal red, brass. A nod to B&O / C&O railroad heritage. Strong contrast.

## PWA capabilities

PITOT works fully offline already because it is a single HTML file with no external dependencies except Google Fonts (which the browser caches after first load). The Phase 4 PWA additions are mostly about installability and presentation:

- **Manifest** (inline as a data URL). Provides app name, icons, theme color, and standalone display mode.
- **Meta tags** for Apple and Android PWA chrome (theme-color, apple-mobile-web-app-capable, etc.).
- **SVG icons** at 192x192, 512x512, 180x180 (Apple touch), and the favicon.
- **Service worker** registration attempt on secure contexts. Caches Google Fonts so they're available without network on subsequent visits. May not register on browsers that restrict Blob-URL SW scope; this is a graceful no-op and PITOT continues to work.

To get a "real" service worker with full offline caching, host PITOT on a domain and ship a separate `sw.js` alongside it. The single-file convention prevents shipping that file inside this distribution.

## Photo OCR

The photo modal includes a "Scan nameplate" button next to the standard "+ Add" button. Pick an image file, click Scan, and PITOT runs the browser's `TextDetector` API on it. Detected text snippets appear in a list, each with a target-field dropdown and an Apply button. Pick the design field, click Apply, and the value writes into the equipment's design block (numeric values parse the first number from the snippet; text values take the whole string).

`TextDetector` is currently supported in Chromium-based browsers on Android, ChromeOS, and recent desktops with OS-level text recognition. Safari and Firefox don't implement it yet. PITOT does not fall back to a CDN-loaded OCR library (would violate the single-file convention); when `TextDetector` is unavailable the user sees a clear "type the values manually" message.

## Multi-AHU air balance roll-up

Every equipment record now has a free-text `zone` field. When filled in, the Report tab gains a new "Multi-AHU air balance roll-up by zone" section in addition to the existing per-AHU section. For each zone in use:

- Sum the final-phase CFM across all AHUs assigned to that zone
- Sum the final-phase CFM across all terminals assigned to that zone (regardless of which individual AHU each terminal is associated to)
- Compute imbalance as `(terminalSum - ahuSum) / ahuSum * 100`
- Flag pass at +/- 10 percent, fail beyond

The per-AHU section continues to be the primary air-balance check; the zone roll-up adds a building-level view for multi-AHU systems serving a common volume.

The zone field is plain text, with no canonicalization. Type "North wing" exactly the same way on every equipment record for them to roll up together. The Bulk Add modal now exposes a zone input as a common-value field, which is the easiest way to ensure consistency when adding many terminals to the same zone.

## Workflow

1. **Project tab**: project, personnel, schedule, reporting (style, firm, supervisor, dates, scope, architect, engineer), notes.
2. **Equipment tab**: add equipment (one at a time or via Bulk Add). The Edit/Add modal now includes a Zone field next to Location and Serves.
3. **Test entry**: opens a phase-tagged test record with section-grouped readings.
4. **Status auto-classifies**: pending, in-progress, balanced, out-of-spec, or punch.
5. **Punch list**: deficiencies blocking balance.
6. **Pump equipment**: pump curve and system curve compute the operating point and render an SVG plot.
7. **Coil tests**: automatic heat-balance display when water and air readings are both present.
8. **Report**: template-driven cover, scope of work, equipment-by-type with section sub-tables, per-AHU air-balance verification, per-zone air-balance roll-up, punch list, Statement of Certification, signature block.
9. **Export**: JSON workbook (no photo binaries).

## Data model

```
{
  schema: 'pitot/workbook@4',
  version: '0.4.0',
  project: { ... reporting fields per v0.3 ... },
  equipment: [ {
    id, tag, type, location, serves, manufacturer, model, serial,
    zone,                                        /* Phase 4 */
    design: { ... per-type fields ... },
    tests: [ { id, phase, date, technician, readings, instrumentUsed,
               instrumentCalDate, notes, photos, traverse?, hood? } ],
    tolerance: { airPercent, waterPercent, tempDegF },
    status, notes, photos, associatedAhu,
    pumpCurve?, systemCurve?
  } ],
  punchList: [ ... ]
}
```

Photos in IndexedDB (`pitot.photos`). Active tab in `pitot.activeTab`. Active theme in `pitot.theme`.

### Schema migration chain

- `@1 -> @2`: adds `photos: []`, `associatedAhu`, `pumpCurve`, `systemCurve`
- `@2 -> @3`: adds reporting block to project
- `@3 -> @4`: adds `equipment.zone` (Phase 4)

Migrations run sequentially; v0.1.0 workbooks come forward to v0.4.0 in one load. Unknown future schemas are rejected with a console warning.

## Tolerance defaults

- Air: 10 percent
- Water: 10 percent
- Temperature: 2 degF (absolute)

## Conventions followed

- Single HTML file, all CSS and JS inline, no build step, no npm
- Opens from `file://`, no backend, no server, no telemetry
- localStorage + IndexedDB only
- Google Fonts (Fraunces, Oswald, Inter, JetBrains Mono) only external resource
- Dark mode default with six selectable themes; print flips to light
- No em dashes anywhere
- Mobile-responsive (768 / 540 / 380 breakpoints) with 40 px touch targets and 16 px input font on coarse pointers

## Validation performed

Static checks on the v0.5.0 release artifact:

- Line count: 8385 lines
- Em dash count: 0 across literal U+2014, `&mdash;`, `&#8212;`, and `\u2014` escapes
- CSS braces balanced: 432 / 432
- JS syntax: `node --check` passes
- HTML tag structure: matched, no mismatches, no unclosed

Logic tests (213 cases, all passed):

- Phase 1: pass/fail engine, status classification (16 cases)
- Phase 2: traverse, hood, coil heat balance, pump operating point, air balance, v1 to v2 migration (33 cases)
- Phase 3.0 - 3.3: report templates, section grouping, defensive guards, bulk tag generation, last-active-tab, tag uniqueness (57 cases)
- Phase 4.0: v3 to v4 migration, theme registry and persistence, zone air balance roll-up (17 cases)
- Phase 4.1: CSV parsing across separators, line endings, quoting, escaped quotes, BOM, empty input; mapping suggester (24 cases)
- Phase 4.2: asymmetric tolerance bounds, formatToleranceBand, psychrometric helpers, coil heat balance with and without wet-bulb data, bulk-edit semantics (33 cases)
- Phase 4.3: csvEscape; AHU vs terminal export header construction; reading and status column toggles; round-trip column-label recognition; projectBarometricPsi at sea level, Denver, non-numeric; coilHeatBalance pressure plumbing; enthalpy variation with altitude (25 cases)
- Phase 5.0: defaultTolerance always returns full asymmetric shape, fresh object per call, mutation isolation, migration of legacy 3-field tolerance backfills the asymmetric fields and preserves the legacy symmetric values (8 cases)

Not validated by automated tests (browser-only):

- Visual rendering of each of the six themes
- PWA install prompt on Chrome / Edge / Safari iOS
- Service worker registration on actual HTTPS hosts
- `TextDetector` OCR on real nameplate photos (varies by browser and OS)
- Print preview at each theme
- CSV import modal step transitions and the primary-button label update
- Pasting a real Excel-exported CSV with all the quirks (Excel quotes everything that looks like a number, sometimes)

## Known issues / known limitations

- **OCR depends on the browser.** No fallback; the message simply tells the user the platform isn't supported. Honest, but means OCR isn't universally available.
- **Service worker may not register from Blob URL** on browsers that restrict SW scope to the SW's URL path. PITOT works offline anyway since it's a single file with no runtime fetches except fonts.
- **Zone field is plain text, no canonicalization.** Spelling matters for roll-up. The Bulk Add modal helps when filling many records with the same zone.

## Deferred to later phases

- Per-body explicit form templates with body-specific row/column layouts (requires study of each body's published forms; licensed-access content)

## Domain glossary

- **TAB**: Testing, Adjusting, and Balancing
- **AABC**: Associated Air Balance Council
- **NEBB**: National Environmental Balancing Bureau
- **TABB**: Testing, Adjusting and Balancing Bureau (SMACNA/SMART)
- **TBE**: Test and Balance Engineer (AABC supervisor credential)
- **CP**: Certified Professional (NEBB supervisor credential)
- **FLA**: Full Load Amps (motor nameplate)
- **MERV**: Minimum Efficiency Reporting Value (filter rating)
- **PD**: Pitch Diameter (sheave)
- **CFM / GPM / ESP / VP**: standard HVAC quantities
- **Statement of Certification**: formal attestation paragraph in a TAB report
- **Zone**: building region used for multi-AHU air-balance aggregation; in PITOT, a free-text label on each equipment record
- **PWA**: Progressive Web Application; installable from the browser

## License

GPL-3.0

Copyright 2026 M.B. Parks, Green Shoe Garage, Cumberland, Maryland.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see the GNU General Public License version 3 in plain text from the Free Software Foundation.

## Author

M.B. Parks (N1HNP), Green Shoe Garage, Cumberland, Maryland.

Part of the Field Instruments catalog. Make. Hack. Learn. Share. Repeat.
