# EQTyCatRisk

EQTyCatRisk is a local web application for identifying historical earthquake–typhoon event pairs in Japan and exploring residential loss calculations for two Chiba case studies.

## Features and limits

- **Coupled event detection (`/events`):** reads the packaged NIED earthquake and JMA typhoon catalogues and identifies simultaneous pairs. Earthquake times are treated as JST; typhoon UTC times are converted once to JST. The current loader screens earthquakes at Mj 5.0 or above and track records at 34 kt or above. The time separation must be at most 72 hours, and the epicentral distance must not exceed the original 30 kt long radius (R30). The current implementation keeps one earthquake per typhoon, prioritizing the larger magnitude; it does not guarantee a globally one-to-one matching on the earthquake side.
- **Coupled risk assessment (`/risk`):** displays the 2004 Ma-on and 2005 Banyan Chiba cases, gridded PGA and wind, and separate earthquake-only, typhoon-only and coupled residential-loss distributions. It uses packaged hazard grids, exposures and vulnerability models. The interface provides preset case calculations; an arbitrary historical event does not automatically have the hazard and exposure inputs required for loss calculation.
- The 2005 web calculation uses common random numbers (CRN) to pair earthquake and coupled realizations; the 2004 calculation uses the legacy unpaired v12 stream. The default is 5,000 realizations, seed 20260706, with at most 5,000 exposure records sampled using the Gaussian copula. This is not the same as sampling every exposure record. The Python CLI defaults to the legacy unpaired method unless `--sampling-method crn` is specified.

## Requirements

- Node.js 20 or later and npm.
- Python 3.10 or later, with `numpy`, `pandas`, `scipy`, and `matplotlib`.
- Internet access for external map tiles. Only use trusted model pickle files.

## Run locally

Open a terminal in the extracted repository directory (the directory containing `package.json`):

```powershell
npm ci
python -m pip install -r requirements.txt
npm run dev -- --host 127.0.0.1 --port 5173
```

