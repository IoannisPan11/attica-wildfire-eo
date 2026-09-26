# Burn Severity Assessment of the 2024 Varnavas Wildfire, Attica, Greece

An end-to-end Earth Observation workflow for assessing burn severity following the August 2024 Varnavas wildfire in northeastern Attica, Greece.

The project uses multi-temporal Sentinel-2 Level-2A imagery to quantify vegetation change, derive burn-severity indicators, map priority areas, and produce decision-ready geospatial outputs for post-fire field assessment.

## Decision Question

**Which areas affected by the August 2024 Varnavas wildfire experienced the greatest vegetation damage and should be prioritised for post-fire field assessment?**

The intended users are forest-management and environmental-restoration authorities planning field inspections and recovery actions.

## Data

- **Mission:** Sentinel-2
- **Product:** Level-2A surface reflectance
- **Backend:** Microsoft Planetary Computer STAC API
- **Study period:** June–October 2024
- **Fire date:** 11 August 2024
- **AOI:** northeastern Attica, Greece
- **Selected acquisitions:** 8 dates — 4 pre-fire and 4 post-fire
- **Source items:** 16 STAC items across MGRS tiles `34SGH` and `35SKC`
- **Bands:** B02, B03, B04, B08, B12 and SCL
- **Analysis grid:** 20 m, EPSG:32634

## Methodology

The workflow is implemented in a single reproducible Jupyter notebook:

1. Query Sentinel-2 L2A imagery through the Planetary Computer STAC catalogue.
2. Select 8 low-cloud acquisitions surrounding the wildfire event.
3. Load real Sentinel-2 pixel data and align both MGRS tiles to a common 20 m EPSG:32634 grid.
4. Mosaic same-date tile pairs into one acquisition per date.
5. Convert stored digital numbers to BOA surface reflectance using the Sentinel-2 quantification value and radiometric offset applicable to the selected processing baselines.
6. Apply pixel-level quality masking using the Sentinel-2 Scene Classification Layer (SCL).
7. Compute NDVI and NBR for each acquisition.
8. Build four-date pre-fire and post-fire median NBR composites.
9. Compute dNBR and classify burn severity using the Key & Benson (2006) FIREMON reference ranges.
10. Vectorise moderate-high and high-severity regions and remove fragments smaller than 0.5 ha.
11. Rank the resulting areas using a transparent severity-weighted extent rule:

   `priority_score = area_ha × mean_dnbr`

Negative retrieved reflectance values are retained in the calibrated reflectance data. For normalized-difference indices, pixels with negative input reflectance or a non-positive band sum are excluded to avoid unstable ratios without modifying the underlying reflectance values.

## Key Results

The final analysis mapped **6,775.9 ha** of moderate-high or high-severity priority area across **201 contiguous polygons** of at least 0.5 ha.

The largest priority region covers **5,663.4 ha**, has a mean dNBR of **0.630**, and is centred at approximately **38.1482° N, 23.9201° E**.

The temporal signal also shows a clear post-fire change. AOI-mean NBR decreases from **0.127** on 7 August, immediately before the fire, to **0.016** on 17 August and **0.001** on 22 August.

The ranking is intended as a field-prioritisation aid rather than a final ecological severity determination.

## Outputs

Final outputs are included in `outputs/` and are also regenerated when the notebook is executed.

### Figures

- `rgb_false_color_pre_post.png` — pre-/post-fire true-colour and SWIR2/NIR/Red false-colour imagery
- `ndvi_nbr_timeseries.png` — AOI-mean NDVI and NBR across all 8 acquisitions
- `dnbr_severity_map.png` — continuous dNBR and classified burn-severity maps

### Rasters

- `dnbr.tif` — continuous dNBR raster
- `burn_severity_class.tif` — classified burn-severity raster

Both rasters are exported in EPSG:32634 as Cloud-Optimized GeoTIFFs when supported by the installed GDAL build, with a tiled GeoTIFF fallback.

### Vectors

- `burned_areas_moderate_high_severity.geojson` — moderate-high/high-severity polygons
- `priority_areas_ranked.geojson` — ranked priority polygons with decision attributes
- `decision_summary.txt` — plain-language decision summary

GeoJSON outputs are provided in WGS84 (EPSG:4326) for straightforward use in GIS and web-mapping applications.

## Repository Structure

```text
attica-wildfire-eo/
├── notebooks/
│   └── attica_wildfire_analysis.ipynb
├── outputs/
│   ├── figures/
│   ├── rasters/
│   └── vectors/
├── pyproject.toml
├── uv.lock
└── README.md
```

## Setup

The project uses `uv` for dependency and environment management.

```bash
uv sync
uv run python -m ipykernel install --user \
  --name attica-wildfire-eo \
  --display-name "Python (Attica Wildfire EO)"
```

Open `notebooks/attica_wildfire_analysis.ipynb`, select the `Python (Attica Wildfire EO)` kernel, and run the notebook from top to bottom.

The workflow retrieves Sentinel-2 imagery remotely from Microsoft Planetary Computer, so execution time depends partly on network conditions.

## Limitations

dNBR is a spectral proxy for burn severity rather than direct ecological ground truth. The Key & Benson (2006) ranges are used as a reference classification and are not locally calibrated for Mediterranean vegetation in Attica.

The post-fire composite spans approximately six weeks, so genuine vegetation change during that period may influence the resulting dNBR. The AOI also extends slightly into UTM zone 35N, while a single EPSG:32634 analysis grid is used for consistency.

Independent fire-perimeter data, field observations, or additional sensors such as Sentinel-1 could be used in future work to validate and extend the analysis.