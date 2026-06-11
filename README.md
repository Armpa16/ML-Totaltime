# WMS-ML — ระบบพยากรณ์เวลารวมบริการรถบรรทุก SCG Roofing

## ปัญหาและเป้าหมาย

โรงงาน SCG Roofing รับรถบรรทุกเข้ามารับสินค้าวันละหลายร้อยคัน แต่ละคันผ่านกระบวนการตั้งแต่รับคิวจนออกจากโรงงาน ซึ่งใช้เวลาไม่เท่ากันขึ้นอยู่กับประเภทรถ ปริมาณสินค้า และสภาพคิวในขณะนั้น

โปรเจกต์นี้สร้าง ML model เพื่อ **ทำนาย `total_time_min`** — เวลารวมทั้งหมดตั้งแต่รถรับคิวจนออกจากโรงงาน เพื่อใช้วางแผนลำดับคิวและแจ้งเวลารอที่แม่นยำขึ้น

---

## total_time_min คืออะไร

เวลารวมประกอบด้วย 4 phase ต่อเนื่องกัน:

```text
รถรับคิว
   │
   ▼  wait_call_min        รอเรียกเข้าลาน
   │
   ▼  prepare_loading_min  รอโหลด
   │
   ▼  loading_time_min     โหลดสินค้าขึ้นรถ
   │
   ▼  close_job_min        ปิดงาน 
   │
รถออกจากโรงงาน
```

| Phase | ความหมาย | เฉลี่ย | P95 |
| --- | --- | ---: | ---: |
| `wait_call_min` | รอเรียกเข้าช่อง | 7.3 min | 19.1 min |
| `prepare_loading_min` | เตรียมสินค้า | 7.2 min | 21.6 min |
| `loading_time_min` | โหลดสินค้า | 25.3 min | 65.0 min |
| `close_job_min` | ปิดงาน | 15.1 min | 32.5 min |
| **`total_time_min`** | **รวมทั้งหมด** | **54.9 min** | **97.1 min** |

> ค่าสถิติจากข้อมูลหลัง outlier cleaning (22,152 แถว)

---

## ผลลัพธ์ (Results at a Glance)

**Best Model: LightGBM (Optuna + 5-Fold CV)**

| Model | Test RMSE | Test MAE | Test R² | Overfit Gap |
| --- | ---: | ---: | ---: | ---: |
| **LightGBM (Optuna+CV)** | **14.65 min** | **11.54 min** | **0.560** | **+0.12** |
| XGBoost (Optuna+CV) | 14.69 min | 11.55 min | 0.560 | +0.13 |
| CatBoost (Optuna+CV) | 14.69 min | 11.58 min | 0.560 | +0.11 |
| Baseline (Ridge) | 15.37 min | 12.18 min | 0.520 | +0.01 |
| DummyRegressor (mean) | 22.16 min | 17.68 min | −0.00 | — |

**การกระจาย error ของ LightGBM บน test set (4,431 แถว)**

| ช่วง error | สัดส่วน | สะสม |
| --- | ---: | ---: |
| ≤ 5 min | 27.3% | 27.3% |
| 5–10 min | 25.1% | **52.4%** |
| 10–15 min | 18.5% | **70.9%** |
| 15–30 min | 24.6% | 95.5% |
| > 30 min | 4.5% | 100% |

> ทำนายได้ภายใน ±10 นาที ถึง 52% ของรถทั้งหมด และภายใน ±15 นาที ถึง 71%

**ผลหลังการทำ Outlier Cleaning**

| Metric | ก่อน | หลัง | เปลี่ยน |
| --- | ---: | ---: | ---: |
| Best RMSE | 23.46 min | 14.65 min | −8.81 |
| ±10 min accuracy | 40.3% | 52.4% | +12.1% |
| >30 min error | 15.5% | 4.5% | −11.0% |

---

## Data Pipeline

### ภาพรวม Flow

