
# ayuquila-basin-geospatial-atlas

**Physical-geographical landscapes and environmental-risk screening atlas for the lower Ayuquila River basin, Jalisco, Mexico.**

Support tool for the **Intermunicipal Environmental Board “JIRA”** — *Junta Intermunicipal de Medio Ambiente para la Gestión Integral de la Cuenca Baja del Río Ayuquila*.

The workflow guides sanitation strategies, mitigates environmental risks, and supports ecological land-use planning using open global datasets in **Google Earth Engine** and **Python / geemap**.

### GeoNode-style composition map

*(Parteaguas, red vial, corrientes y cuerpos de agua, área de estudio, paisajes físico-geográficos, relieve sombreado.)*

![GeoNode-style composition](figures/15_geonode_style_map.png)

---

## ⚠️ Important caveat

All risk indices in this repository are **relative, screening-level indices** built from open global datasets. They are **not calibrated** engineering or hydrological models. Their purpose is to **prioritise areas for field verification** — not to provide final design or regulatory values.

---

## Repository structure

```
ayuquila-basin-geospatial-atlas/
├── README.md
├── LICENSE
├── .gitignore
├── ayuquila_basin.py                 # main Earth Engine + matplotlib workflow (add this file)
├── data/
│   ├── municipal_summary.csv         # mean indices per municipality
│   └── landscape_units_fallback.csv  # area per fallback landscape unit
└── figures/
    ├── 01_a_relief_elevation.png
    ├── 02_b_slope.png
    ├── 03_c_landforms_tpi.png
    ├── 04_d_mean_annual_rainfall.png
    ├── 05_e_mean_annual_temperature.png
    ├── 06_f_hydrology_hand_streams_water.png
    ├── 07_g_land_cover_esa_worldcover_2021.png
    ├── 08_h_vegetation_vigour_wet_season_2025.png
    ├── 09_i_soil_erosion_susceptibility.png
    ├── 10_j_flood_susceptibility.png
    ├── 11_k_wildfire_susceptibility.png
    ├── 12_l_sanitation_water_quality_pressure.png
    ├── 13_integrated_environmental_risk.png
    ├── 13_municipal_risk_heatmap.png
    ├── 14_priority_ranking.png
    ├── 14_proposed_ecological_planning_zones.png
    └── 15_geonode_style_map.png
```

---

## Data sources

| Layer | Source | Use |
|---|---|---|
| Elevation, slope, landforms | NASADEM | Panels A–C |
| Rainfall | CHIRPS (2015–2025) | Panel D |
| Temperature | ERA5-Land, lapse-corrected | Panel E |
| Hydrology | MERIT Hydro, JRC Global Surface Water | Panel F |
| Land cover | ESA WorldCover v200 (2021) | Panel G |
| Vegetation vigour | Sentinel-2 SR Harmonized | Panel H |
| Soils | SoilGrids (0–5 cm) | Panel I (erodibility proxy) |
| Fire history | MODIS MCD64A1 | Panel K |
| Municipal boundaries | FAO GAUL 2015 level 2 | AOI, zonal stats |
| Official reference layers | UNAM / Terra Digitalis GeoNode | GeoNode-style composition map |

---

## Risk indices

The integrated risk index is a weighted sum of four component indices:

```
Integrated = 0.30 · Erosion + 0.25 · Flood + 0.20 · Fire + 0.25 · Sanitation
```

- **Erosion** — RUSLE-like relative index (R·K·LS·C) from CHIRPS, SoilGrids, slope, and NDVI.
- **Flood** — weighted combination of HAND, slope, upstream drainage area, and JRC water occurrence.
- **Fire** — burn frequency (MODIS), slope, seasonal NDVI leaf-drop, and temperature, masked by fuel.
- **Sanitation pressure** — proximity to built-up areas and streams, plus cropland share (proxy for contamination pressure).

`HIGH_RISK_THRESHOLD = 0.60` is used to compute `high_risk_frac` (share of municipal territory above the threshold). Weights `W` are user-editable at the top of the script.

---

## Requirements

```bash
pip install earthengine-api geemap matplotlib numpy pandas pillow requests geopandas shapely osmnx
```

You need an Earth Engine account (`ee.Authenticate()` on first run) and, optionally, a Google Cloud project (`PROJECT = "my-gcp-project"`).

---

## Running the workflow

