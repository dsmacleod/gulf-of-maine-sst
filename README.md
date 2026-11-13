# How warm is the Gulf of Maine?

A Bangor Daily News interactive chart of daily Gulf of Maine sea-surface temperatures (SST) from 1981 to the present. Each year is drawn as a line against the climatology baseline, with callouts for the latest temperature, the anomaly versus the baseline, and how the current year ranks.

**Live page:** https://dsmacleod.github.io/gulf-of-maine-sst/gulf-of-maine-sst.html

## How it works

- `gulf-of-maine-sst.html` is a self-contained page (D3 v7, loaded from d3js.org) that fetches `gom_sst.json` from the same directory and renders the chart, the callouts, and decade highlight buttons (1980s through 2020s).
- The page treats series named like `1991-2020` as climatology baselines (it uses the most recent one), a series named `Preliminary` as preliminary data, and every series named with a four-digit year as that year's daily values. Temperatures are shown in °F, with anomalies also given in °C.
- GitHub Pages serves the repo root from the `main` branch.

## Data

- **Source:** NOAA OISST v2.1, area-weighted daily means for the Gulf of Maine, via [Climate Reanalyzer](https://climatereanalyzer.org/clim/sst_daily/) (University of Maine Climate Change Institute). The data runs 1 to 2 days behind.
- **File:** `gom_sst.json`, an unmodified copy of `https://climatereanalyzer.org/clim/sst_daily/json_2clim/oisst2.1_gom_sst_day.json`. It holds an array of series objects (`name`, `data`, `data_source`) with one value per day of the year.

## Automation

`.github/workflows/update-data.yml` runs daily at 10:00 UTC (5 or 6 a.m. Eastern) and can also be started by hand (`workflow_dispatch`). It downloads the JSON with `curl` and commits `gom_sst.json` as `github-actions[bot]` only when the file has changed. No secrets are needed; the workflow uses the default `GITHUB_TOKEN` with `contents: write`.

## Running locally

The page loads its data with `fetch`, so open it through a local web server rather than as a `file://` URL:

```sh
python3 -m http.server 8000
# then open http://localhost:8000/gulf-of-maine-sst.html
```

To refresh the data by hand:

```sh
curl -sf -A "Mozilla/5.0" -o gom_sst.json \
  "https://climatereanalyzer.org/clim/sst_daily/json_2clim/oisst2.1_gom_sst_day.json"
```

## Other files

- `newspack-merlinone-review.md` is an unrelated review of a Newspack/MerlinOne DAM plugin spec. The chart does not use it.
