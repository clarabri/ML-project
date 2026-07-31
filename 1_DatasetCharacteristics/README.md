# Dataset Characteristics

**[Notebook](exploratory_data_analysis.ipynb)**

## Dataset Information

### Dataset Source
- **Dataset Link:** The data is stored privately, as the videos were recorded with the assurance that they would not be published.
- **Dataset Owner/Contact:** xx
- **Origin:** Self-recorded rowing-ergometer videos of several rowers, filmed from the side under consistent conditions (camera angle, lighting). Pose landmarks were extracted with the **MediaPipe Pose Landmarker**.

### Dataset Characteristics
- **Number of Observations:** 42,109 frames (rows) in the combined dataset `landmarks/normalized/landmarks_all_norm_10fps.csv`
  - **Temporal resolution:** 10 fps (downsampled from the original 30 fps) → 4,210.9 s ≈ **70 minutes** of movement data from **12 videos**
  - Segmented into **1,427 rowing strokes** (1,351 GOOD / 76 BAD), mean stroke rate ≈ **21 spm** (range 16.6 – 24.9 spm), see the note on `stroke_rate_spm` under [Data quality notes](#data-quality-notes)
  - **0 frames without a detected pose** — MediaPipe found a person in every frame
- **Number of Features:** 66 numeric landmark features per frame (33 landmarks × 2 coordinates), plus 3 label columns (`phase`, `GOOD`, `BAD`) → 69 columns in total

### Target Variable/Label

The dataset carries **two** labels, serving two different tasks:

**a) Stroke phase (frame level — main task in this notebook)**
- **Label Name:** `phase`
- **Label Type:** Classification (multiclass, per frame)
- **Label Description:** Which part of the rowing stroke the current frame belongs to. Derived from stroke segmentation at the true stroke boundaries.
- **Label Values:**
  - `0` = unknown — frames outside a recognized stroke
  - `1` = recovery — finish → catch
  - `2` = drive — catch → finish
- **Label Distribution:**

  | Phase | Frames | Share |
  |---|---:|---:|
  | 1 – recovery | 24,834 | 59.0 % |
  | 2 – drive | 16,672 | 39.6 % |
  | 0 – unknown | 603 | 1.4 % |

  Recovery being somewhat longer than the drive is biomechanically plausible.

**b) Technique quality (video/stroke level)**
- **Label Name:** `GOOD` / `BAD` (mutually exclusive one-hot pair)
- **Label Type:** Binary classification
- **Label Description:** Whether the stroke was rowed with correct or with deliberately faulty technique.
- **Label Distribution:** 39,990 GOOD frames vs. 2,119 BAD frames (95.0 % / 5.0 %) — i.e. 1,351 vs. 76 strokes. **Strongly imbalanced.**

### Feature Description

- **Feature Group (landmark coordinates, 66 features):** `<landmark>_x_norm` and `<landmark>_y_norm` for each of the 33 MediaPipe pose landmarks (nose, eyes, ears, mouth, shoulders, elbows, wrists, hand landmarks, hips, knees, ankles, heels, foot indices). Data type `float64`.
  - **Normalization:** body-relative — the hip midpoint is the origin, coordinates are scaled by the mean torso length. Values are therefore expressed in **torso lengths** relative to the average hip position of the respective video. This removes camera position and subject distance as confounding factors; only the movement remains.
  - **No z-coordinate** — the analysis is purely 2D (side view).
  - Typical ranges: shoulder-y ≈ −1 (roughly one torso length above the hips), hip-y ≈ 0, x-coordinates spread over roughly −1 … +0.6 depending on the landmark.
- **Feature (`phase`):** integer label, see above.
- **Features (`GOOD`, `BAD`):** integer 0/1 technique labels, see above.

### Data variants on disk

| Path | Content |
|---|---|
| [Data/OG_Videos/](Data/OG_Videos/) | original `.mov` recordings |
| [Data/skeleton_frames/](Data/skeleton_frames/) | rendered skeleton frames per video (`…-skeleton` raw, `…-skeleton-norm` normalized) |
| [Data/skeleton_videos/](Data/skeleton_videos/) | skeleton overlay videos |
| [landmarks/normalized/](landmarks/normalized/) | per-video `…-landmarks-raw.csv` / `…-landmarks-norm.csv` (30 fps) and `…_10fps.csv` variants, plus `…-strokes.csv` with the stroke boundaries |
| `landmarks/normalized/landmarks_all_norm_10fps.csv` | **combined dataset used for the EDA and for modelling** |
| [dataset_documentation.csv](dataset_documentation.csv) | per-video metadata: duration, fps, frame counts, frames without pose, stroke count, stroke rate |

