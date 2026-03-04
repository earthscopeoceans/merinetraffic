# Instructions Log

# _____________________________________________
# `winnow_last_50_gps_kml`
# _____________________________________________

## Logging Rule

- Write a date header (`## YYYY-MM-DD`) only when the date changes; same-day updates use `###` subsection headings under the existing date.

## 2026-02-26

### Environment
- Use Python 3.10.15 by default unless a different version is explicitly recommended.

### KML processing requirements
- Input is a `.kml` file.
- Retain only data from `<Folder id="GPS points">`.
- Work at `<Placemark>` granularity.
- GPS uniqueness rule: two adjacent placemarks are duplicates only if both are equal:
  - parsed datetime from `<name>`
  - lat/lon from `<coordinates>`
- Keep the 50 most recent unique GPS placemarks.
- Convert datetime format in each point `<name>` from `DD/MM/YY HH:MM` to `YYYY-MM-DD HH:MM`.
- Prepend station code and separate with ` - `:
  - target format: `STATION - YYYY-MM-DD HH:MM`
  - example: `R0061 - 2021-05-12 02:17`

### Station code derivation
- Source is the first `<name>` under `<Document>`.
- Use characters after the first `-`.
- Output station code is always 5 characters.
- First and last characters are preserved; zero-pad in the middle as needed.
- Example mappings:
  - `452.120-R-0061` -> `R0061`
  - `452.020-P-24` -> `P0024`

### Completed work
- Created script: `winnow_kml_gps_points.py`
- Generated output: `position_trimmed_50.kml`
- Run summary on provided file:
  - Station code: `R0061`
  - Input GPS placemarks: `6933`
  - After adjacent dedupe: `6810`
  - Written placemarks: `50`

### Workspace note
- Continue appending future chat-derived instructions to this file.


### (Batch update)

### Script rename and output naming
- Script renamed to: `winnow_last_50_gps_kml`
- Default output name is now station-specific:
  - `last_50_gps_<STATION>.kml`
  - example: `last_50_gps_R0061.kml`

### Wrapper/batch behavior
- Added directory wrapper mode via `-p`.
- `-p` expects a parent directory containing station subdirectories.
- Each station subdirectory is scanned for `position.kml`.
- For each found file, output is written in that same subdirectory as:
  - `last_50_gps_<STATION>.kml`

### Usage
- Single file:
  - `python3 winnow_last_50_gps_kml /path/to/position.kml`
- Batch mode:
  - `python3 winnow_last_50_gps_kml -p /path/to/parent_dir`

### Validation performed
- Single-file mode validated (output in workspace):
  - `last_50_gps_R0061.kml`
- Batch mode validated on local test parent dir with one station subdir.

## Repo file-rename rule
- This is a git-tracked repo; for renames always try `git mv` first.
- If `git mv` fails, fall back to regular `mv`.


### (Python 3.12 optimizations)

### Implemented now
- Replaced full-list sort with `heapq.nlargest(limit, ...)` for selecting the most recent N points.
- Reworked datetime parsing to branch on string shape before parsing, avoiding exception-driven format probing.

### Behavior unchanged
- Output contents and naming rules remain the same.
- Dedupe rule remains adjacent duplicate removal where datetime AND lat/lon both match.

### Validation
- Single-file mode validated with output directory auto-create.
- Batch mode (`-p`) validated with output directory via `-o`.

## 2026-02-27

### (follow-up updates)

### Batch summary improvements
- Updated `winnow_last_50_gps_kml` to collect skipped station directories during `-p` runs.
- End-of-run batch output now prints:
  - processed and skipped totals
  - explicit skipped directory list with reason

### Hidden directory handling
- Batch scanning now ignores dot-directories in parent input path (e.g. `.git`, `.cache`, hidden folders).
- Dot-directories are excluded from processing and from skipped counts.

### Output-path behavior retained
- `-o` accepts either:
  - a file path (`.kml`) in single-file mode, or
  - a directory path (auto-created if missing), used in single-file or batch mode.

## 2026-03-04

### (SOM `_all.txt` support)

### Script refactor
- Extended `winnow_last_50_gps_kml` to support ingesting SOM `*_all.txt` sources from the Princeton hosted index.
- Added CLI mode:
  - `--som-all` to process all matching `*_all.txt` files from the index URL
  - `--som-url` to override source URL (defaults to the Princeton SOM page)

### SOM parsing behavior
- Parses first four relevant fields from each row:
  - station
  - date/time
  - lat
  - lon
- Date parsing for SOM rows uses `%d-%b-%Y %H:%M:%S`.

