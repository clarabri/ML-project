# Literature Review

## Overview
Approaches and solutions that have been applied in similar projects.

**Summary of Each Work**:

---

- **Source 1**: [Automated Rowing Event Assignment, Li et al. 2025](https://doi.org/10.1080/14763141.2025.2528885)
  
  - **[Link to Paper](https://doi.org/10.1080/14763141.2025.2528885)**
  - **Objective**: Classify rowers into **8+** vs. **≤ 2** using:
    - demographics, rowing kinematics, coordination features
  - **Methods**:
    - **Data:** 7 IMUs (body segment orientation)  
    - **Models:** Logistic Regression, Decision Tree, K-Nearest Neighbors (KNN), Support Vector Machine (SVM), Extreme Gradient Boosting (XGBoost), Random Forest  
    - **Validation:** 5-fold cross-validation 
  - **Outcomes**:
    - **🟢 Best (Accuracy: 0.89–0.93):** Decision Tree, XGBoost, Random Forest  
    - **🔴 Lower (Accuracy: ~0.76):** Logistic Regression, KNN, SVM  
  - **Relation to the Project**:
    - Example for a pipeline: **motion → features → ML classification**  
    - Reference for **model selection & comparison**
      
---

- **Source 2**: Analysis of indoor rowing motion using wearable inertial sensors, Bosch et al. 2015

  - **[Link to Paper](https://doi.org/10.4108/eai.28-9-2015.2261465)**
  - **Objective**: Distinguishing between experienced and novice rowers 
  - **Methods**:
  - **Outcomes**:
  - **Relation to the Project**:

---

- **Source 3**: [Title of Source 3]

  - **[Link]()**
  - **Objective**:
  - **Methods**:
  - **Outcomes**:
  - **Relation to the Project**:
