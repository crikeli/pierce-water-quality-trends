# Pierce County Water Quality Index: QA & Change (Demo)

Pierce County's Surface Water Management division publishes a Water
Quality Index (WQI, 0-100) for real monitoring stations across the county
- bacteria, dissolved oxygen, pH, phosphorus, total suspended solids,
temperature, nitrogen, and turbidity, each converted to a sub-index and
combined into an overall score. This project audits that public data for
correctness, compares each station's real 2023 and 2024 annual scores,
and cross-references the result with this portfolio's own stormwater
outfall-audit project.

**Live site: https://crikeli.github.io/pierce-water-quality-trends/**

## Two scope corrections, made before building anything further

This started as a "multi-year trend" idea. Checking the real data first
showed that's not accurate - the public layer holds mostly two real
observation years (2023 and 2024) per station, with a handful of
scattered legacy rows from 2014-2018. So this is a real **2023-to-2024
comparison**, not a multi-year trend line - corrected before writing a
single line of analysis code, not after.

Second: the parameter fields (`PH_ANNUAL`, `DO_ANNUAL`, etc.) turned out
to be pre-converted 0-100 WQI sub-index scores, not raw pH/DO field
measurements - so the QA/QC here checks the index computation and
publication, not raw field-instrument readings against physical bounds
like pH 0-14.

## Real findings

**QA/QC passed cleanly on range validity** - every sub-score and overall
score across 146 records falls within [0, 100]. Reported as a real result
in its own right, not just checks that happened to find nothing.

**`OVERALL_ANNUAL` isn't a simple average of its 8 components** (0.83
correlation, but runs ~15-16 points below a plain mean) - consistent with
standard WQI methodology weighting the worst-performing parameter more
heavily than a flat average would. Documented as expected behavior, not
flagged as an error.

**The same duplicate-station-naming bug from the sibling outfall-audit
project reappears here on a fresh fetch** - `CanyonFallsCreek` vs
`CanyonfallsCreek`, `25MileCreek` vs `Twenty-fiveMileCreek`, same physical
coordinates. Confirms it's a persistent source-data issue, not a one-time
fetch artifact - caught with a corrected version of the same
coordinate-based dedup logic (an earlier draft of this notebook's own
duplicate check had a bug: it flagged every station with 2 real
observation years, not just the ones sharing a coordinate under different
names - fixed before it produced a misleading result).

**2023 to 2024: essentially flat citywide (mean change +0.3), with real
station-to-station variation** - 59 stations compared, 33 improved, 23
declined. Full per-station numbers in the notebook.

**A genuine cross-project question, and an honest null result.** Do water
quality stations near more unconfirmed stormwater outfalls (from the
sibling `pierce-stormwater-outfall-audit` project) show worse or more
declining water quality? Correlation came back essentially zero (-0.02 to
-0.12) - reported as a real null result with a modest sample (59
stations, 1km window), not reframed to look like a finding.

## Repo layout

```
notebooks/
  water_quality_qa_and_change.ipynb   # the full, executed analysis
data/
  build_static_site.py    # renders docs/index.html from the notebook's output
docs/
  index.html                # the deployed static site
  water_quality_change.geojson   # 59 stations, 2023 vs 2024, ~16KB
environment.yml
```

Raw fetches from Pierce County's GIS services (`data/*.geojson`) are
produced locally but git-ignored - the notebook regenerates them from
source on request. The cross-project reference reads
`../pierce-stormwater-outfall-audit/docs/outfalls_all.geojson` directly
from that sibling repo checked out alongside this one.

## Setup

```bash
conda env create -f environment.yml
conda activate pierce-water-quality-trends
```

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/water_quality_qa_and_change.ipynb
python data/build_static_site.py
```

## Stack

geopandas, requests (Pierce County's ArcGIS REST services) - Leaflet for
the deployed static site, Esri World Imagery for basemap context.