1. Clone the repo:

   ```bash
   git clone https://github.com/zafariabbas68/ayuquila-basin-geospatial-atlas.git
   cd ayuquila-basin-geospatial-atlas
   ```

2. Install the requirements above.

3. Open `ayuquila_basin.py` in JupyterLab (cells are marked `# %%`) and run sequentially.

4. Generated outputs:

   - `ayuquila_atlas.png` — 12-panel physical-geographical atlas (A–L).
   - `ayuquila_synthesis.png` — integrated risk, ecological zones, municipal heat map, priority ranking.
   - `municipal_summary.csv` — mean indices per municipality.
   - `landscape_units_fallback.csv` — area per fallback landscape unit.
   - Optional GeoTIFF exports (`EXPORT = True`) → Google Drive folder `JIRA_Ayuquila`.

---

## Key findings (2015–2025 climatology, 2025 Sentinel-2)

### Integrated risk by municipality

| Rank | Municipality | Integrated | High-risk share |
|---|---|---|---|
| 1 | El Grullo | 0.403 | 0.5 % |
| 2 | Tuxcacuesco | 0.391 | 0.5 % |
| 3 | Tonaya | 0.391 | 0.3 % |
| 4 | Ayutla | 0.382 | 0.6 % |
| 5 | Unión de Tula | 0.378 | 0.3 % |
| … | … | … | … |
| 18 | Cuatitlán | 0.232 | 0.3 % |

Full table: [`data/municipal_summary.csv`](data/municipal_summary.csv).

### Dominant drivers

- **Erosion** is the dominant driver across all municipalities (0.41–0.69), reflecting steep terrain and silt-rich soils. Highest in Tuxcacuesco (0.69), Ayutla (0.66), Ejutla (0.64).
- **Sanitation pressure** is second and highly concentrated: El Grullo (0.54), Tonaya (0.40), Ciudad Guzmán (0.40), Ayutla (0.37).
- **Wildfire** susceptibility is fairly uniform (0.22–0.47), peaking in Juchitlán (0.47), Ejutla (0.46), El Limón (0.45), Tonaya (0.44).
- **Flood** susceptibility is comparatively low (0.03–0.29), highest in El Grullo (0.29) and Ayutla (0.13).

### Priority municipalities for intervention

| Priority | Municipality | Rationale | Recommended action |
|---|---|---|---|
| 1 | **Tolimán** | Largest high-risk territory share (1.3 %) | Field-verify erosion hotspots; soil conservation |
| 2 | **Zapotitlán de Vadillo** | Second highest high-risk share (0.8 %) | Slope terracing and gully control |
| 3 | **El Grullo** | Highest integrated risk + highest sanitation pressure | Upgrade wastewater and solid-waste management |
| 4 | **Ayutla** | High erosion, sanitation pressure, high-risk share 0.6 % | Integrated catchment plan |
| 5 | **Tuxcacuesco** | Highest erosion (0.69) | Erosion verification and reforestation |
| 6 | **Tonaya** | High integrated risk, sanitation 0.40 | Sanitation and riparian buffers |
| 7 | **Juchitlán / Ejutla / El Limón** | Highest wildfire susceptibility | Fire-risk communication and fuel management |

---

## Proposed ecological zones

The synthesis map assigns each pixel to one of five management classes:

1. **Conservation of native vegetation / steep slopes** — tree/shrub cover, slope > 35°.
2. **Restoration & soil conservation** — erosion index > 0.6 outside native cover.
3. **Riparian & floodplain management** — HAND < 5 m within 400 m of a stream.
4. **Sustainable agriculture & livestock** — default productive land.
5. **Urban areas & sanitation infrastructure** — WorldCover built-up class.

---

## Figures

### A. Relief & elevation

![A. Relief & elevation](figures/01_a_relief_elevation.png)

### B. Slope

![B. Slope](figures/02_b_slope.png)

### C. Landforms (TPI)

![C. Landforms](figures/03_c_landforms_tpi.png)

### D. Mean annual rainfall

![D. Rainfall](figures/04_d_mean_annual_rainfall.png)

### E. Mean annual temperature

![E. Temperature](figures/05_e_mean_annual_temperature.png)

### F. Hydrology — HAND, streams, water bodies

![F. Hydrology](figures/06_f_hydrology_hand_streams_water.png)

### G. Land cover (ESA WorldCover 2021)

![G. Land cover](figures/07_g_land_cover_esa_worldcover_2021.png)

### H. Vegetation vigour, wet season 2025

![H. NDVI](figures/08_h_vegetation_vigour_wet_season_2025.png)

### I. Soil-erosion susceptibility

![I. Erosion](figures/09_i_soil_erosion_susceptibility.png)

### J. Flood susceptibility

![J. Flood](figures/10_j_flood_susceptibility.png)

### K. Wildfire susceptibility

![K. Fire](figures/11_k_wildfire_susceptibility.png)

### L. Sanitation / water-quality pressure

![L. Sanitation](figures/12_l_sanitation_water_quality_pressure.png)

### Integrated environmental risk

![Integrated risk](figures/13_integrated_environmental_risk.png)

### Mean relative risk by municipality (heat map)

![Municipal heat map](figures/13_municipal_risk_heatmap.png)

### Priority ranking for intervention

![Priority ranking](figures/14_priority_ranking.png)

### Proposed ecological planning zones

![Ecological zones](figures/14_proposed_ecological_planning_zones.png)

### GeoNode-style composition map

*(Parteaguas, red vial, corrientes y cuerpos de agua, área de estudio, paisajes físico-geográficos, relieve sombreado.)*

![GeoNode-style composition](figures/15_geonode_style_map.png)

---

## Figure index

| File | Panel |
|---|---|
| `figures/01_a_relief_elevation.png` | A. Relief & elevation |
| `figures/02_b_slope.png` | B. Slope |
| `figures/03_c_landforms_tpi.png` | C. Landforms (TPI) |
| `figures/04_d_mean_annual_rainfall.png` | D. Mean annual rainfall |
| `figures/05_e_mean_annual_temperature.png` | E. Mean annual temperature |
| `figures/06_f_hydrology_hand_streams_water.png` | F. Hydrology (HAND, streams, water) |
| `figures/07_g_land_cover_esa_worldcover_2021.png` | G. Land cover |
| `figures/08_h_vegetation_vigour_wet_season_2025.png` | H. Vegetation vigour (NDVI) |
| `figures/09_i_soil_erosion_susceptibility.png` | I. Soil-erosion susceptibility |
| `figures/10_j_flood_susceptibility.png` | J. Flood susceptibility |
| `figures/11_k_wildfire_susceptibility.png` | K. Wildfire susceptibility |
| `figures/12_l_sanitation_water_quality_pressure.png` | L. Sanitation / water-quality pressure |
| `figures/13_integrated_environmental_risk.png` | Integrated environmental risk |
| `figures/13_municipal_risk_heatmap.png` | Mean risk by municipality |
| `figures/14_priority_ranking.png` | Priority ranking for intervention |
| `figures/14_proposed_ecological_planning_zones.png` | Proposed ecological planning zones |
| `figures/15_geonode_style_map.png` | GeoNode-style composition map |

---

## Configuration

User-editable variables at the top of `ayuquila_basin.py`:

| Variable | Meaning |
|---|---|
| `PROJECT` | Google Cloud project (or `None`) |
| `YEAR` | Reference year for Sentinel-2 vegetation indices |
| `CLIM_START`, `CLIM_END` | Climatology window |
| `EXPORT` | `True` → export index rasters to Google Drive |
| `EXPORT_FOLDER` | Google Drive folder name |
| `LOCAL_FILES` | Paths to locally downloaded official layers |
| `W` | Risk weights (must sum to 1) |
| `HIGH_RISK_THRESHOLD` | Threshold for `high_risk_frac` |

---

## Recommended next steps

- Use the **high-risk mask** (integrated ≥ 0.60) as a screening layer for field campaigns, not as a regulatory boundary.
- Pair the **sanitation pressure** layer with actual discharge/sewage inventory to site treatment infrastructure.
- Run **validation sampling** in the top-ranked municipalities before adopting zones in land-use plans.
- Re-run the script annually with updated Sentinel-2, CHIRPS, and MODIS data to track trends.

---

## Citation

If you use this atlas or workflow, please cite:

> Zafari, G. A. (2025). *Physical-geographical landscapes and environmental-risk screening atlas for the lower Ayuquila River basin, Jalisco, Mexico*. GitHub: https://github.com/zafariabbas68/ayuquila-basin-geospatial-atlas

Prepared as a supporting technical note for the JIRA decision workflow.

---

## License

See [LICENSE](LICENSE) in the repository root.
