# AGENTS.md

Repo-level instructions for future Codex threads.

## Defaults

- Use Python `3.12.4` unless explicitly told otherwise.
- Treat `gps_winnower.py` as the main script entrypoint.
- Do not include local absolute path prefixes in docs, logs, or user-facing text unless explicitly requested.

## Editing And Repo Rules

- This is a git repo. For renames or moves, try `git mv` first.
- Do not use plain `mv` for renames or moves.
- If `git mv` cannot be used because of permissions or repo state, stop and let the user handle the rename manually.
- Do not bump `VERSION` unless the user explicitly asks.
- Do not amend commits unless the user explicitly asks.

## Project Files

- Keep `README.md` current when CLI behavior, defaults, filenames, or input modes change.
- Keep `CHANGELOG.md` accurate, but do not create a new release version unless the user asks.
- If unreleased changes make the changelog inaccurate, prefer an `Unreleased` section rather than silently changing an existing released entry.
- Keep `codex_prompts.md` updated with durable work-log notes unless the user explicitly pauses logging.

## codex_prompts.md Logging Rule

- Add a new date header (`## YYYY-MM-DD`) only if that date does not already exist as the current active date section.
- For additional updates on the same date, append `### (...)` subsections under the existing date.
- Keep entries concise and focused on lasting project decisions, not transient terminal output.

## gps_winnower.py Behavior To Preserve

- Supported input modes:
- Single local KML file via positional `input_kml`
- Local batch mode via `-p`
- Online SOM batch mode via `--som-all`
- Default SOM URL is `https://geoweb.princeton.edu/people/simons/SOM/` unless overridden with `--som-url`.
- Default output filename pattern is:
- `recent_gps_<STATION>_src-kml.kml`
- `recent_gps_<STATION>_src-som-all.kml`
- Output placemark names use:
- `STATION - DD-Mon-YYYY HH:MM`
- KML output should continue to include provenance metadata in `Document/ExtendedData`:
- `source_type`
- `source_ref`
- `generated_utc`
- `limit`

## README Content Expectations

- Document all supported input modes explicitly.
- Document mode-specific `-o/--output` behavior explicitly.
- Document default values explicitly, including SOM URL and limit.
- Keep examples aligned with the current script name and output naming convention.
