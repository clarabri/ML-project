# Dataset Characteristics

**[Notebook](exploratory_data_analysis.ipynb)**

## Dataset Information

### Dataset Source
- **Dataset Link:** The data is stored privately, as the videos were recorded with the assurance that they would not be published.
- **Dataset Owner/Contact:** Clara B.
- **Origin:** Self-recorded rowing-ergometer videos of 4 rowers (`cla`, `man`, `mar`, `sim`), filmed from the side under consistent conditions (camera angle, lighting). Ten videos are 30 fps, `man-GOOD-slow-MIRRORED-7.5min` is 25 fps and was filmed from the opposite side (x-coordinates are mirrored during preprocessing). Pose landmarks were extracted with the **MediaPipe Pose Landmarker**.

### Dataset Characteristics
- **Number of Observations:** 18,139 frames (rows), from **11 videos**
  - **Temporal resolution:** 10 fps (downsampled from the source material; 52,249 original frames) → 1,816.9 s ≈ **30 minutes** of movement data
  - Segmented into **657 rowing strokes** (578 GOOD / 79 BAD), mean stroke rate ≈ **22 spm** (range 18.4 – 26.0, weighted 21.7). The `-fast` videos average 24.8 spm against 20.8 for `-slow`
  - **0 frames without a detected pose** — MediaPipe found a person in every frame
  - No single video dominates: the largest (`man-GOOD-slow-MIRRORED-7.5min`) contributes 24.6 % of the frames, the remaining ten between 1.9 % and 13.6 %
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
  | 1 – recovery | 10,537 | 58.1 % |
  | 2 – drive | 7,271 | 40.1 % |
  | 0 – unknown | 331 | 1.8 % |

  Recovery being somewhat longer than the drive is biomechanically plausible.

**b) Technique quality (video/stroke level)**
- **Label Name:** `GOOD` / `BAD` (mutually exclusive one-hot pair)
- **Label Type:** Binary classification
- **Label Description:** Whether the stroke was rowed with correct or with deliberately faulty technique.
- **Label Distribution:** 16,020 GOOD frames vs. 2,119 BAD frames (88.3 % / 11.7 %) — i.e. 578 vs. 79 strokes. 

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
| [dataset_documentation.csv](dataset_documentation.csv) | per-video metadata: duration, fps, frame counts, frames without pose, `n_peaks` (detected finishes), `n_strokes` (segmented), stroke rate |

## Exploratory Data Analysis

The exploratory data analysis is conducted in the [exploratory_data_analysis.ipynb](exploratory_data_analysis.ipynb) notebook, which includes:

- **Dataset overview** — per-video table (duration, fps, frames, strokes, stroke rate) and GOOD/BAD stroke counts
- **Missing value analysis** — no missing *feature* values (0 columns with NaN, 0 frames without a detected pose), but a per-video breakdown of missing *labels*: unsegmented strokes and `phase = 0` frames
- **Feature distributions** — histograms of the key shoulder and hip landmarks
- **Phase distribution** — absolute frame counts per phase and hip-x distribution split by phase
- **Correlation analysis** — heatmap over key landmarks and `phase`
- **Data quality assessment / limitations**

### Key Findings

**Feature distributions.** Shoulder-y is narrow and peaked around ≈ −1: the shoulders sit about one torso length above the hips and barely move vertically. Shoulder-x and hip-x are wide, with the hips clearly **bimodal** — the two frequent positions are *catch* (hips forward) and *finish* (hips back). The normalization makes this structure visible because camera position no longer contributes any variance. Hip-y stays close to 0 with low spread, matching real rowing mechanics.

**Phase separability.** Recovery and drive occupy clearly distinguishable hip-x ranges. The `phase` label is therefore directly separable from the landmark data — a positive signal for the modelling step.

**Correlations.** Between landmarks the structure is as expected: `left_shoulder_x` ↔ `right_shoulder_x` = 0.91 and `left_hip_x` ↔ `right_hip_x` = 0.91 (left/right move largely in sync — substantial redundancy), `left_shoulder_x` ↔ `left_hip_x` = 0.98 (upper body and hips swing as one unit). The x/y cross-correlations are weaker than in the raw data, as intended by the normalization.

Linear correlations **with `phase`, however, are weak throughout** — the largest is `left_wrist_y` at −0.40, followed by `right_shoulder_y` (−0.23); hip-x and shoulder-x reach only −0.10 to −0.07. Two conclusions follow:

- The **hands carry more phase signal than the torso**, which fits rowing: the seat moves during both phases, the hands reverse direction at catch and finish.
- Pearson correlation is the wrong instrument here. Recovery and drive traverse the *same* positional range in opposite directions, so a linear coefficient against an ordinal label largely cancels out. The clear separation in the phase-wise hip-x histogram shows the information is present — it lies in the **direction of movement (velocity), not in position**. Derived features (differences between consecutive frames) or sequence models are therefore likely to outperform per-frame position features.

<a id="data-quality-notes"></a>
### Data quality

- **No missing values:** 0 columns with NaN, 0 frames without a detected pose.
- **Consistent:** the 11 per-video CSVs sum to exactly 18,139 rows, matching the combined file, and `dataset_documentation.csv` agrees with the per-video stroke files.
- **Labels:** 668 detected finish peaks yield 657 strokes. The 11 losses are structural — a stroke spans two consecutive finishes, so each video's last peak opens none. 331 frames (1.8 %) carry `phase = 0`, all at the video edges.

## Dataset Limitations

**Advantages of the controlled setup**
- Consistent recording conditions (camera angle, lighting)
- Body-relative normalization removes camera position as a confounding factor
- Complete pose detection — no gaps to interpolate
- Balanced video mix — no recording dominates the dataset
- Clean stroke segmentation at true stroke boundaries, with 98.2 % of frames labelled

**Limitations**
- **Class imbalance:** 79 BAD vs. 578 GOOD strokes (11.7 % of frames). This affects threshold selection and evaluation metrics; precision on the BAD class may be unstable.
- **Small subject pool:** only 4 rowers — risk of overfitting to their individual movement patterns. Both BAD recordings come from a single rower (`cla`), so "bad technique" and "this person" are confounded.
- **Limited error types:** the BAD class covers only a narrow range of technique faults, from two recordings.
- **Narrow stroke-rate range:** all recordings sit between 18.4 and 26.0 spm; higher race rates are not represented.
- **Missing labels, not missing features:** 331 frames (1.8 %) carry `phase = 0`, all of them at the video edges. 17,808 frames are usable for supervised phase classification, and the discarded remainder carries no systematic bias.
- **Small overall size:** ≈ 30 minutes of material / 657 strokes is little for sequence models — augmentation or transfer learning may be needed.
- **Redundant features:** left/right landmark pairs are highly correlated (≈ 0.91) — the effective dimensionality is well below 66.
- **2D only:** no depth information; movement components perpendicular to the camera plane are not captured.

**Next steps**
- Record more rowers to improve generalization
- Add more BAD strokes and further error types, ideally from several rowers, to break the subject/class confound
