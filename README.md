# 🔍 Network Intrusion Detection System (NIDS)

A Random Forest-based Machine Learning pipeline designed for high-risk network traffic classification, validated via 
Stratified Cross-Validation to ensure operational reliability in Security Operations Center (SOC) environments.

## 📌 Overview
This project addresses the challenge of **real-time threat detection** by training a supervised learning model on NSL-KDD
benchmark data. Unlike standard "accuracy-focused" approaches, this system prioritizes **balanced detection across attack types**,
accounting for class imbalance and confidence score uncertainty.

| Metric | Value | Significance |
|--------|-------|--------------|
| **Mean F1-Macro** | 85.39% | Robust performance on imbalanced threat vectors |
| **Std Dev (±2SE)** | ±7.66% | Indicates stable cross-validation folds |
| **Validation Method** | Stratified 5-Fold CV | Prevents data leakage, accounts for class distribution |
| **Model Type** | Random Forest Classifier | High interpretability, handles tabular network traffic well |

---

## 🚀 Key Features

- ✅ **High-Risk Attack Filtering**: Removes attack types with <5 samples to prevent model instability.
- ✅ **Balanced Class Weights**: Mitigates dominance of "normal" traffic in training data.
- ✅ **Confidence Score Workflow**: Integrates probability outputs into SOC alert thresholds (Auto-block vs. Manual Review).
- ✅ **Operational Transparency**: Documents limitations regarding dataset compatibility (KDDTest+ attack type variance).

---

## 🛠 Tech Stack

- **Language**: Python 3.9+
- **Libraries**: `pandas`, `numpy`, `scikit-learn` (`RandomForestClassifier`, `StratifiedKFold`)
- **Visualization**: `matplotlib`, `seaborn`
- **Environment**: Conda / Virtualenv

---

## 📂 Dataset & Preprocessing

### Data Source
- **Training**: NSL-KDD (High-risk subset)
- **Validation**: KDDTest+ (Benchmark validation)

### Challenges & Mitigations
| Challenge | Solution |
|-----------|----------|
| **Class Imbalance** | `class_weight='balanced'` in Random Forest |
| **Sparse Attack Types** | Filtered attacks with ≥5 samples per fold before CV |
| **Dataset Mismatch** | Mapped KDDTest+ to training attack taxonomy (OpenSet awareness) |

### Feature Selection
The model utilizes standard network flow statistics: `duration`, `src_bytes`, `dst_bytes`, `serror_rate`, `rerror_rate`, `num_failed_logins`.

---

## 🧪 Methodology

### 1. Preprocessing
Data was cleaned using `Pandas` to handle missing values and outliers. Features were scaled using `StandardScaler` 
**fit-on-train-only** to prevent data leakage.

### 2. Model Training
- **Algorithm**: Random Forest (`n_estimators=100`, `max_depth=10`)
- **Validation**: Stratified 5-Fold Cross-Validation
- **Metric Optimization**: F1-Macro Score (balanced across all attack classes)

### 3. Cross-Validation Results
To ensure robust performance estimation, the model was validated using 5-fold stratified cross-validation.

| Fold | F1 Score | Status |
|------|----------|--------|
| Fold 1 | 0.8565 | ✅ Stable |
| Fold 2 | 0.8926 | ✅ High |
| Fold 3 | 0.8105 | ⚠️ Lower (Rare Attacks) |
| Fold 4 | 0.8995 | ✅ High |
| Fold 5 | 0.8106 | ⚠️ Lower (Rare Attacks) |

**Mean F1-Macro**: `0.8539`  
**Standard Deviation**: `±0.0766`

---

## 🛡 Attack Type Analysis

### High-Confidence Detection (~0.90–0.99)
| Attack Type | Category | Mean Confidence | Portfolio Impact |
|-------------|----------|-----------------|------------------|
| **portsweep** | Port Scanning | 0.998 ± 0.018 | Auto-Block Candidate |
| **back** | Backdoor/Trojan | 0.998 ± 0.015 | Secondary Validation |
| **teardrop** | DoS (Fragmentation) | 0.990 ± 0.0519 | Auto-Block Candidate |
| **satan** | FTP Attack | 0.976 ± 0.0448 | Manual Review if <0.90 |
| **warezclient** | File Transfer | 0.945 ± 0.125 | Auto-Block Candidate |
| **guess_passwd** | Password Cracking | 0.936 ± 0.0826 | Secondary Validation |
| **mailbomb** | Email Flooding | 0.994 ± 0.0243 | Manual Review if <0.90 |
| **apache2** | Web Application Attack | 0.968 ± 0.0435 | Analyst Review Recommended |

### Medium-Low Confidence Detection (~0.65–0.85)
These attack types often show lower confidence due to sparse training data or protocol variance.

| Attack Type | Category | Portfolio Confidence Score | Recommendation |
|-------------|----------|---------------------------|----------------|
| **buffer_overflow** | Exploitation | 0.710 ± 0.043 | Manual Review Required |
| **xterm** | Shell Execution | 0.761 ± 0.121 | Secondary Validation |

---

## ⚙️ Operational Workflow (SOC Integration)

To translate model outputs into actionable security tasks, the following confidence thresholds are recommended for production deployment:

| Confidence Threshold | Action Required | Rationale |
|----------------------|-----------------|-----------|
| **≥ 0.95** | Auto-Block / Escalate to SOC Lead | High-confidence threats with clear signatures |
| **0.80 – 0.95** | Secondary Validation by Analyst | Medium confidence; potential false positives/negatives |
| **0.65 – 0.79** | Manual Review Required | Low confidence; model uncertainty in rare attack detection |
| **< 0.65** | Investigation Only | Model does not meet deployment threshold |

---

## 📊 High-Risk Attack Filtered Dataset

The following attack types were included in high-risk analysis (hybrid risk score ≥ 0.3):

```python
filtered_attack_types = [
    'portsweep',       # ✅ High Confidence
    'normal',          # ⚠️ Baseline (for threshold calibration)
    'back',            # ✅ High Confidence
    'teardrop',        # ✅ High Confidence
    'satan',           # ✅ High Confidence
    'warezclient',     # ✅ High Confidence
    'buffer_overflow', # ⚠️ Medium-Low Confidence (0.7-0.8)
    'guess_passwd',    # ✅ High Confidence
    'mailbomb',        # ✅ High Confidence
    'apache2',         # ✅ High Confidence
    'xterm'            # ⚠️ Medium-Low Confidence (0.7-0.8)
]

# Total records after filtering: 11 attack types
```

**Note**: The `normal` class is included for baseline traffic analysis and model threshold calibration, not as a threat.

---

## ⚠️ Limitations & Future Work

### Current Limitations
1.  **Dataset Compatibility**: KDDTest+ contains ~36 attack types not fully represented in training data (e.g., `apache2`, `snmpgetattack`).
The model may classify these as "Normal" without explicit OpenSet handling.
3.  **Rare Attack Sparsity**: Attacks with <15 total records were excluded from primary CV analysis to maintain statistical significance.
4.  **Temporal Drift**: Training data is not time-ordered; real-world traffic patterns may drift over time.

### Future Improvements
- ✅ Implement **Active Learning** for rare attack types (e.g., buffer_overflow).
- ✅ Integrate **SHAP/LIME** for feature attribution in high-stakes alerts.
- ✅ Combine with **XGBoost** for ensemble modeling and improved recall on rare vectors.
- ✅ Develop a **Web Dashboard** (Streamlit) for SOC analysts to visualize alerts.

---
