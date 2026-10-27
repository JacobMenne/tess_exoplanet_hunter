# TESS Exoplanet Hunter

An automated time-series photometry pipeline built in Python to detect, vet, and rank exoplanet transit candidates around M-dwarf stars using data from NASA's Transiting Exoplanet Survey Satellite (TESS) and the Mikulski Archive for Space Telescopes (MAST).

## Overview

When an orbiting planet crosses in front of its host star, the star's observed brightness drops by a fraction of a percent at regular intervals. While detecting periodic dips with Box Least Squares (BLS) algorithms is straightforward on clean data, real space-telescope photometry is dominated by instrumental artifacts, spacecraft data-downlink gaps, eclipsing binary stars, and stellar rotation waves.

This project implements a two-stage workflow:
1. **System Calibration (`01_calibration_pipeline.ipynb`):** Validates iterative multi-planet BLS masking and JWST atmospheric scoring on the known multi-planet red dwarf system L 98-59 (TIC 307210830).
2. **Automated Discovery Scanner (`02_discovery_scanner.ipynb`):** Queries the TESS Input Catalog (TIC) for unflagged M-dwarfs (cross-referenced against NASA's live Exoplanet Archive TOI table), stitches multi-sector light curves, runs automated false-positive vetting checks, and generates visual diagnostics for candidate verification.

## Automated Vetting Pipeline

To reduce false positives before manual review, `scan_star_for_transits()` enforces six sequential checks:
* **Contiguous Multi-Sector Stitching:** Stitches up to 3 sectors from the same observing season (within 120 days) to suppress single-sector noise spikes while avoiding multi-year periodogram aliasing.
* **Spacecraft Alias Avoidance:** Evaluates a 10,000-point period grid between 1.1 and 11.5 days to avoid the 13.7-day TESS lunar-synchronous orbit and downlink alias.
* **Transit Count & Coverage Verification:** Requires at least 3 distinct observed transits and a minimum of 15 in-transit data points so multi-sector data gaps cannot produce artificial Signal Detection Efficiency (SDE) spikes.
* **Physical Radius Cap:** Computes estimated planetary radius ($R_p = R_* \sqrt{\Delta F}$) and rejects eclipsing binary companions ($\ge 8.0\,R_\oplus$).
* **Out-of-Transit Baseline SNR:** Measures the ratio of transit depth to out-of-transit phase-folded scatter (`Dip SNR >= 2.5`) to reject fast-rotating starspot waves and noisy flat baselines.
* **Odd/Even Depth Symmetry & Pass Consistency:** Compares odd vs. even transit depths ($< 50\%$ divergence) to filter secondary eclipses, and verifies that at least 75% of individual orbit passes drop below the baseline.

Passing candidates are also scored using Kempton et al.'s (2018) **Transmission Spectroscopy Metric (TSM)** to estimate suitability for atmospheric characterization with the James Webb Space Telescope (JWST).

## Repository Structure

* `01_calibration_pipeline.ipynb` — Multi-planet BLS recovery and TSM validation on L 98-59 b, c, and d.
* `02_discovery_scanner.ipynb` — Batch discovery pipeline, persistent scan logger, and visual vetting suite.
* `ref_3panel_guide.png` & `ref_passes_guide.png` — Reference charts comparing true planetary transit signatures against common instrumental and astrophysical false positives.
* `scanned_stars_log.csv` — Persistent log tracking previously analyzed TIC IDs and their vetting outcomes across sessions.

## Installation and Usage

1. Clone the repository and set up a virtual environment:
   ```bash
   git clone git@github.com:<your-username>/tess_exoplanet_hunter.git
   cd tess_exoplanet_hunter
   python3 -m venv venv
   source venv/bin/activate
   pip install lightkurve astroquery pandas numpy matplotlib ipykernel