# ENS Data Challenge | Qube Research & Technologies

### 🏆 **1st Place Public Leaderboard** | **4th Place Private Leaderboard**
**Final Score (C-Index): 0.7744**

---

## 📋 Project Overview
*Challenge organized by the École Normale Supérieure (ENS) and Qube Research & Technologies (QRT).*

The objective of this project is to predict the **overall survival** of patients with **Acute Myeloid Leukemia (AML)**. The dataset is complex, multimodal (clinical data, genomic mutations, cytogenetic reports), and subject to **right-censorship** (survival time is unknown for surviving patients).

This repository contains the full pipeline, from raw data processing to the final Stacked Generalization model.

---

## 🛠️ Step 1: Feature Engineering & Preprocessing
*(Corresponding to Sections 1 & 2 of the Notebook)*

The raw data requires significant transformation to be machine-learning ready. I implemented a robust preprocessing pipeline focusing on capturing biological signals.

### 1.1 Genomic Data (Target Encoding)
Instead of One-Hot Encoding the sparse mutation data, I used **Target Encoding**.
* **Method:** Each gene is replaced by the median survival time of patients carrying this mutation.
* **Result:** This creates high-value features like `worst_gene` (the mutation with the lowest survival expectancy for the patient).

### 1.2 Text Data (NLP on Cytogenetics)
The `CYTOGENETICS` column contains unstructured text describing chromosomal abnormalities.
* **TF-IDF & SVD:** I vectorized these reports using TF-IDF (n-grams) and reduced dimensionality with Truncated SVD to extract latent prognostic factors.
* **ELN 2017 Rules:** I implemented a hard-coded parser based on the **European LeukemiaNet 2017** guidelines to classify patients into *Favorable*, *Intermediate*, or *Adverse* risk categories using Regex.

### 1.3 Feature Importance Analysis
The chart below (generated in the notebook) confirms that our engineered features are the most predictive. The `worst_gene` feature (Target Encoding) is the #1 predictor, surpassing standard clinical variables.

![Feature Importance](Annexe_B2.png)
*Figure 1: Random Forest Feature Importance. Note the dominance of the engineered 'worst_gene' and 'mean_gene' features.*

---

## 🔍 Step 2: Unsupervised Learning (Clustering)
*(Corresponding to Section 3 of the Notebook)*

Before supervised modeling, I analyzed the population structure to identify hidden patient subgroups.

* **PCA & K-Means:** I projected the high-dimensional data using PCA and applied **K-Means Clustering** ($k=12$).
* **Outcome:** This identified distinct clusters of patients with similar genetic profiles. These Cluster IDs were added as features to the final model.

![PCA Projection](Annexe_B1.png)
*Figure 2: 2D Projection of patient profiles. The colors represent the 12 clusters identified by K-Means, showing distinct biological subgroups.*

---

## 🧠 Step 3: Modeling Strategy (Stacking)
*(Corresponding to Section 4 of the Notebook)*

To handle censorship and maximize the C-Index, I used a **Stacking Ensemble** of 9 diverse models.

### 3.1 Handling Censorship
Standard regression cannot handle censored data (patients who don't die during the study). I used two strategies:
1.  **Time Expansion:** For classification models, I duplicated patient rows for multiple time intervals to learn the probability of survival at $t, t+1, t+2...$
2.  **Semi-Supervised Correction:** For regression models, I imputed the survival time of censored patients using a Random Forest trained on non-censored patients.

### 3.2 The Stacking Architecture
I trained **9 base models** including Histogram Gradient Boosting (HGBT), Random Forests, MLP (Neural Network), KNN, and Lasso.

The correlation matrix below shows why Stacking works: while tree-based models (bottom left) are correlated, the **KNN and Lasso** models (darker squares) are decorrelated. This diversity stabilizes the final prediction.

![Correlation Matrix](Annexe_B3.png)
*Figure 3: Pearson Correlation Matrix between the 9 base models. Diversity (low correlation) is key for ensemble performance.*

---

## 📈 Step 4: Final Results
*(Corresponding to Section 5 of the Notebook)*

The final predictions are a weighted average of the 9 models.

* **Public Leaderboard:** **1st Place** (0.7744)
* **Private Leaderboard:** 4th Place

### Clinical Validation (Kaplan-Meier)
To verify the model's usefulness, I plotted survival curves for patients predicted as "Low Risk", "Intermediate Risk", and "High Risk". The curves are clearly separated, proving the model effectively stratifies patients.

![Kaplan-Meier Curves](Annexe_B4.png)
*Figure 4: Kaplan-Meier survival curves on the validation set. The model successfully distinguishes high-risk patients (bottom curve) from low-risk patients (top curve).*

---
