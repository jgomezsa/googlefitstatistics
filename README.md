# Google Fit Statistics

Exploratory analysis of a Google Fit export from [Google Takeout](https://takeout.google.com/),
in a single Jupyter notebook: [google_fit_exploration.ipynb](google_fit_exploration.ipynb).

No personal data is included in this repository — bring your own export.

## Usage

1. Request a Google Takeout export that includes **Fit**, then unzip it so this layout exists
   next to the notebook (or in any parent folder):

   ```
   Takeout/Fit/
   ├── Actividades/                   # one TCX per tracked session
   ├── Métricas de actividad diaria/  # daily + 15-min CSVs
   ├── Todas las sesiones/            # one JSON per session
   └── Todos los datos/               # raw data streams
   ```

   Folder and column names may be in another language; the notebook normalizes them.

2. Install the dependencies and open the notebook:

   ```bash
   pip install pandas numpy matplotlib jupyter
   jupyter notebook google_fit_exploration.ipynb
   ```

## What the notebook covers

1. Export inventory — file counts, sizes and date ranges per folder
2. Daily metrics — steps, distance, calories, cardio points
3. Intraday rhythm — 15-minute buckets, time-of-day routine
4. Sessions — walking / running / biking workouts
5. TCX trackpoints — inside a single session (handles per-lap distance resets)
6. Raw data streams — including which ones are empty
7. Takeaways

The `Takeout/` folder, zip archives and data files are excluded via `.gitignore`.
