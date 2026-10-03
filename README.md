# Google Fit Statistics

Exploratory analysis of a Google Fit export from [Google Takeout](https://takeout.google.com/),
in a single Jupyter notebook: [google_fit_exploration.ipynb](google_fit_exploration.ipynb).

No personal data is included in this repository — bring your own export.

## 1. Get your Google Fit data

1. Go to [takeout.google.com](https://takeout.google.com/) and sign in with the Google account
   that holds your Fit data.
2. Click **Deselect all**, then scroll down and tick only **Fit**. Leave "All Fit data
   included" as it is.
3. Click **Next step**, then choose:
   - **Transfer to:** send download link via email
   - **Frequency:** export once
   - **File type & size:** `.zip`. Any size works; the Fit export is usually a few hundred MB.
4. Click **Create export**. Google emails you a download link when the export is ready, which
   can take from a few minutes to a few hours.
5. Download the zip and extract it into the root of this repository, so the folder
   structure looks like this:

   ```
   googlefitstatistics/
   ├── google_fit_exploration.ipynb
   └── Takeout/
       └── Fit/
           ├── Actividades/                   # one TCX per tracked session
           ├── Métricas de actividad diaria/  # daily + 15-min CSVs
           ├── Todas las sesiones/            # one JSON per session
           └── Todos los datos/               # raw data streams
   ```

   The folder names follow your Google account's language. The notebook recognizes both the
   Spanish names above and their English equivalents (`Activities/`, `Daily activity metrics/`,
   `All sessions/`, `All data/`). The `Takeout/` folder may also sit in any parent folder of
   the notebook.

`Takeout/`, zip archives and data files are excluded via `.gitignore`, so your data is never
committed.

## 2. Set up and run

Create the environment with [uv](https://docs.astral.sh/uv/). It installs Python 3.12 if
needed, plus the exact package versions pinned in `uv.lock`. Then open the notebook:

```bash
uv sync
uv run jupyter lab google_fit_exploration.ipynb
```

To use VS Code instead, select `.venv` as the notebook kernel.

If you plan to commit, enable the output-stripping git filter once per clone, so no personal
results end up in the repository:

```bash
uv run nbstripout --install
```

To run the notebook headless (the executed copy goes to `out/`, which is not tracked):

```bash
uv run jupyter nbconvert --to notebook --execute google_fit_exploration.ipynb --output-dir out
```

## 3. What you get

The notebook runs top to bottom and produces these tables and charts.

**1. Export inventory**
- Table of file counts, sizes and date ranges for each Takeout folder

**2. Daily metrics**
- Completeness of each metric (how many days actually have a value)
- Daily step count with 7- and 30-day averages and a 10,000-step reference line
- Distance, active minutes and cardio points over time (30-day averages)
- Average daily distance per month
- Steps by weekday (box plot) and by month of the year
- Calendar heatmap of steps, one row per year
- Share of days reaching 10,000 steps each month, plus the longest goal streaks
- Tracked walking / running / biking minutes per month
- Correlation matrix between the daily metrics

**3. Intraday rhythm (15-minute buckets)**
- When the steps happen: average steps per 15 minutes across the day, one line per weekday
- Heatmap of average steps by hour and weekday
- Share of each day's steps taken before noon

**4. Sessions (tracked workouts)**
- Summary table per activity type (count, hours, km, median duration and pace)
- Sessions per month, stacked by activity
- Distribution of session duration and distance
- Pace against distance, and median pace per month
- What time of day sessions start
- Cross-check of session distance against the daily totals

**5. TCX trackpoints (inside a session)**
- Trackpoint sampling interval
- Cumulative distance and speed within the three longest sessions
- Cross-check of TCX distance against the session summaries

**6. Raw data streams**
- Catalog of every raw stream with point counts, including which ones are empty
- Duration and size of raw step-count points
- Raw step deltas on the busiest day
- Daily cardio points from the raw stream
- Hours per classified activity (walking, still, in vehicle, …)
- Hand-entered weight and height values

**7. Takeaways**
- Headline numbers for your export, and notes on what the data can and cannot support

## 4. Location heatmap (Google Maps Timeline)

A second notebook, [timeline_heatmap.ipynb](timeline_heatmap.ipynb), maps your location
history. It doesn't use Google Fit data, so it needs its own export.

1. Export Timeline from the phone, because Takeout no longer includes it for most accounts:
   - **Android:** Settings → Location → Location services → Timeline → **Export Timeline data**
   - **iPhone:** Google Maps → Your Timeline → settings → export
2. Put the exported JSON in a `Timeline/` folder next to the notebook. The file name
   depends on the phone's language (`Timeline.json`, `Cronología.json`, …), and the notebook
   finds it by its contents. An older Takeout `Records.json` (Location History) also works.
3. Run `uv run jupyter lab timeline_heatmap.ipynb`.

It produces:
- Recorded points per year, by kind of record (route points, visits, trips, raw GPS fixes)
- A world map with every point drawn at low opacity
- The main region and your busiest city, each as low-opacity points and as a hexagon
  density map
- An interactive full-page heatmap (`out/timeline_heatmap.html`) with live sliders for blob
  size, blur, intensity and compression, and a dark / light / street basemap switch. Its style
  follows [ry-li/google-maps-timeline-heatmap](https://github.com/ry-li/google-maps-timeline-heatmap).

The basemap tiles come from Esri, so its servers see which map area is being drawn.

## Limitations

For the Fit notebook, what you can analyse depends on what Google Fit recorded. With a phone only (no wearable),
expect to have:
- **Usable:** steps, distance, speed and activity time.
- **Empty:** sleep, heart rate and respiratory rate. The streams exist in the export but
  contain no data.
- **No routes:** the TCX files have no GPS coordinates.
- **Unreliable cycling distance:** a pocket pedometer can't measure it, so only cycling
  duration is meaningful.