## Exploratory Data Analysis

The exploratory data analysis is conducted in the [exploratory_data_analysis.ipynb](exploratory_data_analysis.ipynb) notebook, which includes:

- **Dataset overview** — per-video table (duration, fps, frames, strokes, stroke rate) and GOOD/BAD stroke counts
- **Missing value analysis** — **no missing values**; 0 columns with NaN, 0 frames without a detected pose
- **Feature distributions** — histograms of the key shoulder and hip landmarks
- **Phase distribution** — absolute frame counts per phase and hip-x distribution split by phase
- **Correlation analysis** — heatmap over key landmarks and `phase`
- **Data quality assessment / limitations**

### Key Findings

**Feature distributions.** Shoulder-y is narrow and peaked around ≈ −1: the shoulders sit about one torso length above the hips and barely move vertically. Shoulder-x and hip-x are wide, with the hips clearly **bimodal** — the two frequent positions are *catch* (hips forward) and *finish* (hips back). The normalization makes this structure visible because camera position no longer contributes any variance. Hip-y stays close to 0 with low spread, matching real rowing mechanics.

**Phase separability.** Recovery and drive occupy clearly distinguishable hip-x ranges. The `phase` label is therefore directly separable from the landmark data — a positive signal for the modelling step.

**Correlations.** Hip-x and shoulder-x correlate measurably with `phase`, i.e. the horizontal body swing genuinely carries the phase information. Between landmarks: `left_shoulder_x` ↔ `right_shoulder_x` ≈ 1.00 and `left_hip_x` ↔ `right_hip_x` ≈ 1.00 (left/right move in sync — strong redundancy), `shoulder_x` ↔ `hip_x` ≈ 0.95+ (upper body and hips swing as one unit). The x/y cross-correlations are weaker than in the raw data, as intended by the normalization.

<a id="data-quality-notes"></a>
### Data quality notes on `dataset_documentation.csv`

Two columns of the metadata file are inconsistent with the actual landmark files. The landmark CSVs are authoritative — their 10 fps frame counts sum to exactly 42,109, matching the combined dataset.

- **`stroke_rate_spm` is inflated.** For 10 of the 12 videos the value is exactly **3× the true stroke rate** (the 30 fps / 10 fps factor was applied by mistake), for the two `MIRRORED` videos ≈ 2.1×. Recomputed from stroke count and true duration, the rates are **16.6 – 24.9 spm (mean ≈ 20.9, weighted 20.3)** instead of the listed 49.7 – 74.8 spm. The corrected range is the physiologically plausible one for steady-state rowing.
- **`duration_s` / `reduced_frames` are wrong for the two `MIRRORED` videos.** `man-GOOD-slow-MIRRORED-40min` lists 2,000.2 s / 1,900 reduced frames but actually contributes 23,970 frames ≈ 2,397 s (≈ 40 min); `man-GOOD-slow-MIRRORED-7.5min` lists 374.7 s but contributes 4,463 frames ≈ 446.3 s. Both `duration_s` values were derived from `original_frames / 30`, which does not hold after mirroring.

## Dataset Limitations

**Advantages of the controlled setup**
- Consistent recording conditions (camera angle, lighting)
- Body-relative normalization removes camera position as a confounding factor
- Stroke segmentation at true stroke boundaries yields clean per-stroke sequences
- Complete pose detection — no gaps to interpolate

**Limitations**
- **Class imbalance:** only 76 BAD vs. 1,351 GOOD strokes (5.0 % of frames). This affects threshold selection and evaluation metrics; precision on the BAD class may be unstable.
- **Small subject pool:** few recorded rowers — risk of overfitting to their individual movement patterns.
- **Limited error types:** the BAD class covers only a narrow range of technique faults.
- **Dominant single video:** `man-GOOD-slow-MIRRORED-40min` contributes 23,970 of the 42,109 frames (57 %), so one rower dominates the GOOD class.
- **Narrow stroke-rate range:** all recordings sit between ≈ 17 and 25 spm; higher race rates are not represented, even though the file names distinguish "fast" and "slow".
- **Redundant features:** left/right landmark pairs are almost perfectly correlated (≈ 1.00) — the effective dimensionality is well below 66.
- **2D only:** no depth information; movement components perpendicular to the camera plane are not captured.

**Next steps**
- Record more rowers to improve generalization
- Add more BAD strokes and further error types to balance the technique classes
- Fix `stroke_rate_spm` and the `MIRRORED` durations in `dataset_documentation.csv`
