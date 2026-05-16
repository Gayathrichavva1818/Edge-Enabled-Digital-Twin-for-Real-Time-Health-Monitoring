# 🏥 Edge-Enabled Digital Twin Framework for Real-Time Health Monitoring

> Final Year Project — IIITDM Kancheepuram
> CHAVVA GAYATHRI (CS22B2018)
> Department of Computer Science and Engineering - Major in AI

---

##  Project Overview

This project implements a complete **Edge-Enabled Digital Twin (DT) healthcare monitoring system** deployed on a **Raspberry Pi** edge device. The system continuously monitors patient vital signs using IoT sensors, runs AI inference locally (no cloud needed), and triggers real-time alerts when abnormal conditions are detected.

The project extends the reference paper:
> *"A Digital Twin Framework for Real-Time Healthcare Monitoring: Leveraging AI and Secure Systems for Enhanced Patient Outcomes"* — Discover Internet of Things, Springer 2025

---

## 🆕 What This Project Adds Beyond the Paper

| Feature | Reference Paper | This Project |
|---|---|---|
| Edge Device | NodeMCU ESP8266 | Raspberry Pi 4B |
| Models Compared | 3 models | 8 models (4 Hybrid + 4 DL) |
| Class Imbalance | Not addressed | SMOTE + Class Weights |
| Deep Learning | Not included | LSTM, BiLSTM, Attention, TCN |
| Dataset | 20 participants + MIMIC-III | 20 real + 80 synthetic + MIMIC-III |
| Deployment | Jupyter Notebook | Flask API + Streamlit + Email Alerts |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────┐
│         PHYSIOLOGICAL SENSORS       │
│   MAX30102 (HR + SpO2)              │
│   MLX90614 (Body Temperature)       │
└──────────────┬──────────────────────┘
               │ I2C Protocol
               ▼
┌─────────────────────────────────────┐
│       RASPBERRY PI (EDGE)           │
│  • Data Preprocessing               │
│  • SMOTE + Class Weight Balancing   │
│  • AI Inference (8 Models)          │
│  • Flask REST API                   │
└──────────────┬──────────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
   NORMAL            ABNORMAL
       │                │
       ▼                ▼
  Cloud Storage    Alert System
  (SQLite +        (Email + SMS)
   Firebase)            │
       │                ▼
       └──────► Digital Twin
                Simulation +
                Dashboard Update
```

---

##  Models Implemented

### 4 Hybrid Models
| Model | Description |
|---|---|
| **MXBoost (Proposed)** | MLP + XGBoost with mode voting |
| RF + XGB Blend | Random Forest + XGBoost weighted blend |
| LGBM + CatBoost Vote | LightGBM + CatBoost average voting |
| Autoencoder + XGB | Deep feature extraction + XGBoost |

### 4 Deep Learning Models
| Model | Description |
|---|---|
| LSTM | Temporal dependency capture |
| BiLSTM | Bidirectional temporal learning |
| LSTM + Attention | Attention-weighted time steps |
| TCN | Dilated causal convolutions |

---

## 📊 Dataset

| Source | Records | Type |
|---|---|---|
| Real sensor data | 20 | Raspberry Pi collected |
| Synthetic data | 80 | MIMIC-III structure mirrored |
| MIMIC-III | 1,177 | Anonymised ICU records |
| **Total** | **1,277** | Combined training corpus |

**Health Targets:**
- TargetHR — Heart Rate (abnormal if < 60 or > 100 bpm)
- TargetSpO2 — Oxygen Saturation (abnormal if < 95%)
- TargetBT — Body Temperature (abnormal if < 36.1°C or > 37.2°C)
- TargetDM — Diabetes Risk (abnormal if Glucose > 140 or Creatinine > 1.5)

---

## 📁 Project Structure

```
digital_twin_healthcare/
│
├── 📄 generate_synthetic_data.py   → Generate 100-patient synthetic dataset
├── 📄 models_train.py              → Train all 8 models with SMOTE
├── 📄 visualizations.py            → All paper figures + extended plots
├── 📄 flask_api.py                 → Central REST API
├── 📄 dashboard.py                 → Streamlit real-time dashboard
├── 📄 sensor_reader.py             → MAX30102 + MLX90614 sensor reader
├── 📄 alerts.py                    → Email + SMS alert system
├── 📄 requirements.txt             → All Python dependencies
│
├── 📁 data/
│   ├── synthetic_100_patients.csv  → Synthetic dataset
│   └── mimic/                      → MIMIC-III CSV files (place here)
│
├── 📁 models/                      → Saved trained models (auto-created)
└── 📁 outputs/                     → All plots and results (auto-created)
```

---

## ⚙️ Installation

### Step 1 — Clone or download the project
```bash
cd digital_twin_healthcare
```

### Step 2 — Install dependencies
```bash
pip install -r requirements.txt
```

### Step 3 — For Raspberry Pi sensor support only
```bash
pip install adafruit-circuitpython-mlx90614 max30102 RPi.GPIO adafruit-blinka
```

---

## ▶️ How to Run

### Step 1 — Generate Synthetic Data
```bash
python generate_synthetic_data.py
```

### Step 2 — Train All 8 Models

**With MIMIC-III + Synthetic (Recommended):**
```bash
python models_train.py --mimic data/mimic/ --synthetic data/synthetic_100_patients.csv
```

**With Synthetic Only (Demo):**
```bash
python models_train.py --synthetic data/synthetic_100_patients.csv
```

**Skip training, regenerate plots only:**
```bash
python models_train.py --skip-train
```

### Step 3 — Start Flask API (Terminal 1)
```bash
python flask_api.py
```
Runs at: http://localhost:5000

### Step 4 — Start Dashboard (Terminal 2)
```bash
python -m streamlit run dashboard.py
```
Opens at: http://localhost:8501

### Step 5 — Start Sensor Reader (Terminal 3)
```bash
# Demo mode (uses synthetic data — works on laptop/PC)
python sensor_reader.py --demo

