# Support Vector Machine (SVM) Final Evaluation Report

### 1. Objective and Methodology

The objective of this phase was to evaluate the classification capabilities of a Support Vector Machine (SVM) on a network intrusion dataset. Because SVM performance is highly dependent on hyperparameter configuration and mathematical geometry, Bayesian Optimization (Optuna) was utilized to conduct a comprehensive search across four distinct kernels (Linear, RBF, Polynomial, Sigmoid).

To assess the model's resilience to class imbalance, the optimization was performed under two conditions:

1. **Baseline Optimization:** Trained on the raw, highly imbalanced data.
2. **SMOTE Optimization:** Trained on data balanced using the Synthetic Minority Over-sampling Technique.

Models were evaluated using 3-fold cross-validation, optimizing specifically for the Macro F1-Score to ensure a fair balance between Precision and Recall.

---

### 2. Baseline Model Performance (Without SMOTE)

When optimizing the raw, imbalanced dataset, the Bayesian optimizer identified the following configuration as the mathematical peak:

* **Selected Kernel:** Linear
* **Regularization ($C$):** 2.465
* **Macro F1-Score:** 0.9625

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.90 | Successfully detected 114 out of 126 attacks (12 False Negatives). |
| **Anomaly Precision** | 0.97 | Only generated 3 False Positives out of 2,682 normal connections. |
| **Overall Accuracy** | 0.99 | Highly accurate overall, though skewed by the majority class. |

**Analytical Insights:** Because the anomaly class was severely underrepresented, the data was relatively sparse. Optuna proved that a simple, straight hyperplane (`kernel='linear'`) with low regularization ($C=2.46$) was sufficient to slice off the most obvious attacks without bleeding into normal traffic. This resulted in exceptionally high precision. However, the rigidity of the linear boundary caused the model to miss 12 sophisticated attacks, representing a critical vulnerability in an intrusion detection system.

---

### 3. SMOTE-Enhanced Model Performance

To address the missed attacks, the dataset was balanced using SMOTE, and the Optuna optimization was run again from scratch. The introduction of synthetic data caused a drastic shift in the optimal geometry:

* **Selected Kernel:** Radial Basis Function (`rbf`)
* **Regularization ($C$):** 40.117
* **Gamma ($\gamma$):** 0.0165
* **Macro F1-Score:** 0.9597

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.96 | Successfully detected 121 out of 126 attacks (Only 5 False Negatives). |
| **Anomaly Precision** | 0.92 | Generated 11 False Positives out of 2,682 normal connections. |
| **Overall Accuracy** | 0.99 | Maintained near-perfect overall accuracy. |

**Analytical Insights:**
When SMOTE synthesized thousands of new attack data points, the linear boundary became mathematically obsolete. A straight line cannot accommodate a dense, balanced cluster of anomalies without sweeping into normal traffic and destroying precision. Optuna recognized this and seamlessly shifted to the **RBF kernel**.

By projecting the data into a higher dimension, the RBF kernel was able to draw complex, localized, curved boundaries around the newly synthesized attack clusters. The high regularization parameter ($C=40.11$) allowed the model to construct strict defense margins, while the low gamma ($\gamma=0.0165$) ensured these boundaries remained generalized enough to avoid overfitting.

---

### 4. Final Conclusion & Model Selection

The **SMOTE-Enhanced SVM utilizing the RBF kernel** is definitively the superior configuration for this architecture.

In cybersecurity applications, minimizing False Negatives (missed attacks) is the highest priority. By leveraging the geometric flexibility of the RBF kernel, the SVM successfully absorbed the SMOTE data, resulting in a critical 6% increase in Anomaly Recall. It effectively traded a negligible 8 additional false alarms to successfully capture 7 highly evasive network attacks that the linear baseline completely missed. This proves the SVM's capability to provide a highly robust, non-linear defense against network intrusions.