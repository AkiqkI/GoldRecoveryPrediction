# 🥇 Predicting Gold Recovery with Machine Learning

## 📋 Project Overview

This project develops a machine learning model to predict gold recovery from raw ore in an industrial setting. Utilizing data from Zyfra, a company specializing in efficiency solutions for heavy industry, the goal is to forecast the amount of gold recovered at different stages of the extraction and purification process. Accurate prediction can significantly optimize operational efficiency and resource management.

## 🔗 Data Sources

The dataset consists of industrial measurements across various stages of the gold recovery process, provided in three CSV files:
-   `gold_recovery_train.csv`: Training dataset with features and target variables.
-   `gold_recovery_test.csv`: Test dataset (missing some features and all target variables).
-   `gold_recovery_full.csv`: Complete dataset with all features.

Each dataset is indexed by `date` (time of measurement), where rows close in time tend to exhibit similar characteristics.

## 🚀 Project Steps & Methodology

### 1. Data Preparation
-   **Loading and Inspection:** Datasets were loaded, and their structure, data types, and `date` index verified. Noted significant column differences between train (87 columns) and test (53 columns).
-   **Recovery Calculation Validation:** The `rougher.output.recovery` and `final.output.recovery` values were recalculated using the official formula. A Mean Absolute Error (MAE) of 0.0 confirmed perfect consistency with the provided data, implying that the official columns are reliable where present.
-   **Test Set Feature Check:** Confirmed that the missing columns in the test set were exclusively output and calculation parameters, preventing data leakage to the model during inference.
-   **Missing Value Handling:**
    -   Rows with `NaN` values in target columns (`rougher.output.recovery`, `final.output.recovery`) were dropped from the training set (~16% of rows).
    -   `NaN` values in input/state features were imputed using linear time interpolation, preserving the temporal nature of the data.
    -   After this, both training and test sets had a clean, consistent feature space.

### 2. Exploratory Data Analysis (EDA)
-   **Metal Concentrations:** Analyzed Au, Ag, and Pb concentrations across different purification stages. Au concentration consistently increased, Ag decreased, and Pb remained relatively stable, confirming the effectiveness of the purification process.
-   **Feed Particle Size:** Compared `rougher.input.feed_size` and `primary_cleaner.input.feed_size` distributions between the training and test sets. Statistical tests (t-test, KS-test) showed significant differences. As per project guidelines, these features were excluded from the final model evaluation to prevent bias from distribution shifts.
-   **Total Concentrations & Anomaly Removal:** Calculated total metal concentrations (Au+Ag+Pb+Sol) at various stages. Identified and removed rows where total concentration was zero, as these represent physically impossible states. This cleaned approximately 2.5% of the training data and 6.3% of the test data.

### 3. Modeling
-   **Evaluation Metric (sMAPE):** Implemented the Symmetric Mean Absolute Percentage Error (sMAPE) as the primary evaluation metric. The final sMAPE was a weighted average: `0.25 * sMAPE(rougher.output.recovery) + 0.75 * sMAPE(final.output.recovery)`.
-   **Model Training & Cross-Validation:** Multiple regression models (Linear, Ridge, Decision Tree, Random Forest, Gradient Boosting) were trained and evaluated using 5-fold cross-validation. 
    -   Two scenarios were considered:
        -   **Scenario A (Strict):** Excluded `feed_size` variables.
        -   **Scenario B (Exploratory):** Included `feed_size` variables.
    -   **Random Forest Regressor** consistently achieved the lowest weighted sMAPE in both scenarios (~5%).
-   **Final Model Testing:** The best Random Forest model (hyper-tuned via grid search) was re-trained on the full clean training data using **Scenario A features** (`n_estimators=300`, `max_depth=None`). Predictions were generated for the test set. The final weighted sMAPE on cross-validation was **4.98%**.

## 💡 Conclusion

The final model, a **tuned Random Forest Regressor**, effectively predicts gold recovery with a cross-validated weighted sMAPE of approximately **4.98%**. The thorough data preparation, rigorous EDA (including anomaly removal and distributional checks), and a robust modeling approach ensure a reliable and accurate predictive tool for Zyfra's gold extraction process.

## 🔮 Next Steps
1.  **Refine Hyperparameters:** Further exploration of Random Forest hyperparameters.
2.  **Feature Engineering:** Investigate new features (e.g., ratios, interactions).
3.  **Validation on New Data:** Continuously validate the model with new plant data to ensure long-term robustness.