# Real sensors (Raspberry Pi only)
python sensor_reader.py
```

### Step 6 — Test Email Alerts (Optional)
```bash
python alerts.py --test
```

---

## 📊 Key Results

| Model | Accuracy | F1-Score | Test Time |
|---|---|---|---|
| **MXBoost (Proposed)** | **0.9625** | 0.6250 | **3.9ms** |
| AdaBoost | 0.9625 | **0.9167** | 9.9ms |
| Naive Bayes | 0.9500 | 0.8264 | ~0ms |
| Random Forest | 0.9250 | 0.8250 | 53.0ms |
| SVM | 0.9250 | 0.7698 | 1.2ms |

**MXBoost Per-Target Performance:**
| Target | Precision | Recall | F1 |
|---|---|---|---|
| HR | 1.0000 | 1.0000 | 1.0000 |
| SpO2 | 1.0000 | 1.0000 | 1.0000 |
| BT | 0.9105 | 0.9000 | 0.8778 |
| DM | 0.9025 | 0.9500 | 0.9256 |

---

## 📡 Flask API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/reading` | Receive sensor data, run inference |
| GET | `/api/latest` | Latest patient reading |
| GET | `/api/history` | Historical readings |
| GET | `/api/stats` | System statistics |
| GET | `/api/digital_twin/<id>` | Patient DT state |

---

## 🔔 Alert System Setup

### Email Alerts (Gmail)
1. Enable **App Passwords** in your Gmail account
2. Go to: https://myaccount.google.com/apppasswords
3. Open `alerts.py` and fill in:

```python
EMAIL_CONFIG = {
    "sender_email" : "your_email@gmail.com",
    "app_password" : "your_app_password",
    "recipients"   : ["doctor@hospital.com"],
}
```

### SMS Alerts (Twilio - Optional)
1. Create free account at twilio.com
2. Fill in `alerts.py`:

```python
SMS_CONFIG = {
    "enabled"      : True,
    "account_sid"  : "your_twilio_sid",
    "auth_token"   : "your_twilio_token",
    "from_number"  : "+1234567890",
    "to_numbers"   : ["+91XXXXXXXXXX"],
}
```

---

## 📈 Output Figures Generated

All figures saved to `./outputs/`:

| File | Description |
|---|---|
| `fig7_confusion_matrix.png` | Consolidated 8×8 confusion matrix |
| `fig8_radar_charts.png` | Baseline vs Hybrid radar charts |
| `fig9_performance_bars.png` | Accuracy, Precision, Recall, F1 bars |
| `fig10_autocorrelation_avp.png` | Actual vs Predicted ACF |
| `fig11_autocorrelation_rolling.png` | ACF + Rolling Average |
| `table5_macro_weighted.png` | Per-class metrics table |
| `table6_model_comparison.png` | Full model comparison |
| `ext1_grouped_bar.png` | All models accuracy and F1 |
| `ext2_f1_heatmap.png` | F1 heatmap: models × targets |
| `ext3_time_comparison.png` | Training and testing time |
| `ext4_per_target_all_models.png` | Per-target all models |
| `model_comparison_9models.csv` | Full results CSV |

---

## 🔧 Tech Stack

| Layer | Technology |
|---|---|
| Hardware | Raspberry Pi 4B, MAX30102, MLX90614 |
| Protocol | I2C, HTTP/REST, JSON, SMTP |
| ML/DL | MXBoost, LSTM, BiLSTM, TCN, RF, XGBoost, LGBM, CatBoost |
| Imbalance | SMOTE + Class Weights |
| API | Flask + SQLite |
| Dashboard | Streamlit + Plotly |
| Alerts | Gmail SMTP + Twilio SMS |
| Cloud | Firebase Realtime Database |

---

## 🔮 Future Work

- **Hierarchical Federated Learning** — multi-hospital collaborative training without sharing raw patient data
- **1D-CNN and Transformer** models for improved temporal pattern recognition
- **Continuous Glucose Sensor** integration for live DM prediction
- **Hospital Deployment** with EHR system integration
- **Quantum-Resistant Security** for long-term data protection

---

## 📚 References

1. Jameil & Al-Raweshidy, "A Digital Twin Framework for Real-Time Healthcare Monitoring," *Discover Internet of Things*, Springer 2025.
2. Gupta et al., "Hierarchical Federated Learning for Smart Healthcare," *IEEE CIC*, 2021.
3. Johnson et al., "MIMIC-III, a freely accessible critical care database," *Scientific Data*, 2016.
4. Chawla et al., "SMOTE: Synthetic Minority Over-sampling Technique," *JAIR*, 2002.
5. Chen & Guestrin, "XGBoost: A Scalable Tree Boosting System," *KDD*, 2016.

---

## 👩‍💻 Author

**CHAVVA GAYATHRI**
Roll No: CS22B2018
B.Tech — Computer Science and Engineering (Major in AI)
IIITDM Kancheepuram

**Guide:** Dr. Bhukya Krishna Priya
Assistant Professor, Dept. of CSE-AI, IIITDM Kancheepuram

---

*Digital Twin Healthcare | Raspberry Pi Edge AI | MXBoost Model*
