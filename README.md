# Southern Torrent Salamander Seep Detection

LiDAR-based terrain analysis and anomaly detection for mapping groundwater seeps — critical microhabitat for the Southern Torrent Salamander (*Rhyacotriton variegatus*) — using USGS 3DEP data and convolutional autoencoders.

Part of the [Pacific Northwest Conservation AI](https://github.com/Hestia-s-Creations/EEIS) project suite by Hestia's Creations.

## What It Does

Detects groundwater seepage zones in Pacific Northwest forests using high-resolution LiDAR terrain derivatives. Seeps are critical microhabitat for the Southern Torrent Salamander, a species of conservation concern, but are nearly invisible from satellite imagery alone. This project combines topographic analysis with machine learning anomaly detection to map probable seep locations at landscape scale.

### Key Capabilities

- **WhiteboxTools terrain analysis** - TWI, flow accumulation, topographic openness, depression breaching from 1m LiDAR DEMs
- **Multiscale Topographic Wetness Index** - novel discriminating feature at 1m, 5m, 10m, 30m scales
- **Convolutional autoencoder anomaly detection** - trained on "normal" (non-seep) terrain, seeps detected as reconstruction anomalies
- **Weak supervision via Snorkel** - programmatic labeling functions from TWI thresholds, slope criteria, expert rules
- **Sentinel-2 vegetation moisture** - NDWI seasonal composites as supplementary features
- **Probabilistic output maps** with confidence scores for field survey prioritization

## Target Metrics

| Metric | Target | Published Range |
|--------|--------|----------------|
| F1 Score | 65-75% | 60-80% (ephemeral stream detection) |
| Precision (high-conf) | > 80% | For targeted field validation |
| Omission Rate | < 40% | Concentrated in small/obscured seeps |

## Why This Is Hard

This is the most challenging of the four Pacific Northwest Conservation AI projects:
- **Dense vegetation** obscures ground returns in LiDAR
- **Ephemeral flow** — seeps vary seasonally, satellite timing rarely matches
- **Sub-meter features** strain 1m DEM resolution
- **No labeled training data** — no public seep location dataset exists
- **Ground truth requires field visits** with thermal cameras

## Data Sources (all free)

| Source | Data | Resolution |
|--------|------|------------|
| USGS 3DEP | LiDAR point clouds / DEMs | 1m |
| Oregon DOGAMI | State LiDAR holdings | 1m |
| Sentinel-2 | Vegetation moisture (NDWI) | 10-20m |
| USGS NHD | Stream network reference | Vector |
| PRISM | Precipitation/temperature | 4km |

## Project Structure

```
seep-detection/
+-- src/
|   +-- data/           # 3DEP LiDAR download, DEM processing, Sentinel-2 acquisition
|   +-- terrain/        # WhiteboxTools terrain indices (TWI, flow, openness, curvature)
|   +-- features/       # Multiscale feature engineering, Sentinel-2 moisture indices
|   +-- labeling/       # Weak supervision labeling functions (Snorkel-style)
|   +-- models/         # Convolutional autoencoder, anomaly scoring
|   +-- evaluation/     # Metrics, spatial assessment, field validation integration
|   +-- visualization/  # Probability maps, terrain overlays, survey prioritization
+-- notebooks/          # Exploratory analysis and prototyping
+-- tests/              # Unit and integration tests
+-- configs/            # Study area definitions, model hyperparameters, labeling rules
+-- data/               # Local data cache (gitignored)
+-- results/            # Model outputs and figures (gitignored)
```

## Quick Start

```bash
pip install -r requirements.txt

# Download 3DEP LiDAR DEM for study area
python -m src.data.download_lidar --bbox "-123.5,43.5,-122.5,44.5" --output data/dem/

# Compute terrain indices
python -m src.terrain.compute --dem data/dem/study_area.tif --output data/terrain/

# Generate multiscale features
python -m src.features.multiscale_twi --dem data/dem/study_area.tif --scales 1,5,10,30

# Generate weak labels via labeling functions
python -m src.labeling.generate --terrain data/terrain/ --output data/labels/

# Train autoencoder on non-seep terrain
python -m src.models.train_autoencoder --config configs/autoencoder_baseline.yaml

# Generate seep probability map
python -m src.models.predict --model results/models/autoencoder.pt --output results/seep_probability.tif
```

## Approach

1. **Terrain derivatives** from 1m LiDAR via WhiteboxTools: TWI, flow accumulation, topographic openness, plan/profile curvature, depression detection
2. **Multiscale TWI** at 1m/5m/10m/30m — seeps show characteristic high-TWI at fine scales that diminishes at coarser scales (novel discriminating feature)
3. **Weak supervision** — no labeled seep data exists, so use Snorkel-style labeling functions combining TWI thresholds, slope criteria, stream proximity, and geomorphic rules to generate noisy training labels
4. **Convolutional autoencoder** trained on confirmed non-seep terrain patches; seep locations produce high reconstruction error (anomaly detection)
5. **Sentinel-2 NDWI** seasonal composites as supplementary moisture signal
6. **Field validation integration** — output ranked candidate sites for targeted field surveys with thermal cameras

## Novel Contributions

- **Multiscale TWI as seep discriminator** — characterizing how wetness index changes across spatial scales
- **Autoencoder anomaly detection for seeps** — no existing work applies this to groundwater feature detection
- **Weak supervision for rare hydrological features** — programmatic labeling from geomorphic domain knowledge
- Potentially publishable at 65-75% F1

## Relationship to EEIS

Integrates with [EEIS](https://github.com/Hestia-s-Creations/EEIS) watershed monitoring. Seep locations inform EEIS disturbance impact assessment — logging near seeps has disproportionate impact on salamander habitat. LiDAR terrain data complements EEIS's Sentinel-2 spectral analysis. Seep probability maps are served through the shared PostGIS/Leaflet stack.

## References

- Original spec: "Pacific Northwest Conservation AI: Technical Specifications for Four Climate-Transferable Projects"
- WhiteboxTools: https://www.whiteboxgeo.com/
- USGS 3DEP: https://www.usgs.gov/3d-elevation-program
- Ratner et al. (2017) Snorkel: Rapid Training Data Creation with Weak Supervision

## License

Copyright 2025-2026 Hestia's Creations

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
