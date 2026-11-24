# Cut Optimization

A standalone browser application for arranging cut pieces on available stock bars. It calculates a cutting plan, shows the pieces assigned to each bar, and creates a printable report. The application is delivered as a single HTML file with embedded CSS and JavaScript. It requires no server or build step.

## Contents

- [Features](#features)
- [Getting started](#getting-started)
- [Workflow](#workflow)
- [Entering data](#entering-data)
- [Optimization approach](#optimization-approach)
- [Saving and clearing data](#saving-and-clearing-data)
- [PDF and printing](#pdf-and-printing)
- [Known limitations](#known-limitations)
- [Technical overview](#technical-overview)

## Features

- Multiple stock lengths, each with an optional quantity limit.
- Multiple piece types with quantities and cut lengths.
- Configurable saw kerf, start/end trim, and display unit.
- Paste tab-separated data copied from Excel.
- Visual cutting layouts with colors, offcuts, and material utilization.
- Reports for stock used, pieces cut, and pieces that could not be placed.
- Project fields for project number, project name, phase, and designer.
- Save inputs in the current browser's local storage.
- Print the report or save it as a PDF using the browser's print dialog.

## Getting started

1. Download or clone the repository.
2. Open the HTML file in a modern browser such as Chrome, Edge, Firefox, or Safari.
3. Enter stock, pieces, and cutting settings.
4. Select **Calculate cuts** to create a report.

The app runs entirely in the browser. No local server is required. Some browsers restrict `localStorage` for pages opened directly from `file://`. If saving inputs does not work, open the file through a local web server or review the browser's local-file storage settings.

## Workflow

### 1. Stock bars

Add one row for each stock type:

| Field | Description |
| --- | --- |
| Name | A recognizable name for the stock type. |
| Count | Available quantity. Leave blank for unlimited supply; `0` means no stock is available. |
| Length | Nominal length of each bar. |
| Colour | Optional colour label shown in the stock report. |

The initial form contains one stock type named `Bar`, with a length of `6000` and unlimited supply. Start/end trim is subtracted once from each nominal stock bar to calculate its usable length.

### 2. Pieces to cut

Add one row for each piece type:

| Field | Description |
| --- | --- |
| Name | Piece name or code. |
| Count | Required quantity. Blank means one piece; `0` means no pieces for that row. |
| Length | Length of each piece. |

Four blank piece rows are shown on a fresh start. Select **+ Add piece** to add more. Rows without a positive length are ignored.

### 3. Cutting settings

- **Saw kerf (blade width):** material removed by the blade. The calculation includes kerf between consecutive pieces and does not add an extra cut after the final piece on a bar.
- **Start/end trim:** length subtracted from every nominal stock bar before pieces are assigned.
- **Unit:** unit displayed in the report, for example `mm`.
- **Search effort:** number of attempts in the first search phase. Higher values may increase calculation time; the input is capped at 5000.

Use the same unit for all stock lengths, piece lengths, trim, and kerf.

### Pasting from Excel

Copy cells from Excel and paste them into the first applicable input cell. Use these column orders:

- Pieces: `Name`, `Count`, `Length`.
- Stock: `Name`, `Count`, `Length`, and optionally `Colour`.

A header row may be included; the app attempts to detect and skip it. Trailing blank rows are ignored.

## Optimization approach

The app uses a heuristic for the one-dimensional cutting-stock problem:

1. A fast packing pass places pieces on stock bars and selects a suitable available stock type for each bar.
2. It varies the piece order with a pseudo-random generator seeded from the input. Identical inputs therefore produce repeatable results.
3. For up to 14 distinct combinations of piece name and length, it enumerates possible cutting patterns and combines repeated patterns to cover the demand.
4. Candidate results are ranked by: fewest unplaced pieces, least total stock length, fewest bars, and fewest distinct layouts.

Pieces longer than the largest usable stock length are reported and excluded from optimization. Pieces that cannot be placed because of limited stock are listed in the report.

## Saving and clearing data

**Save inputs** stores the current form data in the current browser's `localStorage` under the key `cutopt`. On a later visit, the app attempts to restore those values automatically.

Storage is specific to the browser and origin used to open the HTML file. Another browser, browser profile, or device has separate storage. **Clear all** clears the form and report on screen but leaves the saved browser copy intact. The **Remove all stored data** link in the header deletes `cutopt` from the current browser storage and reloads the page.

## PDF and printing

Calculate a cutting plan, then select **Export to PDF**. In the browser's print dialog, choose **Save as PDF** or a printer. The input forms and controls are hidden in print mode; the report and cutting layouts remain visible.

## Known limitations

- The optimizer is heuristic and does not guarantee a mathematically optimal solution.
- Pattern enumeration is skipped when there are more than 14 unique piece-name/length combinations. If the number of generated patterns exceeds the internal limit, the app falls back to the faster search phase.
- Pattern search has an internal time limit, so results depend on search effort and input complexity.
- Pieces with the same length but different names are treated as separate types and are not automatically combined.
- Offcuts are shown in the report but are not saved as reusable stock for future jobs.
- The app does not model profile orientation, visible faces, clamping allowance, machine-specific restrictions, material defects, or other production constraints unless they are represented in the entered lengths.
- Review the cutting plan and settings before using it as a production instruction.

## Technical overview

- **HTML:** page structure and forms.
- **CSS:** screen and print styling.
- **JavaScript:** input handling, optimization, reporting, Excel paste handling, and browser storage.
- **No external dependencies:** no framework, package manager, or backend is required.

The optimizer is in the HTML file's `<script>` section. Storage, initialization, and print rules are included in the same file.
