

# Logistic Regression Performance Analysis & Evaluation

### 1. Objective and Methodology

The objective of this phase was to evaluate the classification capabilities of Logistic Regression on a network intrusion dataset. As a parametric, linear algorithm, Logistic Regression separates classes by calculating a single, straight hyperplane ($z = w^T x + b$).

To assess the model's resilience to class imbalance and its geometric limitations, it was evaluated under two conditions:

1. **Baseline Optimization:** Trained on the raw, highly imbalanced data.
2. **SMOTE Optimization:** Trained on data balanced using the Synthetic Minority Over-sampling Technique (SMOTE).

The models were evaluated based on their ability to balance Anomaly Recall (minimizing missed attacks) and Anomaly Precision (minimizing false alarms).

---

### 2. Baseline Model Performance (Without SMOTE)

When trained on the raw, imbalanced dataset, the Logistic Regression model established a highly precise, albeit conservative, linear boundary.

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.92 | Successfully detected 116 out of 126 attacks (10 False Negatives). |
| **Anomaly Precision** | 0.97 | Only generated 3 False Positives out of 2,682 normal connections. |
| **Macro F1-Score** | 0.97 | Indicates an excellent balance across the imbalanced classes. |

**Analytical Insights:** In its raw state, the anomaly class is sparse. The algorithm successfully calculated a straight boundary that sliced off the most obvious attacks without bleeding into normal traffic, resulting in a near-perfect Precision score of 0.97. However, because network attacks often share mathematical similarities with heavy normal traffic, 10 sophisticated attacks fell on the "normal" side of this rigid line, representing a vulnerability in the detection pipeline.

---

### 3. SMOTE-Enhanced Model Performance

To address the missed attacks, the dataset was balanced using SMOTE. The introduction of synthetic attack data severely degraded the model's performance, exposing the geometric limitations of linear classifiers.

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.94 | Successfully detected 119 out of 126 attacks (7 False Negatives). |
| **Anomaly Precision** | 0.64 | **Generated 68 False Positives** out of 2,682 normal connections. |
| **Macro F1-Score** | 0.87 | Represents a severe degradation in overall model balance. |

**Analytical Insights:**
The severe precision collapse (-33%) is a direct result of the algorithm's mathematical rigidity. SMOTE synthesizes thousands of new data points, artificially increasing the density and spatial footprint of the anomaly class. To accommodate this massive influx of new attack data, the Logistic Regression model was forced to shift its straight linear boundary aggressively toward the normal traffic cluster.

Because the boundary is a perfectly straight line, it cannot mold or curve around dense data clusters. Sweeping it forward successfully captured 3 previously missed attacks, but accidentally swallowed a massive portion of the normal traffic space, causing the False Positive rate to spike from 3 to 68.

---

### 4. Final Conclusion & Model Selection

For the Logistic Regression architecture, the **Baseline Model (Without SMOTE)** is definitively the superior configuration.

While balancing data with SMOTE is generally best practice for anomaly detection, it is mathematically incompatible with this specific linear model on this dataset. The SMOTE-enhanced version traded a catastrophic 33% penalty to Precision (generating 65 new false alarms) for a marginal 2% gain in Recall (catching just 3 additional attacks). In a real-world cybersecurity environment, this influx of false positives would cause severe alert fatigue. Ultimately, Logistic Regression's inability to mold to the non-linear realities of dense network traffic makes the baseline, imbalanced approach the only viable option for this specific algorithm.