Open [event detection](http://127.0.0.1:5173/events) or [risk assessment](http://127.0.0.1:5173/risk). If the port is occupied, use the address reported by Vite. The server chooses `D:\Anaconda\python.exe` on Windows if present, otherwise `python`; to specify another interpreter, set `$env:EQTY_PYTHON` before starting the server.

`npm run build` checks and builds the frontend. The calculation and recognition API is provided by the Vite development server: `npm run preview` or GitHub Pages can display packaged results but cannot execute those Python endpoints. The development API is intended for a trusted local environment and is not a hardened public service.

## Inputs and calculations

The recognition implementation and Japanese catalogues are under `data/Sample data 1/`. The NIED/JMA source files retain their original Japanese-language data and encodings to preserve the input records. `Recognize` regenerates the coupling CSVs and frontend JSON. Risk inputs, case definitions and the model are under `data/Sample data 2/Preprocessing/`; `MC_Convergence_Study/run_mc_study.py` provides helper functions used by 2005 CRN mode and must remain in place.

For example, from the repository root:

```powershell
python "data/Sample data 2/Preprocessing/run_v12_loss_calculation.py" --case 2005 --samples 5000 --sampling-method crn
python "data/Sample data 2/Preprocessing/run_v12_loss_calculation.py" --case 2004 --samples 5000
```

CLI results are written to `data/Sample data 2/risk-loss/`. The web API additionally updates `public/data/risk-loss/`. Loss values in the CSV column `loss_oku` are in 100 million JPY; conversion to million USD uses `loss_oku * 100 / exchange_rate`, with the rate in `cases.json`. A replacement case requires compatible precomputed PGA, wind, exposure and vulnerability inputs; it is not generated from an event identifier alone.

The exposure merge does not cover every positive exposure record in the supplied data. Interpret results within the matched coverage. The scientific model is not changed to force coupled loss above earthquake-only loss.

## Package contents and license

This release retains the operational source, reference outputs, both case input grids, exposure, model, original catalogues, and CRN helper. It excludes reproducible dependencies, local backups, research-study outputs, and build products. The housing exposure data are stored once as a losslessly compressed CSV (`data/Sample data 2/Preprocessing/inputs/Chiba_exposure.csv.gz`). The calculation program reads the compressed file directly. The original uncompressed input is approximately 53 MB. See `LICENSE` and verify source-data rights before redistributing.

---

## Details of Exposure Matching and Vulnerability Surface Modeling

### 1. Exposure

#### E — Exposure

The `Exposure/` directory provides residential exposure data for Chiba Prefecture.

The exposure data are located at: `EQTyCatRisk-main\\data\\Exposure`

For every prefecture, the `Chiba Grid ID` file contains the identifier and coordinates of each 1 km × 1 km grid cell. The aggregated residential value for each grid identifier and structural class (`vulner_ID`) is in `data/Sample data 2/Preprocessing/inputs/Chiba_exposure.csv.gz`; this replaces the duplicate `Chiba Grid Value.csv` file. Exposure values are expressed in **Japanese yen (JPY)**.

#### Residential structure classification

The redundant `vulnerability_ID` field is intentionally omitted from this reference table.

| vulner_ID | structural_type | construction_year | storey | occupancy |
|---:|---|---|---|---|
| 1 | Wood | 1925_1990 | 2_3 | resident |
| 4 | Wood | 1925_1990 | 1 | resident |
| 7 | Wood | 1925_1990 | 4_7 | resident |
| 10 | Wood | 1925_1990 | 8_14 | resident |
| 13 | Wood | 1925_1990 | 15_ | resident |
| 16 | Wood | 1991_1999 | 2_3 | resident |
| 19 | Wood | 1991_1999 | 1 | resident |
| 22 | Wood | 1991_1999 | 4_7 | resident |
| 25 | Wood | 1991_1999 | 8_14 | resident |
| 28 | Wood | 1991_1999 | 15_ | resident |
| 31 | Wood | 2000_2010 | 2_3 | resident |
| 34 | Wood | 2000_2010 | 1 | resident |
| 37 | Wood | 2000_2010 | 4_7 | resident |
| 40 | Wood | 2000_2010 | 8_14 | resident |
| 43 | R_C | 2000_2010 | 15_ | resident |
| 46 | R_C | 2011_ | 2_3 | resident |
| 49 | R_C | 2011_ | 1 | resident |
| 52 | R_C | 2011_ | 4_7 | resident |
| 55 | R_C | 2011_ | 8_14 | resident |
| 58 | R_C | 2011_ | 15_ | resident |
| 61 | R_C | 1925_1990 | 2_3 | resident |
| 64 | R_C | 1925_1990 | 1 | resident |
| 67 | R_C | 1925_1990 | 4_7 | resident |
| 70 | R_C | 1925_1990 | 8_14 | resident |
| 73 | R_C | 1925_1990 | 15_ | resident |
| 76 | R_C | 1991_1999 | 2_3 | resident |
| 79 | R_C | 1991_1999 | 1 | resident |
| 82 | R_C | 1991_1999 | 4_7 | resident |
| 85 | R_C | 1991_1999 | 8_14 | resident |
| 88 | R_C | 1991_1999 | 15_ | resident |
| 91 | R_C | 2000_2010 | 2_3 | resident |
| 94 | R_C | 2000_2010 | 1 | resident |
| 97 | R_C | 2000_2010 | 4_7 | resident |
| 100 | R_C | 2000_2010 | 8_14 | resident |
| 103 | Steel | 2000_2010 | 15_ | resident |
| 106 | Steel | 2011_ | 2_3 | resident |
| 109 | Steel | 2011_ | 1 | resident |
| 112 | Steel | 2011_ | 4_7 | resident |
| 115 | Steel | 2011_ | 8_14 | resident |
| 118 | Steel | 2011_ | 15_ | resident |
| 121 | Steel | 1925_1990 | 2_3 | resident |
| 124 | Steel | 1925_1990 | 1 | resident |
| 127 | Steel | 1925_1990 | 4_7 | resident |
| 130 | Steel | 1925_1990 | 8_14 | resident |
| 133 | Steel | 1925_1990 | 15_ | resident |
| 136 | Steel | 1991_1999 | 2_3 | resident |
| 139 | Steel | 1991_1999 | 1 | resident |
| 142 | Steel | 1991_1999 | 4_7 | resident |
| 145 | Steel | 1991_1999 | 8_14 | resident |
| 148 | Steel | 1991_1999 | 15_ | resident |
| 151 | Steel | 2000_2010 | 2_3 | resident |
| 154 | Steel | 2000_2010 | 1 | resident |
| 157 | Steel | 2000_2010 | 4_7 | resident |
| 160 | Steel | 2000_2010 | 8_14 | resident |
| 163 | Masonry | 2000_2010 | 15_ | resident |
| 166 | Masonry | 2011_ | 2_3 | resident |
| 169 | Masonry | 2011_ | 1 | resident |
| 172 | Masonry | 2011_ | 4_7 | resident |
| 175 | Masonry | 2011_ | 8_14 | resident |
| 178 | Masonry | 2011_ | 15_ | resident |
| 181 | Masonry | 1925_1990 | 2_3 | resident |
| 184 | Masonry | 1925_1990 | 1 | resident |
| 187 | Masonry | 1925_1990 | 4_7 | resident |
| 190 | Masonry | 1925_1990 | 8_14 | resident |
| 193 | Masonry | 1925_1990 | 15_ | resident |
| 196 | Masonry | 1991_1999 | 2_3 | resident |
| 199 | Masonry | 1991_1999 | 1 | resident |
| 202 | Masonry | 1991_1999 | 4_7 | resident |
| 205 | Masonry | 1991_1999 | 8_14 | resident |
| 208 | Masonry | 1991_1999 | 15_ | resident |
| 211 | Masonry | 2000_2010 | 2_3 | resident |
| 214 | Masonry | 2000_2010 | 1 | resident |
| 217 | Masonry | 2000_2010 | 4_7 | resident |
| 220 | Masonry | 2000_2010 | 8_14 | resident |
| 283 | Wood | 2000_2010 | 15_ | resident |
| 286 | Wood | 2011_ | 2_3 | resident |
| 289 | Wood | 2011_ | 1 | resident |
| 292 | Wood | 2011_ | 4_7 | resident |
| 295 | Wood | 2011_ | 8_14 | resident |
| 298 | Wood | 2011_ | 15_ | resident |

### 2. Vulnerability

The vulnerability surfaces are developed using OpenSees. The publicly available Dynamic Analysis of 2-Story Moment Frame example [1] is adopted as the reference model. This example represents a two-story, single-bay steel moment-resisting frame. By modifying the material definitions in OpenSees, separate models are constructed for wood, steel, reinforced concrete (RC), and masonry structures.

The seismic input is the ground-motion record obtained at K-NET station MYG004 during the 2011 Great East Japan Earthquake. Wind inputs are based on the wind spectrum recommended by the Architectural Institute of Japan (AIJ).

The ground-motion record is amplitude-scaled to peak ground acceleration levels of 0, 0.5, 1.0, 1.5, and 2.0 g. Wind inputs are specified at mean wind speeds of 0, 15, 30, 45, and 60 m/s.

The vulnerability surfaces are constructed from the structural responses under different combinations of earthquake and wind loading. The resulting surfaces are illustrated in [`data/Vulnerability/data.png`](data/Vulnerability/data.png).

## Reference

[1] OpenSeesWiki, Dynamic Analysis of 2-Story Moment Frame.
<https://opensees.berkeley.edu/wiki/index.php?title=Dynamic_Analysis_of_2-Story_Moment_Frame>

For the numerical representation of exposure and vulnerability, refer to [`data/Sample data 2/Preprocessing/run_v12_loss_calculation.py`](data/Sample%20data%202/Preprocessing/run_v12_loss_calculation.py). This script documents the exposure-value mapping, vulnerability parameters, and numerical outputs used in the loss-calculation workflow.

The script can be used as a reference when reproducing or extending the quantified exposure and vulnerability results for alternative coupled events.
