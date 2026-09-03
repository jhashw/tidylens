# TidyLens — Specification

What the tool was built to do, and what changed once it met real data.

## Problem

The first pass over an unfamiliar dataset is the same work every time: work out what each column should contain, catch values that don't belong, standardize dates and text, then look at distributions. That pass is usually done ad hoc in a notebook and thrown away.

Goal: declare the cleaning rules explicitly, have the tool enforce them, get back both a cleaned file and an analysis view.

Constraint: runs entirely in the browser. The datasets most worth cleaning are the ones that can't be uploaded to a third-party service.

## Original requirements

**Input** — CSV or XLSX. Test dataset: food delivery orders (order ID, customer ID, restaurant, cuisine, cost, day of week, rating, prep time, delivery time).

**Cleaning**
- Schema validation
- A data type per column
- An allowed list of values per column, enforced
- Explicit date format selection for time columns
- String handling: special characters, encoding, trailing whitespace, case standardization
- Flag anything else inconsistent and ask the user how to handle it

**Analysis**
- Summary statistics: mean, median, standard deviation
- Histograms
- Any two columns plotted against each other
- Correlations
- Outlier identification

**UI flow**
- Drag and drop or open a file
- Preview the data with row and column counts
- Per column: set type, set allow or disallow list, pick time format
- Export the cleaned file
- Analyze step with filtering, hover to inspect, fitted trendline
- Dark mode
- Filters synced across all plots

## What changed during iteration

### Parsing
- Time-only values now parse (`13:41`, `13:41.4`, am/pm, fractional seconds) — a timestamp column was being missed entirely
- Added `YYYYMMDD` and Excel serial date handling
- Date detection reworked: counts dates independently of numbers, weighs column-name hints (`_at`, `date`, `time`), detects Unix-epoch ranges
- Replaced the CDN spreadsheet library with a self-contained `.xlsx` reader (unzip, shared strings, style-driven date formats)
- Embedded the example dataset instead of fetching it
- Last two both serve the no-network constraint: a tool promising data never leaves the machine shouldn't make requests to load

### Cleaning
Requirement 6 above, flagging inconsistencies and asking how to handle them, grew into most of the cleaning interface:
- Custom fill value for blanks on every data type, with type validation and one-click "Switch to Text" when the value won't cast
- Inline "Standardise to this" on flagged issues, so one field fixes a column
- Row-level audit: rows failing across most columns can be kept, blanked, or dropped
- Identical issues across columns group into one row with a single fix-all action
- Filled cells counted separately from blanked ones

Also:
- Column rules single-select; "Add a column to edit" opens more without losing edits
- Time-only columns default to a time output format, so they stop printing `1970-01-01`
- ID-like columns excluded from plot and stats defaults — an order ID has a mean, but not a meaningful one

### Analysis
- Multiple scatter charts, each with independent axes, trendline and outlier state
- Per-chart "remove outliers from plot"
- Trendline off by default, equation shown when on — a fitted line on an unexamined plot implies a relationship the user hasn't verified
- Time-series mode: time axis, multi-select series, line-only rendering, 0–1 normalization, per-series trend and outliers, drag-to-filter on time
- Hover snaps only to buckets holding data
- Per-series statistics compute from raw values, not bucket means
- Histogram bin controls exposed
- Export: save picker, falling back to top document, then in-document anchor, with a panel explaining the block when the embedding frame forbids downloads

### Interface
- Dark mode by default; named TidyLens, brand in accent gold
- Raw-data preview step added ahead of the schema page
- Source page rebuilt: short blurb, drop pane, two feature columns
- Schema page: three separated sections, larger action buttons, solid Analyse, more spacing
- Column list marks problem columns red, clean ones green
- Empty state in Column rules ("Pick a column to get started")
- Clearer wording: "Blank when value is", "selected" rather than "ticked"
- Analysis panes given pale per-column tints instead of monochrome
- Larger, distinct plot headings; axis labels as HTML overlays; tick count adapts to width
- Natural-language blurb under each analysis pane
- Hover explanations for Pearson r, R², slope, point counts
- Correlation matrix color-coded green and red with a legend
- Fonts vendored locally

## Implementation note

Built with Claude Design against this specification. The behaviour and interface above were refined over subsequent iterations.
