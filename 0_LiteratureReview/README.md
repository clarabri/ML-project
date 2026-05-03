# Literature Review

## Overview
Approaches and solutions that have been applied in similar projects.

**Summary of Each Work**:

---

- **Source 1**: [Automated Rowing Event Assignment, Li et al. 2025](https://doi.org/10.1080/14763141.2025.2528885)
  
  - **Objective**: Classify rowers into **8+** vs. **≤ 2** using:
    - demographics, rowing kinematics, coordination features
  - **Methods**:
    - **Data:** 7 IMUs (body segment orientation)  
    - **Models:** Logistic Regression, Decision Tree, K-Nearest Neighbors (KNN), Support Vector Machine (SVM), Extreme Gradient Boosting (XGBoost), Random Forest  
    - **Validation:** 5-fold cross-validation 
  - **Outcomes**:
    - **Best Accuracy:** Decision Tree, XGBoost, Random Forest (0.89–0.93)
    - **Lower Accuracy:** Logistic Regression, KNN, SVM (~0.76)
  - **Relation to the Project**:
    - Example for a pipeline: **motion → features → ML classification**  
    - Reference for **model selection & comparison**
    - **Limitation**: use of on-body sensor instead of video analysis --> look for studies using MediaPipe or sim.
      
---

- **Source 2**: [Analysis of indoor rowing motion using wearable inertial sensors, Bosch et al. 2015](https://doi.org/10.4108/eai.28-9-2015.2261465)

  - **Objective**: Distinguishing between experienced and novice rowers 
  - **Methods**:
    - ML recognizing two main phases of a stroke: drive and recovery
    - measuring: Absolute Angle + Stroke Timing Consistency; therefore:
      - developed quantitative model in which the reference angles for a certain positions in the stroke phase are defined
      - rower's stroke rates are compared to fix stroke rates depending on strokes per minutes (20 and 30)
    - **only** one experienced rower as reference rower (due to limited data)
    - **ML**:
      - KNN, WEKA for Analysis
      - performance evaluation: Matthew Correlation Coefficient (MCC)
  - **Outcomes**:
    - differences between experienced and novice rowers in stroke timing consistency
    - no differences: absolute posture angles
  - **Relation to the Project**:
    - same Goal: differentiating good vs. bad rowing technique --> bias in Bosch et al.: study assumes that inexperienced rowers will not perform the rowing technique in an optimal way compared to an experienced rower
    - **Limitation**: use of on-body sensor instead of video analysis --> look for studies using MediaPipe or sim.


---

- **Source 3**: [Title of Source 3]

  - **[Link]()**
  - **Objective**:
  - **Methods**:
  - **Outcomes**:
  - **Relation to the Project**:
