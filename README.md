# Canterbury Road Crash Hotspots: Emerging Trends 2000–2022

A spatial and space-time analysis of 82,146 reported road crashes across the Canterbury region, built end to end in ArcGIS Pro with a fully scripted `arcpy` workflow. The project identifies where crashes concentrate, and, more usefully, classifies **how each black-spot has changed over two decades**, distinguishing locations where risk is intensifying from those where it has cooled.

> **Live web app:** _[coming soon: ArcGIS Online dashboard link]_

![Canterbury emerging crash hotspots, 2000 to 2022](docs/canterbury-emerging-hotspots.png)

---

## Why this project

Most crash maps show you where crashes are. That is useful, but it is a snapshot, and a location that looks dangerous today may already be improving. The more actionable question for anyone prioritising road safety spend is *which* black-spots are getting worse, which are chronic, and which are on the mend. This project answers that by combining density and cluster analysis with a space-time trend classification over 23 complete years of data.

It also demonstrates a practical, reproducible GIS pipeline: a real open dataset taken from raw download through cleaning, projection, multiple spatial-statistics workflows, a documented data-quality decision, and a publication-quality cartographic output, all driven from Python inside ArcGIS Pro.

## Key findings

- Across roughly 13,300 analysis cells, **diminishing hot spots outnumber intensifying ones by more than forty to one** (372 diminishing versus 9 intensifying). In plain terms, far more of Canterbury's crash black-spots have cooled over the study period than have worsened.
- A **persistent red core remains in central Christchurch** (162 persistent hot spots), where crash risk has stayed statistically high year after year. These chronic locations, not the rare intensifying ones, are where sustained attention is most warranted.
- The annual crash count peaked at **4,542 in 2007** and then declined to a plateau of roughly **3,200 per year**, consistent with the broadly cooling spatial trend.

| Emerging trend classification | Cells | Reading |
|---|---|---|
| No Pattern Detected | 12,476 | No significant space-time trend |
| Diminishing Hot Spot | 372 | Risk cooling over time |
| Sporadic Hot Spot | 233 | On-and-off hot spot |
| Persistent Hot Spot | 162 | Chronically high risk |
| Historical Hot Spot | 30 | Was hot, since cooled |
| Consecutive Hot Spot | 25 | Recent sustained heat |
| New Hot Spot | 17 | Newly emerged |
| Intensifying Hot Spot | 9 | Actively worsening |

## Data

- **Source:** New Zealand Transport Agency (NZTA) Crash Analysis System (CAS), the national record of all traffic crashes reported to Police, available from 1 January 2000.
- **Extract:** Canterbury region only, 2000 to 2023, downloaded as GeoJSON from the NZTA open data portal.
- **Licence:** CC BY 4.0. The raw data is **not** committed to this repository; it should be downloaded from the source and attributed.
- **Working projection:** New Zealand Transverse Mercator 2000 (EPSG:2193).

## Method

The entire analysis is scripted in `arcpy` and runs top to bottom in the notebook at `notebooks/road_safety_hotspots_arcpy.ipynb` (ArcGIS Pro 3.5.3). The workflow:

1. **Import and clean.** GeoJSON imported with a Z-safe pattern (the source carries Z-aware geometry, which otherwise conflicts with the projected output coordinate system), filtered to Canterbury, and projected to NZTM 2000.
2. **Derived fields.** A severity weight (fatal, serious, minor, non-injury) for weighting the density surface, and a date field derived from the crash year for the space-time analysis.
3. **Kernel Density** (Spatial Analyst), severity-weighted, for a continuous overview surface.
4. **Optimized Hot Spot Analysis** (Getis-Ord Gi\*), which returns statistically significant hot and cold spots with confidence levels rather than a raw heatmap.
5. **Density-based clustering** (DBSCAN) to isolate discrete black-spots.
6. **Space-Time Cube and Emerging Hot Spot Analysis** (Space Time Pattern Mining), the centrepiece, classifying each location's trend across annual time steps.

### Data-quality decision

The most recent year in any CAS extract is materially incomplete because of reporting and processing lag, and non-injury crashes in particular can take months to appear. The download's final year held only 486 crashes against a baseline near 3,200 per year. Because an incomplete tail year collapses across every location at once, it would bias the emerging-hotspot classification toward "diminishing". The temporal analysis therefore **excludes the final incomplete year (2023)**, using 2000 to 2022. That year is retained in the density, hotspot and cluster steps, which are time-agnostic.

## Repository structure

```
canterbury-crash-hotspots/
├── README.md
├── notebooks/
│   └── road_safety_hotspots_arcpy.ipynb   # full reproducible arcpy workflow
├── docs/
│   └── canterbury-emerging-hotspots.png    # exported layout (the hero map)
└── data/
    └── (CAS data not committed; see Data section for the source)
```

## Reproducing the analysis

1. Download the Canterbury CAS extract as GeoJSON from the NZTA open data portal.
2. Open the notebook on the **ArcGIS Pro Python environment** (`arcgispro-py3` or a clone). It requires `arcpy`, the Spatial Analyst extension (for Kernel Density), and the Space Time Pattern Mining tools (core).
3. Update `PROJECT_DIR` and the input filename in the notebook, confirm the `CONFIG` values against the inspection cell, then run top to bottom.
4. Symbolise `crash_emerging_hotspots` in ArcGIS Pro on the pattern field, and export the layout.

## Limitations

- **Annual granularity.** Open CAS records the crash year only, with no time-of-day or day-of-week detail.
- **Location offset.** Public CAS coordinates are snapped or offset for privacy, so hotspot *areas* are reliable while exact positions are approximate.
- **Reported crashes only.** Under-reporting is greatest for minor and non-injury crashes.
- **Incomplete tail year** excluded from the temporal analysis, as described above.
- **Parameter choices** (cell sizes, search radii, cluster thresholds, cube intervals) are documented in the notebook and should be tuned to the question at hand.

## Tools

ArcGIS Pro 3.5.3, `arcpy`, Spatial Analyst, Space Time Pattern Mining, Python. Cartography and layout in ArcGIS Pro.

## Author and licence

**Melanie van Enter** — GIS and geospatial specialist, Canterbury, New Zealand.

Code in this repository is released under the MIT Licence. The underlying crash data is © NZTA, licensed CC BY 4.0.
