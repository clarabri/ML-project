# Baseline Model

**[Notebook](baseline_model.ipynb)**

## Baseline Model Results

### Model Selection
- **Baseline Model Type:** `RandomForestClassifier` (100 trees, `class_weight="balanced"`), benchmarked against a `DummyClassifier` (`most_frequent`) as the floor.
- **Rationale:** the EDA found only weak *linear* correlation between landmark positions and the labels, and the coordinates are strongly collinear (left/right shoulder x ≈ 0.91) — both are handled natively by trees. `class_weight="balanced"` counters the 88/12 imbalance; no scaling needed.

### Task and Data
- **Task:** per-frame binary classification of rowing technique, `GOOD` (1) vs. `BAD` (0)
- **Features:** the 66 normalized landmark coordinates (33 landmarks × x, y). Normalized rather than raw, so the model cannot key on camera position or subject distance.
- **Samples:** 17,808 frames in 657 strokes, 88.5 % GOOD / 11.5 % BAD. The 331 frames outside any stroke (`phase = 0`) are dropped.

### Model Performance

| Model | Accuracy | ROC-AUC | Macro F1 | Recall (BAD) |
|-------|---------|---------|----------|--------------|
| DummyClassifier | 0.836 | 0.500 | 0.455 | 0.00 |
| Random Forest | 0.981 | 0.998 | 0.965 | 0.91 |

- **Cross-Validation:** 5-fold `GroupKFold` by stroke — accuracy 0.983 ± 0.003, macro F1 0.958 ± 0.006, ROC-AUC 0.997 ± 0.001
- **Leave-one-video-out:** GOOD videos 86.1 % of frames correct, **BAD videos 5.4 %**

> ⚠️ **These numbers do not mean the baseline works** — see [The Core Problem](#the-core-problem) below.

### Evaluation Methodology
- **Data Split:** 80 / 20 via `GroupShuffleSplit` grouped by stroke. A random shuffle would leak: a stroke spans ~27 correlated frames, so neighbouring frames of one stroke would land on both sides. Groups use the real boundaries from the per-video `…-strokes.csv`; an assertion verifies no stroke crosses the split.
- **Metrics:** accuracy (misleading under imbalance, shown for completeness), macro F1, ROC-AUC, recall on BAD, confusion matrix.

<a id="the-core-problem"></a>
### The Core Problem: the model learns people, not rowing

With 0.981 accuracy the Random Forest looks like a solved problem. It never learned what bad technique looks like — it learned **who is in the frame**.

**1. The features are a fingerprint.** Trained on the exact same 66 columns, the same forest predicts:

| Prediction target | Accuracy | Chance |
|-------------------|---------:|-------:|
| Which of the 4 rowers | **99.6 %** | 25 % |
| Which of the 11 recordings | **96.6 %** | 9 % |

Normalization removes camera position and distance, but not body proportions or personal movement habits.

**2. That shortcut is enough to win.** All 2,052 BAD frames come from `cla-BAD` and `cla-BAD-2`, and strokes of those two recordings sit on both sides of the split. So a single rule — *looks like those two → BAD* — already gives 0.98 accuracy, with no notion of technique. Any classifier would find it: it is the easiest signal and matches the label perfectly.

**3. Remove the shortcut and nothing remains.** Shown a BAD recording it never trained on, the model marks only **~5 % of its frames as BAD**. Held-out `cla-GOOD-slow-2` shows the same from the other side: it is classified almost entirely as BAD, because `cla` is three times more often BAD than average among the remaining training frames. The model decides by person, not by technique.

**Why cross-validation misses it.** CV only measures generalization to whatever you hold out. Grouping by stroke asks *"I know this recording — can I classify more strokes from it?"* (yes, 98 %), never *"here is a new rower — is the technique faulty?"*. The ±0.003 across folds is not robustness: all five share the same shortcut, and a consistent flaw reads like stability.

This is a property of the **data**, not the model. With BAD examples from one rower, "faulty technique" and "this person" are mathematically inseparable — an LSTM or CNN would change nothing.

### Metric Practical Relevance

The system should tell a rower when a stroke is technically wrong. Missing a bad stroke gives false reassurance, so **recall on BAD is the metric that matters**; a false alarm merely costs trust. Accuracy is near-useless here — predicting GOOD for everything already yields 0.836.

The honest figure for this baseline is therefore the leave-one-video-out result (**5.4 % of BAD frames detected**), not the 0.981 from the random split. Future models must be judged on held-out *recordings*, not held-out strokes.

## Next Steps

- **Collect BAD recordings from more rowers** — the prerequisite for validating any model honestly
- Report leave-one-video-out (or leave-one-subject-out) alongside the random split
- Add temporal / delta features — the movement signal lies in the direction of motion, not in absolute position
- Aggregate features per stroke instead of classifying single frames
- Try LSTM or 1D-CNN to model the stroke as a sequence

This baseline serves as the reference point for the [Model Definition and Evaluation](../3_Model/README.md) phase.
