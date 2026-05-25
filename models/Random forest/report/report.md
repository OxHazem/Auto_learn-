

# Random Forest Performance Analysis & Evaluation

### 1. Objective and Methodology

The objective of this phase was to evaluate the classification capabilities of a Random Forest ensemble on a network intrusion dataset. Because a Random Forest relies on hierarchical, threshold-based logic rather than geometric distance calculations, data scaling and logarithmic transformations were intentionally excluded from the pipeline to maximize computational efficiency.

The model was optimized using Optuna (Bayesian Optimization) to tune the ensemble's structural parameters: the number of trees (`n_estimators`), the maximum tree depth (`max_depth`), and the internal node splitting rules (`min_samples_split`). To assess the model's resilience to class imbalance, the optimization was performed under two conditions:

1. **Baseline Optimization:** Trained on the raw, highly imbalanced data.
2. **SMOTE Optimization:** Trained on data balanced using the Synthetic Minority Over-sampling Technique.

Models were evaluated using 3-fold cross-validation, optimizing specifically for the Macro F1-Score to ensure a fair balance between Precision and Recall.

---

### 2. Baseline Model Performance (Without SMOTE)

When optimizing the raw, imbalanced dataset, the Bayesian optimizer identified the following architecture as the mathematical peak:

* **Number of Trees:** 231
* **Max Depth:** 29
* **Min Samples Split:** 2
* **Macro F1-Score:** 0.9831

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.95 | Successfully detected 120 out of 126 attacks (6 False Negatives). |
| **Anomaly Precision** | 1.00 | Generated **0 False Positives** out of 2,682 normal connections. |
| **Overall Accuracy** | 1.00 | Statistically perfect accuracy at scale. |

**Analytical Insights:** The baseline Random Forest performed exceptionally well on the raw data, achieving a flawless 1.00 Precision score with absolutely zero false alarms. However, an analysis of the hyperparameters reveals the mechanical cost of this precision. Because the anomaly class was so scarce, the trees were forced to grow incredibly deep (`max_depth = 29`) and make highly specific, granular splits down to the absolute minimum threshold (`min_samples_split = 2`). This structural complexity indicates the model had to practically memorize individual, rare attack signatures. While accurate on the test set, this extreme depth poses a slight overfitting risk and resulted in 6 missed attacks (False Negatives).

---

### 3. SMOTE-Enhanced Model Performance

To address the missed attacks and improve generalization, the dataset was balanced using SMOTE, and the Optuna optimization was run again from scratch. The introduction of synthetic attack data caused a highly desirable structural shift in the Random Forest:

* **Number of Trees:** 232
* **Max Depth:** 15
* **Min Samples Split:** 6
* **Macro F1-Score:** 0.9861

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.96 | Successfully detected 121 out of 126 attacks (5 False Negatives). |
| **Anomaly Precision** | 0.99 | Generated only 1 False Positive out of 2,681 normal connections. |
| **Overall Accuracy** | 1.00 | Maintained near-perfect statistical accuracy. |

**Analytical Insights:**
By providing the model with a dense, balanced cluster of synthetic attacks, the algorithm no longer had to hunt for rare anomalies. This is mathematically proven by the massive shift in the optimal hyperparameters: the required **Max Depth dropped drastically from 29 to 15**, and the **Min Samples Split increased from 2 to 6**.

Because SMOTE filled in the continuous gaps between the attack vectors, the Random Forest was able to formulate broader, more generalized, and significantly simpler threshold rules. This generalized, shallower tree structure successfully captured an additional attack (lowering False Negatives to 5) while suffering a negligible penalty of just a single False Positive.

---

### 4. Final Conclusion

The **SMOTE-Enhanced Random Forest** is the optimal configuration for this architecture.

In a cybersecurity context, missing an active network intrusion is vastly more dangerous than generating a false alarm. By utilizing SMOTE, the Random Forest was able to simplify its internal decision trees, preventing the memorization of training data while simultaneously increasing its Anomaly Recall. The model effectively traded a single false alarm for the successful capture of a highly evasive attack, resulting in an incredibly robust, threshold-based defense system.