```text
data/raw/
vwTimeStampDashboard_v3.csv
        │
        ▼  01_ViewRawData          สำรวจข้อมูลดิบ
        │
        ▼  02_TransFormData        แปลง datetime, encode, คำนวณ phase times
        │
        ▼  03_CleanData            ลบ null / duplicate / invalid rows
        │
        ▼  04_FeatureEngineering   สร้าง 27 features
        │
        ▼  05_EDA_TotalTime        วิเคราะห์ distribution ของ target
        │
        ▼  06_OutlierHandling      ตัด outlier ด้วย domain thresholds
        │
        ▼  07_TrainTestSplit       แบ่ง train/test ตาม time (80/20)
        │
   ┌────┴────┐
   ▼         ▼
train.csv  test.csv
(17,721)   (4,431)
   │
   ├──▶  01_BaselineModel      Ridge, Lasso, Linear Regression
   ├──▶  02_AdvancedModels     RF, XGBoost, LightGBM, CatBoost (Optuna)
   ├──▶  03_ModelEvaluation    เปรียบเทียบ + SHAP
   └──▶  04_ErrorAnalysis      วิเคราะห์ error ตาม segment
```

### รายละเอียดแต่ละ Notebook

#### PrepareData/

| Notebook | ทำอะไร | Input | Output |
| --- | --- | --- | --- |
| `01_ViewRawData` | สำรวจ schema, missing values, dtype | `vwTimeStampDashboard.csv` | — |
| `02_TransFormData` | แปลง datetime → float, คำนวณ wait/prepare/loading/close_job_min, encode CarType/PickListType | `vwTimeStampDashboard.csv` | `vw_timestamp_dashboard_transformed.csv` |
| `03_CleanData` | ลบแถวที่มี null ใน columns สำคัญ, ลบ duplicate, กรอง phase time ที่ติดลบ | `..._transformed.csv` | `vw_timestamp_dashboard_clean.csv` |
| `04_FeatureEngineering` | สร้าง time features (hour/dow/month), queue features, rolling avg, interaction terms | `..._clean.csv` | `vw_timestamp_dashboard_featured.csv` |
| `05_EDA_TotalTime` | วิเคราะห์ข้อมูล Plot distribution, skewness, correlation ของ total_time_min กับทุก feature | `..._featured.csv` | — (figures) |
| `06_OutlierHandling` | ตัด outlier ด้วย domain threshold (phase + CarType + total), modified Z-score | `..._featured.csv` | `vw_timestamp_dashboard_featured_no_outlier.csv` |
| `07_TrainTestSplit` | Sort by `OperatorCarConfirm`, ตัด 80% train / 20% test ตามเวลา (temporal split) | `..._no_outlier.csv` | `data/processed/train.csv`, `test.csv` |

#### Train/

| Notebook | ทำอะไร | Output |
| --- | --- | --- |
| `01_BaselineModel` | เทรน DummyRegressor, LinearRegression, Ridge, Lasso — เป็น lower bound | `baseline_best_totaltime.joblib` |
| `02_AdvancedModels` | เทรน RF (RandomizedSearch 50 iter) + XGBoost/LightGBM/CatBoost (Optuna 100 trials, 5-Fold CV) — ใช้เวลา ~1.5 ชั่วโมง | `lgb/xgb/cat_tuned_totaltime.joblib`, `best_model_totaltime.joblib` |
| `03_ModelEvaluation` | เปรียบเทียบ metrics ทุก model, residual plot, feature importance, SHAP analysis | `final_metrics_totaltime.csv`, figures |
| `04_ErrorAnalysis` | วิเคราะห์ว่า model ทำนายผิดมากตอนไหน แบ่งตาม CarType, PickListType, ช่วงเวลา, ปริมาณสินค้า | figures |

---

## Data

### แหล่งข้อมูล

| ไฟล์ | คำอธิบาย | แถว |
| --- | --- | ---: |
| `data/raw/vwTimeStampDashboard` | ข้อมูลหลัก — timestamp ทุก phase ของแต่ละ PackList | 32,505 | — |

### Columns หลักในข้อมูลดิบ

| Column | ประเภท | ความหมาย |
| --- | --- | --- |
| `PackListNo` | ID | เลขที่ใบ Pack List (1 คัน = 1 แถว) |
| `CarNo` | ID | ทะเบียนรถ |
| `CarType` | Categorical | ประเภทรถ: 0=10W, 1=4W, 2=6W, 3=Trailer |
| `PickListType` | Categorical | ประเภทคำสั่ง: Walk-in / SmartQ / etc. |
| `PostLocationName` | Categorical | ช่องโหลดสินค้า |
| `QueueTime` | Datetime | เวลาที่รถรับคิว |
| `OperatorCarConfirm` | Datetime | เวลาที่ operator เรียกรถเข้าช่อง |
| `PostingTime` | Datetime | เวลาที่ปิดงาน (รถออก) |
| `TruckSeqNo` | Numeric | ลำดับที่รถของวันนั้น |
| `*SapAmount` | Numeric | ปริมาณสินค้าแยกตามประเภท (tile/fitting/accessories) |

### ข้อมูลหลัง pipeline

| ขั้นตอน | จำนวนแถว | หมายเหตุ |
| --- | ---: | --- |
| Raw | 32,505 | ข้อมูลดิบ |
| หลัง Clean | ~26,538 | ลบ null / invalid |
| หลัง Outlier Removal | 22,152 | ตัด phase outlier + total > 200 min |
| Train set | 17,721 | 2025-01-02 → 2026-02-14 |
| Test set | 4,431 | 2026-02-14 → 2026-05-18 |

---

## Features (27 features)

### กลุ่มที่ 1 — ข้อมูลรถและคำสั่ง (5 features)

| Feature | ความหมาย |
| --- | --- |
| `CarType` | ประเภทรถ (0=10W, 1=4W, 2=6W, 3=Trailer) — ตัวแปรที่มีผลมากที่สุด |
| `PickListType` | ประเภทคำสั่ง (Walk-in, SmartQ, etc.) |
| `PrepareForward` | รถ PrepareForward หรือไม่ (เข้าช่องได้ทันที ไม่ต้องรอคิว) |
| `PostLocationName` | ช่องโหลดสินค้า |
| `TruckSeqNo` | ลำดับที่รถของวันนั้น |

### กลุ่มที่ 2 — เวลา/วันที่ (4 features)

| Feature | ความหมาย |
| --- | --- |
| `hour` | ชั่วโมงที่รถเข้า (0–23) |
| `day_of_week` | วันในสัปดาห์ (0=จันทร์) |
| `week_of_month` | สัปดาห์ที่เท่าไรของเดือน (1–5) |
| `month` | เดือน (1–12) |

### กลุ่มที่ 3 — ปริมาณสินค้า (7 features)

| Feature | ความหมาย |
| --- | --- |
| `total_sap_amount` | ปริมาณสินค้ารวมทุกประเภท (SAP units) — correlation สูงสุด (r=+0.62) |
| `total_tile_amount` | ปริมาณกระเบื้อง |
| `total_fitting_amount` | ปริมาณอุปกรณ์ fitting |
| `total_accessories_amount` | ปริมาณอุปกรณ์เสริม |
| `product_group_count` | จำนวนกลุ่มสินค้าที่มีในคำสั่ง |
| `sap_per_group` | ปริมาณสินค้าเฉลี่ยต่อกลุ่ม |
| `car_type_x_sap` | interaction: CarType × total_sap |

### กลุ่มที่ 4 — สถานะคิว ณ ขณะนั้น (7 features)

| Feature | ความหมาย |
| --- | --- |
| `queue_waiting` | จำนวนรถที่รอคิวอยู่ |
| `queue_loading` | จำนวนรถที่กำลังโหลดอยู่ |
| `queue_closing` | จำนวนรถที่กำลังปิดงานอยู่ |
| `total_queue` | รวมทุก queue |
| `available_bays` | ช่องโหลดที่ว่างอยู่ |
| `queue_x_bays` | interaction: total_queue × available_bays |
| `queue_x_sap` | interaction: total_queue × total_sap |

### กลุ่มที่ 5 — Rolling Statistics (4 features)

| Feature | ความหมาย |
| --- | --- |
| `rolling_avg_time_last5` | ค่าเฉลี่ย total_time ของ 5 คันก่อนหน้า (ทุกประเภทรถ) |
| `rolling_avg_cartype_last10` | ค่าเฉลี่ย total_time ของ 10 คันล่าสุดที่เป็นประเภทรถเดียวกัน — **top feature จาก SHAP** |
| `avg_time_by_cartype` | ค่าเฉลี่ย historical ของประเภทรถนั้น |
| `inter_arrival_min` | ระยะห่างระหว่างคันนี้กับคันก่อนหน้า (นาที) |

> **ทำไม rolling features ถึงสำคัญ**: สภาพโรงงานเปลี่ยนตลอดเวลา — ถ้า 10 คันก่อนหน้าใช้เวลานาน แสดงว่าขณะนั้นโรงงานช้า คันปัจจุบันก็น่าจะช้าด้วย

---

## โครงสร้างโฟลเดอร์

```text
WMS-ML/
├── data/
│   ├── raw/                    ข้อมูลดิบต้นฉบับ 
│   ├── interim/                ข้อมูลระหว่าง pipeline (_transformed, _clean, _featured, _no_outlier)
│   └── processed/              train.csv และ test.csv พร้อมใช้เทรน
│
├── models/                     model files (.joblib) + results (.csv)
│   ├── best_model_totaltime.joblib       LightGBM — best model
│   ├── lgb_tuned_totaltime.joblib
│   ├── xgb_tuned_totaltime.joblib
│   ├── cat_tuned_totaltime.joblib
│   ├── baseline_best_totaltime.joblib
│   ├── advanced_results_totaltime.csv    metrics ของทุก model
│   └── final_metrics_totaltime.csv
│
├── notebooks/
│   ├── PrepareData/            01–07 pipeline เตรียมข้อมูล
│   └── Train/                  01–04 training + evaluation
│
├── reports/
│   └── figures/                กราฟทุกอันที่ generate จาก notebooks
│
├── scripts/
│   └── setup_env.ps1           สร้าง virtual environment อัตโนมัติ
│
├── src/
│   └── utils/
│       └── paths.py            helper สำหรับ path constants
│
├── .gitignore
├── requirements.txt
└── run_jupyter.ps1
```

---

## Setup

### Prerequisites

- Python 3.10 หรือสูงกว่า
- PowerShell (Windows)

### ติดตั้ง

```powershell
# สร้าง virtual environment และติดตั้ง dependencies อัตโนมัติ
.\scripts\setup_env.ps1
```

หรือทำทีละขั้น:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### เปิด Jupyter

```powershell
.\run_jupyter.ps1
```

---

## วิธี Reproduce ผลลัพธ์

รัน notebook ตามลำดับนี้ทีละไฟล์ (Kernel → Restart & Run All):

### ขั้นที่ 1 — เตรียมข้อมูล (~5 นาที รวม)

```text
notebooks/PrepareData/01_ViewRawData.ipynb
notebooks/PrepareData/02_TranFormData.ipynb
notebooks/PrepareData/03_CleanData.ipynb
notebooks/PrepareData/04_FeatureEngineering.ipynb
notebooks/PrepareData/05_EDA_TotalTime.ipynb
notebooks/PrepareData/06_OutlierHandling.ipynb
notebooks/PrepareData/07_TrainTestSplit.ipynb
```

ผลลัพธ์: `data/processed/train.csv` และ `test.csv`

### ขั้นที่ 2 — เทรน Baseline (~1 นาที)

```text
notebooks/Train/01_BaselineModel.ipynb
```

ผลลัพธ์: `models/baseline_best_totaltime.joblib`

### ขั้นที่ 3 — เทรน Advanced Models (~1.5–2 ชั่วโมง)

```text
notebooks/Train/02_AdvancedModels.ipynb
```

> ขั้นนี้ใช้เวลานานเพราะ Optuna รัน 100 trials × 5-Fold CV สำหรับแต่ละ model
> ถ้าต้องการเร็วขึ้น ลด `N_TRIALS` ใน cell `imports` ลง เช่น `N_TRIALS = 30`

ผลลัพธ์: `models/lgb/xgb/cat_tuned_totaltime.joblib`, `models/best_model_totaltime.joblib`

### ขั้นที่ 4 — ประเมินผลและวิเคราะห์ (~2 นาที)

```text
notebooks/Train/03_ModelEvaluation.ipynb
notebooks/Train/04_ErrorAnalysis.ipynb
```

ผลลัพธ์: ตาราง metrics, SHAP plots, error analysis ใน `reports/figures/`

---

## Dependencies หลัก

| Package | การใช้งาน |
| --- | --- |
| `pandas`, `numpy` | จัดการข้อมูล |
| `scikit-learn` | Baseline models, preprocessing, cross-validation |
| `xgboost` | XGBoost model |
| `lightgbm` | LightGBM model (best performer) |
| `catboost` | CatBoost model |
| `optuna` | Hyperparameter tuning (Bayesian TPE) |
| `shap` | Model explainability |
| `matplotlib`, `seaborn` | Visualization |
| `joblib` | บันทึก/โหลด model |
