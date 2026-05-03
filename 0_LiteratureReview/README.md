# Literature Review

## Overview
Approaches and solutions that have been applied in similar projects.

**Summary of Each Work**:

---

## Source 1

**Title:** Automated Rowing Event Assignment: A Machine Learning Approach  
**Authors:** Yumeng Li, Rachel M. Koldenhoven, Nigel C. Jiwan, Jieyun Zhan & Ting Liu (2025)  
**Journal:** *Sports Biomechanics*, 24(12), 3557–3569  

🔗 **Link:** https://doi.org/10.1080/14763141.2025.2528885  

---

## Objective
Assign rowers to different rowing event categories:
- **8+ (large crews)**
- **≤ 2 (small boats)**  

Based on:
- Demographics  
- Rowing kinematics  
- Coordination features  

---

## Methods

### Data Collection
- 7 synchronized **IMUs (Inertial Measurement Units)**  
- Attached to participants' bodies  
- Captured **angular orientation of body segments**  

### Machine Learning Models
The following supervised ML models were used:

- Logistic Regression  
- Decision Tree  
- K-Nearest Neighbors (KNN)  
- Support Vector Machine (SVM)  
- Extreme Gradient Boosting (XGBoost)  
- Random Forest  

**Validation Method:**  
- 5-Fold Cross-Validation  

---

## Outcomes

### 🟢 Best Performance (Accuracy: 0.89 – 0.93)
- Decision Tree  
- Extreme Gradient Boosting  
- Random Forest  

### 🔴 Lower Performance (Accuracy: ~0.76)
- Logistic Regression  
- K-Nearest Neighbors  
- Support Vector Machine  

---

## Relation to This Project

- Demonstrates how **rowing motion can be transformed into usable data**  
- Provides **relevant ML models for classification tasks**  
- Serves as a **benchmark for model selection and performance comparison**  


- **Source 1**: Automated rowing event assignment: a machine learning approach,
                by: Yumeng Li, Rachel M. Koldenhoven, Nigel C. Jiwan, Jieyun Zhan & Ting Liu (2025), 
                in: SPORTS BIOMECHANICS, 24/12, 3557–69.

  - **[[Link](https://doi.org/10.1080/14763141.2025.2528885)]()**
  - **Objective**: assigning rowers to different rowing events (8+ and <=2 event groups) based on their demographics, rowing kinematics and coordination features
  - **Methods**:
    - 7 synchronised IMUs (inertial measurement units) were affixed to participants’ bodies to monitor the angular orientation of body segments in absolute space.
    - **ML Models used**:
      -   logistic regression,
      -   decision tree,
      -   K-nearest neighbours,
      -   support vector machine,
      -   extreme gradient boosting, and
      -   random forest
      -   + 5fold cross validation
  - **Outcomes**:
    - best classification performance (accuracy ranging from 0.89 to 0.93):
      - decision tree,
      - extreme gradient boosting, and
      - random forest.
    - lower classification accuracy (accuracy = 0.76):
      - Logistic regression,
      - K-nearest neighbours, and
      - support vector machine. 
  - **Relation to the Project**: Example for processing rowing motion into data + possible models to use

- **Source 2**: Analysis of indoor rowing motion using wearable inertial sensors,
                by Bosch, S., Shoaib, M., Geerlings, S., Buit, L., Meratnia, N., & Havinga, P. (2015),
                in: In Proceedings of the 10th EAI International Conference on Body Area Networks.

  - **[[Link](https://doi.org/10.4108/eai.28-9-2015.2261465)]()**
  - **Objective**: machine learning model (K-nearest neighbour) to distinguish between experienced and novice rowers 
  - **Methods**:
  - **Outcomes**:
  - **Relation to the Project**:

- **Source 3**: [Title of Source 3]

  - **[Link]()**
  - **Objective**:
  - **Methods**:
  - **Outcomes**:
  - **Relation to the Project**:
