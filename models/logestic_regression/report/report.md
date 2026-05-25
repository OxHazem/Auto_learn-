# Logistic Regression Performance Analysis: The Impact of SMOTE on Linear Decision Boundaries

### 1. Executive Summary

During the evaluation phase of the intrusion detection pipeline, a Logistic Regression classifier was trained and tested under two conditions: a baseline dataset (highly imbalanced) and a dataset balanced using the Synthetic Minority Over-sampling Technique (SMOTE).

The empirical results presented an inverse relationship compared to the previously tested K-Nearest Neighbors (KNN) model. While SMOTE vastly improved the distance-based KNN, it severely degraded the performance of the Logistic Regression model.

* **Baseline LR (No SMOTE):** Achieved an Anomaly Recall of 0.92 and a Precision of 0.97 (10 False Negatives, 3 False Positives).
* **SMOTE LR:** Achieved an Anomaly Recall of 0.94, but Precision collapsed to 0.64 (~8 False Negatives, ~66 False Positives).

The baseline model without SMOTE is the superior Logistic Regression configuration for this dataset due to the geometric limitations of linear classifiers.

### 2. The Mathematical Mechanics: Why SMOTE Harmed Precision

The severe drop in precision (-33%) when applying SMOTE to Logistic Regression is a direct result of the algorithm's mathematical rigidity.

Logistic Regression is a linear classifier. It separates classes by calculating a single, perfectly straight hyperplane ($z = w^T x + b$).

1. **The Baseline State:** In its raw, imbalanced state, the model draws a conservative line close to the minority anomaly cluster. Because network attacks often share mathematical similarities with heavy normal traffic, a few attacks inevitably fall on the "normal" side of this straight line (causing 10 False Negatives). However, the line is safe enough to ensure almost zero normal traffic is flagged (Precision: 0.97).
2. **The SMOTE Disruption:** SMOTE synthesizes new anomaly data points between existing ones. This artificially increases the density and spatial footprint of the anomaly class. To accommodate this massive influx of new attack data, the Logistic Regression model is forced to shift its straight linear boundary aggressively toward the normal traffic cluster.
3. **The Result:** By shifting this rigid boundary, the model successfully captures 2 previously missed attacks. However, because the boundary is a straight line, sweeping it into the normal traffic space accidentally captures dozens of perfectly normal connections, causing the False Positive rate to spike from 3 to approximately 66.

### 3. Algorithm Comparison: Linear Rigidity vs. Spatial Flexibility

This highlights a critical distinction in algorithm selection for cybersecurity datasets. A flexible algorithm like KNN easily handles SMOTE because it draws localized, non-linear boundaries around the newly synthesized data. It can isolate anomalies without disrupting the normal traffic space. Logistic Regression lacks this geometric flexibility; it cannot bend around normal traffic, forcing a severe trade-off between Recall and Precision.

### 4. Conclusion: Determining the "Best" Model

For the Logistic Regression architecture, the **Baseline Model (Without SMOTE) is the definitively better model.** In intrusion detection, prioritizing Recall (catching attacks) is generally paramount. However, the SMOTE-enhanced Logistic Regression only improved Anomaly Recall by a marginal 2% (catching just 2 additional attacks), while incurring an unacceptable 33% penalty to Precision (generating over 60 new false alarms). This trade-off is mathematically inefficient.

Ultimately, Logistic Regression's inability to mold to the complex, non-linear realities of network traffic makes it an inferior choice for this specific dataset when compared to geometrically flexible algorithms.

---

Now that we have successfully evaluated two algorithms (and proven why non-linear flexibility is so important for this data), we need to implement our final two classification models. Would you like to build a **Support Vector Machine (SVM)** to see how a more advanced boundary behaves, or move straight to a **Random Forest** to leverage the power of decision trees?