### Output behavior
- Generates one output KML per station file as:
  - `last_50_gps_<STATION>.kml`
- Applies existing selection rules:
  - adjacent duplicate removal where datetime AND lat/lon both match
  - keep most recent 50 points after dedupe
  - output placemark names in `STATION - YYYY-MM-DD HH:MM`

### Existing modes retained
- Single KML mode (positional input file)
- Batch local directory mode (`-p`)


### (class-based refactor)

### Architecture update
- Refactored `winnow_last_50_gps_kml` to class-based structure using `GPSKMLWinnower`.
- Added two primary methods:
  - `process_kml_file(...)` for KML input
  - `process_online_som_all(...)` for online SOM `*_all.txt` ingestion

### Supporting method split
- Common dedupe + recent-selection logic is centralized in `select_recent_unique(...)`.
- KML writer logic is centralized in `write_records_to_kml(...)`.
- Batch local folder mode now reuses `process_kml_file(...)`.

### Behavior retained
- Most-recent-50 selection after adjacent dedupe (datetime AND lat/lon match).
- Name formatting: `STATION - YYYY-MM-DD HH:MM`.
- Dot-directory skip in local batch mode and skipped-item summaries.


### (verification + output differentiation planning)

### Verification completed
- Verified KML method with local `position.kml`:
  - station parsed as `P0006`
  - wrote 50 placemarks to `last_50_gps_P0006.kml`
- Verified SOM text method using local mock index and `_all.txt` files:
  - processed multiple station files
  - applied adjacent dedupe and recent-point selection
  - generated station-scoped KML outputs with expected name/date formatting

### Next design consideration captured
- Need a stable convention to differentiate output artifacts by input source (KML vs SOM text), potentially through filename and/or KML metadata.


### (provenance implementation verification)

### Source-differentiated filenames
- KML output now uses `last_50_gps_<STATION>__src-kml.kml`.
- SOM output now uses `last_50_gps_<STATION>__src-som-all.kml`.

### Embedded provenance metadata
- Document `ExtendedData` now includes:
  - `source_type`
  - `source_ref`
  - `generated_utc`
  - `limit`

### Verification status
- KML path verified with local `position.kml`:
  - wrote 50 placemarks
  - produced `last_50_gps_P0006__src-kml.kml`
  - metadata fields present
- SOM path already verified in prior run:
  - produced station outputs with `__src-som-all` suffix
  - metadata fields present


### (KML vs SOM positional verification)

### Verification run
- Generated KML-source output from local `position.kml` for station `P0006`.
- Generated SOM-source outputs from live Princeton `*_all.txt` index.
- Compared `P0006` output pairs (`__src-kml` vs `__src-som-all`).

### Result
- Not nearly identical for this specific pair.
- Date windows differ substantially:
  - local KML output points span 2024-07 to 2024-10
  - SOM output points span 2025-03 to 2025-06
- No overlapping output timestamps in the final 50-point windows.
- Large nearest-neighbor separation confirms mismatch for this sample pair.

### Interpretation
- The two inputs are likely not synchronized snapshots for the same station/time period.
- Comparison should be done on overlapping date windows (or same source epoch) to validate positional agreement.



### (script rename)

### Rename
- Renamed standalone script to `gps_winnower.py`.
- `git mv` was attempted first; fallback `mv` was used due to local `.git/index.lock` permission issue.

### Docs
- Updated README references and usage examples to `python3 gps_winnower.py ...`.


### (output filename separator tweak)

### Naming change
- Updated output filename convention to remove double underscore before source tag.
- New format:
  - `last_50_gps_<STATION>_src-kml.kml`
  - `last_50_gps_<STATION>_src-som-all.kml`

### Verification
- Local run confirmed output now uses single underscore before `src-`.
- README examples updated to match.


### (README input-mode clarification pass)

### Documentation update
- Reworked README to explicitly document all input modes and defaults:
  - single local KML (`input_kml`)
  - local station batch (`-p`)
  - online SOM batch (`--som-all`)
- Added explicit default SOM URL and `--limit` default.
- Added mode-specific `-o` behavior and default output locations.
- Clarified mutual exclusivity of input modes and current source-tagged output naming.

### (versioning bootstrap)

### Versioning files
- Added `VERSION` with initial value `0.1.0`.
- Added `CHANGELOG.md` with initial `0.1.0` release notes.

### CLI update
- Added `--version` flag to `gps_winnower.py`.
- `--version` reads from `VERSION` (fallback: `0.0.0+unknown`).

### Docs
- Updated README to document `--version`, `VERSION`, and `CHANGELOG.md`.
