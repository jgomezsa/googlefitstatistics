# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

Exploratory analysis of a Google Fit export from Google Takeout. Everything lives in one
Jupyter notebook, [google_fit_exploration.ipynb](google_fit_exploration.ipynb); there is no
package, test suite or build step.

## Environment (uv)

Managed with uv: Python pinned in `.python-version` (3.12), dependencies in `pyproject.toml`,
exact versions locked in `uv.lock` (commit both). Runtime deps: `pandas numpy matplotlib`;
dev group: `jupyter nbstripout`.

```bash
uv sync                                    # create/update .venv from the lockfile
uv run jupyter lab google_fit_exploration.ipynb
uv add <pkg>   /   uv add --dev <pkg>      # never pip install into .venv
# Verify the whole notebook runs (output to gitignored out/):
uv run jupyter nbconvert --to notebook --execute google_fit_exploration.ipynb --output-dir out
```

After changing notebook code, run the headless command above to check it still executes.
Note: pandas is 3.x (copy-on-write, string dtype by default), so avoid chained assignment.

The data (`Takeout/Fit/`) is **personal and never committed** — `.gitignore` excludes
`Takeout/`, zips, `*.csv`, `*.tcx` and `*.json`. The notebook finds it by walking up from
the working directory (`find_fit_root()`).

## Privacy rules

- Commit the notebook **with outputs cleared**. Outputs contain personal numbers, dates and
  charts from the export. `.gitattributes` routes `*.ipynb` through the `nbstripout` filter,
  but the filter only works after `uv run nbstripout --install` has been run in that clone.
  Check with `git config --get filter.nbstripout.clean`.
- Don't hardcode values taken from a specific export (dates, totals, filenames, weight,
  height) in code or markdown cells; compute them at runtime.
- Don't add emails, names or other identifying details to committed files. Local-only notes
  belong in a gitignored file.

## Data layout (`Takeout/Fit/`)

| Folder | Contents | Granularity |
|---|---|---|
| `Métricas de actividad diaria/` | one CSV per day + one aggregated CSV (same name as the folder) | 15-min buckets / daily |
| `Todas las sesiones/` | one JSON per tracked session | one row per workout |
| `Actividades/` | one TCX per tracked session | trackpoint (~20–60 s) |
| `Todos los datos/` | raw & derived Fit data streams (large, ~hundreds of MB) | raw sample |

Folder and column names depend on the export locale (the reference export is Spanish).
Session and TCX filenames carry the **local** start time + UTC offset
(`YYYY-MM-DDTHH_MM_SS+HH_MM_<label>`); `startTime` inside the session JSON is UTC.

## Notebook structure

Setup → Plot style → 7 numbered sections (anchors `#1`–`#7`):

1. Export inventory — file counts / sizes / date ranges per folder
2. Daily metrics — aggregated daily CSV, completeness, trends, calendar heatmap, 10k-step goal
3. Intraday rhythm — all per-day CSVs, 15-min buckets
4. Sessions — session JSONs, per-activity stats, pace, cross-check against daily totals
5. TCX trackpoints — per-session detail, cross-check against session JSONs
6. Raw data streams — `Todos los datos/`, including which streams are empty
7. Takeaways — headline numbers + what the data does / does not support

Key DataFrames built along the way: `daily`, `intraday`, `sessions`, `tcx`, `catalog`,
`steps_raw`. Later cells depend on earlier ones — run top to bottom.

## Conventions

- **Locale-independence**: `_norm()` lowercases and strips accents; `_subdir()` matches
  folders by Spanish *or* English keywords; `tidy_columns()` maps columns through `COLMAP`
  to English snake_case. New lookups should go through these, not literal Spanish names.
- **Timezone**: `TZ = 'Europe/Madrid'`, used when converting raw-stream nanosecond timestamps.
- **Plot styling is centralized** in the "Plot style" cell — reuse it, don't restyle per chart:
  - colors: `SURFACE`, `INK`, `INK_SOFT`, `INK_MUTED`, `GRID`, `AXIS`
  - `SERIES`: categorical palette assigned in fixed order, never cycled
  - `ACT_COLOR`: stable color per activity type (walking / running / biking)
  - `BLUES`: single-hue sequential ramp for heatmaps
  - `finish(ax, title, subtitle, ylabel, xlabel, yfmt)`: shared title/subtitle/grid/tick chrome
- Large `Todos los datos/` files are one data point per line: stream them with
  `iter_points()` / `load_stream()` / `scan_stream()` rather than `json.load`.
- Code style: small helper functions, short comments explaining *why*, single-quoted strings,
  f-strings with thousands separators for printed numbers.

## Established data-quality findings

These come from analysing the reference export; don't re-derive them, but keep in mind
another export may differ.

- **Steps, distance, speed** are complete and reliable; the daily CSV, session JSONs and TCX
  files agree where they overlap.
- **Calories** cover only part of the days and partly come from a third-party app, so
  calorie trends reflect app setup rather than energy expenditure.
- **Walking/running/biking durations** in the daily CSV are only filled on days with a
  tracked session.
- **Session files (JSON/TCX) cover a narrower window** than the daily series (they start with
  a later phone). A device change can bias long-term step trends.
- **Empty streams**: sleep, heart rate, respiratory rate. Weight/height are single hand-entered
  points. This is a phone-pedometer dataset (no wearable).
- **No GPS** in the TCX files (no `LatitudeDegrees`), so no route or elevation analysis.
- **Biking distance and pace are artefacts** (pedometer in a pocket on a bike); only biking
  *duration* is usable. Biking is excluded from all pace charts.
- **TCX `DistanceMeters` resets at every `<Lap>`.** `parse_tcx()` adds a running lap offset
  to get session-cumulative distance. Taking the file-wide max loses ~30% of distance.
  The bug was caught by the TCX-vs-session-JSON cross-check, so keep cross-checks like that
  when adding new parsers.
- Instantaneous TCX speed: drop sample pairs < 5 s apart or spanning a lap boundary
  (jitter otherwise reads as absurd speeds).
