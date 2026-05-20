# Project Report: K-Nearest Neighbors (KNN) Intrusion Detection Pipeline

## 1. Exploratory Data Analysis & Dimensionality Reduction

The initial phase focused on identifying redundancies and removing statistical noise to ensure the K-Nearest Neighbors algorithm could calculate distances effectively without being overwhelmed by high dimensionality.

* **Action:** Calculated both the Spearman Correlation Matrix (for continuous variables) and the Correlation Ratio ($\eta$) (for categorical-to-numerical relationships).
* **Observation:** Several numerical features were mathematically identical or provided zero additional information. For example, `same_server_rate` and `different_server_rate` were exact mathematical inverses (-0.96 correlation). Features like `reset_error_rate` were almost entirely dictated by the categorical `connection_status` ($\eta = 0.98$). Features like `source_bytes` exhibited near-zero variance.
* **Result:** Dropped 12 highly redundant or zero-variance columns.
* **Why:** In KNN algorithms, highly correlated features act as a "double vote" for the same underlying pattern, artificially inflating the importance of that pattern. Removing them stabilized the feature space.

---

## 2. Feature Transformation & The "Outlier" Dilemma

Network traffic data natively violates the standard assumptions of machine learning models due to extreme skewness and heavy tails.

* **Action:** Categorized features into skewed continuous variables, bounded rates, discrete counts, and binary indicators. Applied a $\log(1+x)$ transformation exclusively to the heavily right-skewed and count-based features.
* **Observation:** Initial attempts to use standard Interquartile Range (IQR) and Quantile clipping mathematically erased critical data. Zero-inflated columns (like `fragment_errors`) were flattened entirely, and massive data spikes were capped to normal traffic levels.
* **Result:** Removed all clipping mechanisms from the pipeline.
* **Why:** In cybersecurity, extreme outliers are not statistical errors; they are the attacks (e.g., a massive spike in `destination_bytes` during exfiltration). Clipping masked these attacks. The log transformation successfully compressed the massive numerical ranges into a mathematically readable format while strictly preserving the extreme peaks so the model could detect them.

---

## 3. KNN-Specific Scaling Requirements

Distance-based algorithms require a perfectly level playing field to prevent larger numbers from dominating the mathematical calculations.

* **Action:** Applied `MinMaxScaler` to all numerical features.
* **Observation:** Standard robust scaling (`RobustScaler`) centers data well but does not enforce hard limits on maximum and minimum values.
* **Result:** All features were forced into a strict $0.0$ to $1.0$ grid.
* **Why:** If a transformed feature ranges from 0 to 12, and a rate feature ranges from 0 to 1, Euclidean distance calculations will treat the larger feature as 12 times more important. `MinMaxScaler` guarantees that every single feature contributes equally to the distance calculation.

---

## 4. Preventing Data Leakage

Translating text-based categories (like `protocol` and `service_type`) into numerical formats must be done without exposing the model to future data.

* **Action:** Implemented the train-test split *before* passing the data into `scikit-learn`'s `OneHotEncoder`.
* **Result:** Configured the encoder with `handle_unknown='ignore'`.
* **Why:** Encoding before splitting allows the model to "see" unique categories that only exist in the test set, creating an artificial advantage known as data leakage. Splitting first ensures the model evaluates unseen data fairly, and ignoring unknown variables prevents the pipeline from crashing if a novel service type appears during deployment.

---

## 5. Resolving Extreme Class Imbalance (SMOTE)

The dataset contained a massive imbalance, with normal traffic vastly outnumbering actual cyber attacks.

* **Observation:** The baseline model achieved a deceptive 99.7% accuracy. However, because KNN uses a majority vote from neighboring data points, the sheer volume of normal traffic "outvoted" the minority attack data, resulting in 6 False Negatives (missed attacks).
* **Action:** Replaced the standard pipeline with `imbalanced-learn`'s `Pipeline` and injected Synthetic Minority Over-sampling Technique (SMOTE).
* **Result:** SMOTE mathematically synthesized new, geometrically valid attack vectors within the training data.
* **Why:** Duplicating existing data leads to overfitting. Synthesizing data forced the KNN algorithm to recognize tighter attack clusters. The pipeline successfully increased Anomaly Recall to 98% (reducing missed attacks to 3), demonstrating a highly successful precision/recall trade-off (90% precision) that prioritized the capture of genuine threats over minimizing false alarms.