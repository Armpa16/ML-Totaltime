# Machine Learning Project

โครงโปรเจกต์นี้ออกแบบมาสำหรับงาน Machine Learning ที่ทำงานผ่าน Jupyter Notebook เป็นหลัก
แต่ยังคงแยกโค้ด reusable ออกจาก notebook เพื่อให้ดูแลง่ายขึ้นเมื่อโปรเจกต์ใหญ่ขึ้น

## โครงสร้างโฟลเดอร์

```text
d:\ML
|-- configs/                # config กลางของโปรเจกต์
|-- data/
|   |-- external/           # data จากแหล่งภายนอก
|   |-- interim/            # data ระหว่างแปลง/clean
|   |-- processed/          # data พร้อมใช้ train/evaluate
|   `-- raw/                # data ดิบต้นฉบับ
|-- models/                 # model artifacts
|-- notebooks/              # Jupyter notebooks
|-- reports/
|   `-- figures/            # รูปและกราฟสำหรับรายงาน
|-- scripts/                # helper scripts
|-- src/
|   |-- data/               # data loading / preprocessing
|   |-- features/           # feature engineering
|   |-- models/             # train / evaluate logic
|   |-- utils/              # utility functions
|   `-- visualization/      # plotting helpers
|-- .gitignore
|-- requirements.txt
`-- run_jupyter.ps1
```

## การเริ่มต้นใช้งาน

1. สร้าง virtual environment
2. ติดตั้ง dependencies
3. เปิด Jupyter Lab หรือ Jupyter Notebook

```powershell
.\scripts\setup_env.ps1
.\run_jupyter.ps1
```

หรือถ้าต้องการทำทีละขั้น:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
.\run_jupyter.ps1
```

## แนวทางใช้งาน

- ใช้ `notebooks/01_exploration.ipynb` สำหรับสำรวจข้อมูล
- ย้ายโค้ดที่ใช้ซ้ำไปไว้ใน `src/`
- เก็บ dataset ตามสถานะไว้ใน `data/raw`, `data/interim`, `data/processed`
- เก็บโมเดลที่เทรนแล้วใน `models/`
- เก็บกราฟหรือ output สำคัญไว้ใน `reports/figures/`

## คำแนะนำ

- ไม่ควรใส่ dataset ขนาดใหญ่ลง git
- notebook ควรโฟกัสที่ analysis และ experiment
- logic ที่ใช้ซ้ำหลายครั้งควรแยกไปเป็น module ใน `src/`
