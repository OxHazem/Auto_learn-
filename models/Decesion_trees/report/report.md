

# Decision Tree Performance Analysis & Evaluation

### 1. Objective and Methodology

The objective of this phase was to evaluate the classification capabilities of a single Decision Tree on a network intrusion dataset. As a non-parametric algorithm, a Decision Tree relies entirely on hierarchical, threshold-based logic (e.g., splitting data based on True/False conditions) rather than geometric distance. Consequently, data scaling and logarithmic transformations were excluded from the pipeline.

The model was optimized using Optuna (Bayesian Optimization) to tune its structural parameters: maximum tree depth (`max_depth`), internal node splitting rules (`min_samples_split`), and the mathematical function used to measure split quality (`criterion`).

To assess the model's structural resilience and risk of overfitting, the optimization was performed under two conditions:

1. **Baseline Optimization:** Trained on the raw, highly imbalanced data.
2. **SMOTE Optimization:** Trained on data balanced using the Synthetic Minority Over-sampling Technique.

---

### 2. Baseline Model Performance (Without SMOTE)

When optimizing the raw, imbalanced dataset, the Bayesian optimizer identified a highly complex tree architecture as the mathematical peak:

* **Max Depth:** 29
* **Min Samples Split:** 4
* **Criterion:** Gini Impurity
* **Macro F1-Score:** 0.99

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.98 | Successfully detected 124 out of 126 attacks (2 False Negatives). |
| **Anomaly Precision** | 0.98 | Generated only 3 False Positives out of 2,682 normal connections. |
| **Overall Accuracy** | 1.00 | Statistically perfect accuracy at scale. |

**Analytical Insights:** On paper, the baseline Decision Tree achieved exceptional metrics. However, an analysis of the hyperparameters reveals a critical structural vulnerability: **Overfitting**. Because the anomaly class was so scarce, the tree was forced to grow to an extreme depth (`max_depth = 29`) to isolate individual, rare attack signatures. A single decision tree extending to 29 levels is highly brittle; it has practically memorized the training data. While it performed well on this specific test set, such a complex, rigid structure is highly susceptible to failing when exposed to novel, real-world network traffic.

---

### 3. SMOTE-Enhanced Model Performance

To observe how the tree handles balanced, dense data, it was retrained using SMOTE. The influx of synthetic attack data caused a drastic and highly desirable structural shift in the algorithm's logic:

* **Max Depth:** 16
* **Min Samples Split:** 4
* **Criterion:** Gini Impurity
* **Macro F1-Score:** 0.98

#### Performance Metrics

| Metric | Score | Interpretation |
| --- | --- | --- |
| **Anomaly Recall** | 0.97 | Successfully detected 122 out of 126 attacks (4 False Negatives). |
| **Anomaly Precision** | 0.95 | Generated 6 False Positives out of 2,682 normal connections. |
| **Overall Accuracy** | 1.00 | Maintained near-perfect statistical accuracy. |

**Analytical Insights:**
When SMOTE synthesized thousands of new attack points, it filled the continuous gaps between the sparse anomalies. This allowed the Decision Tree to find broader, much more generalized threshold rules. This is mathematically proven by the hyperparameter shift: the optimal **Max Depth collapsed from 29 down to 16**.

Because the tree was forced to generalize its rules rather than memorize the data, its test metrics experienced a slight, natural regression (Recall dropped by 1%, Precision dropped by 3%). However, structurally, this shallower tree is vastly superior. A tree of depth 16 is significantly more robust and trustworthy for deploying into a live production environment than a brittle tree of depth 29.

---

### 4. Final Conclusion

The evaluation of the single Decision Tree highlights the fundamental challenge of using standalone threshold algorithms on highly imbalanced data.

While the Baseline model achieved higher immediate test scores, its extreme depth (29 levels) indicates severe overfitting, rendering it a high-risk model for real-world deployment. The **SMOTE-Enhanced Decision Tree** represents the more structurally sound configuration; by utilizing balanced data, the algorithm successfully generalized its threshold rules (reducing depth to 16) while still maintaining an excellent 0.97 Recall and 0.95 Precision.

Ultimately, while the single SMOTE tree performs well, its slight drop in metrics when forced to generalize perfectly demonstrates why single decision trees are often upgraded to ensemble methods to achieve both generalization and high precision simultaneously.