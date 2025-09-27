
# 🧠 Cognitive Load and Response Time Estimation from Physiological Signals

---

## Step 1. Data Understanding & Preprocessing

### 1.1 Dataset and Goal
This project utilizes a multi-sensor dataset of participants engaged in cognitive tasks. The primary goal is to predict a user's **response time** by analyzing their physiological signals, mainly from Electroencephalography (EEG) and Eye-Tracking (ET).

**Signals Used:**
1.  **Electroencephalography (EEG):** Brain activity, specifically the power in Delta, Theta, Alpha, Beta, and Gamma frequency bands.
2.  **Eye-Tracking (ET):** Gaze location, pupil size, and derived metrics like fixations and saccades.
3.  **Task Performance Logs:** The ground truth `Response Time` for each task.

### 1.2 Data Preprocessing
The raw sensor data was extensive and required significant cleaning and synchronization.
- **Time Alignment:** All data streams (EEG, Eye-Tracking, Task Logs) were synchronized to a common timeline.
- **Data Segmentation:** Continuous recordings were segmented into individual trials corresponding to each question/task.
- **Noise Reduction:** Outlier values and sensor artifacts were handled to ensure data quality.
- **IVT Algorithm:** The I-VT (Velocity-Threshold) algorithm was applied to the raw eye-gaze data to classify eye movements into **fixations** and **saccades**, providing crucial behavioral metrics.

The processed data for each trial was saved into separate, synchronized CSV files (`_eeg.csv`, `_eye.csv`, `_ivt.csv`, `_meta.csv`).

---

## Step 2. Feature Engineering

To build a robust predictive model, we extracted meaningful statistical features from the time-series data of each trial. This step is crucial for transforming raw sensor signals into a format suitable for machine learning.

**Features Extracted:**

1.  **From EEG Data:**
    - For each frequency band (Delta, Theta, Alpha, Beta, Gamma):
        - `mean`, `std`, `var`, `min`, `max` of the band power across the trial.
        - **Spectral Entropy:** A measure of the complexity and predictability of the signal in each band.
    - **EEG Ratios:** The **Theta/Beta Ratio**, a well-known indicator of cognitive load and attention.

2.  **From Eye-Tracking (ET & IVT) Data:**
    - **Pupil Dilation:** `mean`, `std`, `var`, `min`, `max` of the average pupil size.
    - **Gaze Variability:** The standard deviation of gaze coordinates (x, y).
    - **Fixation Features:** `Average Fixation Duration` and `Total Fixation Count`.
    - **Saccade Features:** `Average Saccade Amplitude`.
    - **Blink Features:** A simple `Blink Count` was approximated by identifying moments of pupil data loss.

✅ The process resulted in a final feature set with **38 distinct features** for each trial.
✅ This dataset, `final_features.csv`, formed the basis for all subsequent model training.

---

## Step 3. Model Training & Evaluation

We trained and evaluated three different types of models to find the most effective one for predicting response time from our engineered features.

| Model Type                 | Description                                                                 |
| -------------------------- | --------------------------------------------------------------------------- |
| **Gradient Boosting** | A powerful tree-based ensemble model that builds predictors sequentially.   |
| **Bidirectional LSTM (BiLSTM)** | A deep learning model designed to process sequential (time-series) data.  |
| **TCNN (Temporal CNN)** | A deep learning model using 1D convolutions to find patterns over time.     |

**Results (Final Performance):**

| Model               | Final MAE | Accuracy (%) |
| ------------------- | --------- | ------------ |
| **Gradient Boosting** | **0.22** | **97.54%** |
| BiLSTM              | 7.11     | 58.94%       |
| TCNN                | 7.60     | 56.82%       |

🔴 The deep learning models (BiLSTM, TCNN) performed poorly. This is likely because our **strong feature engineering** already captured the most important temporal information, making them more suitable for traditional ML models.

✅ **Best Model:** Gradient Boosting Regressor

---

## Step 4. Model Interpretation with SHAP

To understand *why* the Gradient Boosting model was so effective, we used **SHAP (SHapley Additive exPlanations)** to analyze its feature importance.


**Key Insights from SHAP Analysis:**
- The analysis revealed that both EEG and Eye-Tracking features were critical for accurate predictions.
- **Top Features:** Features like `eeg_alpha_spectral_entropy`, `eye_pupil_mean`, and `eeg_theta_beta_ratio` were among the most influential.
- **Conclusion:** The model learned to effectively combine signals from both brain activity and eye movements to estimate cognitive load and, consequently, response time.

---

## ✅ Final Conclusion

- **Feature Engineering is Key:** The success of this project hinged on transforming raw time-series signals into a rich set of statistical features. Using simple averages was insufficient, but a comprehensive feature set (`mean`, `std`, entropy, etc.) yielded excellent results.
- **Traditional ML Outperforms DL:** For this feature-rich, tabular dataset, the **Gradient Boosting** model significantly outperformed more complex deep learning architectures.
- **Reliable Estimation:** The final model achieved a **Mean Absolute Error of 0.22** and an accuracy of **~97%**, proving that **response time can be reliably estimated from a combination of EEG and Eye-Tracking signals.**