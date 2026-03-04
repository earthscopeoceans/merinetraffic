## merinetraffic
## Tools to interface MERMAIDs with MarineTraffic.com

### _______________________________________
### KML Winnowing for "Custom Area" imports

### Script: `gps_winnower.py`
### Purpose: reduce GPS history to last N unique points and export KML

### Supported Input Modes
The script supports three mutually exclusive input modes.

1. Single local KML file (`position.kml` style)
- Provide positional `input_kml`.
- Example:
```bash
python3 gps_winnower.py /path/to/position.kml
```

2. Local station directory batch (`-p`)
- Provide `-p /path/to/parent_dir`.
- Parent dir should contain station subdirectories, each with `position.kml`.
- Hidden dot-directories are ignored.
- Example:
```bash
python3 gps_winnower.py -p /path/to/stations_parent
```

3. Online SOM `*_all.txt` batch (`--som-all`)
- Provide `--som-all` to fetch and process all matching files from SOM index.
- Uses SOM URL default unless overridden with `--som-url`.
- Example:
```bash
python3 gps_winnower.py --som-all
```

### SOM URL Default
If `--som-all` is used and `--som-url` is not provided, default source is:
- `https://geoweb.princeton.edu/people/simons/SOM/`

Override example:
```bash
python3 gps_winnower.py --som-all --som-url "https://geoweb.princeton.edu/people/simons/SOM/"
```

### Core Processing Rules
- Keep only points from KML `<Folder id="GPS points">` (for KML inputs).
- Uniqueness rule: adjacent duplicate points are dropped only when both match:
  - datetime
  - lat/lon
- Keep most recent points after dedupe.
- Default point count is 50 (`--limit 50`).
- Output `<name>` format:
  - `STATION - YYYY-MM-DD HH:MM`

### Station Code Rule
Station code is always 5 characters, derived from document/station naming conventions.
Examples:
- `452.120-R-0061` -> `R0061`
- `452.020-P-24` -> `P0024`

### CLI Options and Defaults
- `input_kml`:
  - positional; used only in single-file mode
- `-p, --path`:
  - local batch mode root directory
- `--som-all`:
  - online SOM batch mode
- `--som-url`:
  - SOM index URL
  - default: `https://geoweb.princeton.edu/people/simons/SOM/`
- `--limit`:
  - number of most recent unique points to keep
  - default: `50`
- `--version`:
  - prints the current script version from `VERSION`
- `-o, --output`:
  - output file or directory depending on mode (see below)

### Output Behavior by Mode
1. Single KML mode (`input_kml`)
- `-o` omitted:
  - writes next to input as `last_50_gps_<STATION>_src-kml.kml`
- `-o` is `.kml` path:
  - writes exactly that file
- `-o` is directory path:
  - creates directory if needed
  - writes `last_50_gps_<STATION>_src-kml.kml` inside

2. Local batch mode (`-p`)
- `-o` omitted:
  - writes output per station in each station subdirectory
- `-o` directory provided:
  - creates directory if needed
  - writes all station outputs there
- skipped station dirs (for example missing `position.kml`) are summarized at end

3. SOM online mode (`--som-all`)
- `-o` omitted:
  - writes into default directory: `som_last_50_kml`
- `-o` directory provided:
  - creates directory if needed
  - writes all station outputs there
- `-o` as `.kml` file is not allowed in this mode

### Output Naming Convention
- KML source output:
  - `last_50_gps_<STATION>_src-kml.kml`
- SOM source output:
  - `last_50_gps_<STATION>_src-som-all.kml`

Examples:
- `last_50_gps_R0061_src-kml.kml`
- `last_50_gps_R0061_src-som-all.kml`

### Embedded KML Provenance Metadata
Generated KML includes `Document/ExtendedData` keys:
- `source_type`
- `source_ref`
- `generated_utc`
- `limit`

### Last-Tested Runtime Environment
- OS: Darwin Kernel Version 23.6.0 (`RELEASE_ARM64_T6031`)
- Python: 3.12.4

### Versioning
- Current version is stored in `VERSION`.
- Show version:
```bash
python3 gps_winnower.py --version
```
- Release notes are tracked in `CHANGELOG.md`.

## Contact
- Joel D. Simon: `jdsimon@bathymetrix.com`

## Attribution
- Script development and iterative modifications were performed in collaboration with Codex (OpenAI).
