# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog, and this project follows Semantic Versioning.

## [0.1.0] - 2026-03-04

### Added
- Standalone `gps_winnower.py` CLI supporting:
  - single-file KML input mode
  - local batch directory mode (`-p`)
  - online SOM `_all.txt` mode (`--som-all`, `--som-url`)
- Last-N unique point selection (default `--limit 50`) with adjacent duplicate removal.
- Source-tagged output naming:
  - `last_50_gps_<STATION>_src-kml.kml`
  - `last_50_gps_<STATION>_src-som-all.kml`
- Embedded output provenance metadata (`source_type`, `source_ref`, `generated_utc`, `limit`).
- CLI version reporting via `--version`, sourced from `VERSION`.
