# 5.0 Train Seed Model

## 1. วัตถุประสงค์

โค้ดนี้ใช้สำหรับ **Train YOLO Model ด้วย Seed Dataset**

โดยนำ:

```text
P711_Seed/
├── images/
├── labels/
└── data.yaml
```

จาก **STEP 3.0 + STEP 4.0**

มา Train ด้วย:

```text
yolo11n.pt
```

ผลลัพธ์สำคัญคือ:

```text
best.pt
```

ซึ่งเป็น Model ที่มี Performance ดีที่สุดระหว่างการ Train

---

# 2. Flow การทำงาน

```text
STEP 3.0
P711_Seed
   │
   ▼
STEP 4.0
data.yaml
   │
   ▼
STEP 5.0
YOLO Training
   │
   ├── epochs = 100
   ├── imgsz = 960
   ├── batch = 8
   └── GPU = 0
   │
   ▼
runs/detect/
└── P711Test_Seed/
    └── weights/
        ├── best.pt
        └── last.pt
```

---

# 3. Import Library

```python id="x1c3l7"
from pathlib import Path
from ultralytics import YOLO
```

### `Path`

ใช้จัดการ Path ของ Dataset และ Model

### `YOLO`

ใช้โหลด Model และสั่ง Train

---

# 4. กำหนด Dataset

```python id="6y4w8z"
DATASET = Path(
    r"C:\Users\pawor\Desktop\P711Test"
)
```

กำหนด Dataset หลัก

---

# 5. หา `data.yaml` ของ Seed

```python id="m2v9qp"
yaml = next(
    f for f in DATASET.rglob("data.yaml")
    if "seed" in f.parent.name.lower()
)
```

ส่วนนี้ใช้ค้นหา:

```text
data.yaml
```

ภายใน Dataset

---

## `rglob("data.yaml")`

ค้นหา `data.yaml` ทุก Folder ย่อย

ตัวอย่าง:

```text
P711Test/
├── data.yaml
└── P711_Seed/
    └── data.yaml
```

จะค้นพบทั้งสองไฟล์

---

## `if "seed" in f.parent.name.lower()`

ตรวจชื่อ Folder ที่อยู่ก่อน `data.yaml`

ตัวอย่าง:

```text
P711_Seed/data.yaml
```

Folder คือ:

```text
P711_Seed
```

จาก:

```python id="7u7n0k"
f.parent.name.lower()
```

จะได้:

```text
p711_seed
```

ซึ่งมีคำว่า:

```text
seed
```

จึงเลือกไฟล์นี้

---

# 6. ทำไมใช้ `next()`

```python id="q9c4e1"
next(
    f for f in ...
)
```

หมายถึง:

> เอาไฟล์แรกที่ตรงกับเงื่อนไข

ดังนั้น `yaml` จะกลายเป็น Path เช่น:

```text
C:\Users\pawor\Desktop\P711Test\P711_Seed\data.yaml
```

---

# 7. โหลด YOLO Model

```python id="j7k3p8"
model = YOLO(
    r"C:\Users\pawor\Desktop\yolo11n.pt"
)
```

โหลด Pretrained Model:

```text
yolo11n.pt
```

แล้วเก็บไว้ใน:

```python id="s4m2x6"
model
```

Model นี้จะเป็นจุดเริ่มต้นของการ Train

---

# 8. `model.train()`

```python id="v3n8q1"
results = model.train(
```

สั่งให้ YOLO เริ่ม Training

ผลลัพธ์จะถูกเก็บใน:

```python id="a5k7m2"
results
```

---

# 9. `data`

```python id="p6r4t8"
data=str(yaml)
```

บอก YOLO ว่า Dataset Configuration อยู่ที่ไหน

ตัวอย่าง:

```text
C:\Users\pawor\Desktop\P711Test\P711_Seed\data.yaml
```

ไฟล์นี้บอก:

```text
Dataset Path
Train Path
Validation Path
Class Names
```

---

# 10. `epochs=100`

```python id="b2n6v9"
epochs=100
```

กำหนดจำนวนรอบการ Train:

```text
100 Epochs
```

### Epoch คืออะไร?

หนึ่ง Epoch คือการให้ Model เห็น Training Dataset ครบหนึ่งรอบ

ดังนั้น:

```text
Epoch 1
Epoch 2
Epoch 3
...
Epoch 100
```

---

# 11. `imgsz=960`

```python id="c8f3m1"
imgsz=960
```

กำหนดขนาด Image ที่นำเข้า Model:

```text
960 × 960
```

โดย Ultralytics จะจัดการ Resize / Letterbox ตาม Pipeline ของมัน

### ผลของ `imgsz`

ขนาดใหญ่:

```text
960
```

มักช่วยเรื่อง Object ขนาดเล็ก แต่ใช้:

* GPU Memory มากขึ้น
* Training ช้าลง

---

# 12. `batch=8`

```python id="d5h9q2"
batch=8
```

หมายถึงในแต่ละ Training Step ใช้:

```text
8 Images
```

ตัวอย่าง:

```text
Batch 1 → Image 1-8
Batch 2 → Image 9-16
Batch 3 → Image 17-24
```

ถ้า GPU Memory ไม่พอ อาจเกิด:

```text
CUDA out of memory
```

---

# 13. `device=0`

```python id="e7k2w5"
device=0
```

หมายถึงใช้ GPU หมายเลข:

```text
GPU 0
```

ถ้ามี GPU หลายตัว:

```text
0 → GPU ตัวแรก
1 → GPU ตัวที่สอง
```

ถ้าไม่มี GPU ที่เหมาะสม การกำหนด `device=0` อาจทำให้ Train ไม่สำเร็จ

---

# 14. `name`

```python id="r3m8v6"
name=f"{DATASET.name}_Seed"
```

`DATASET.name` คือ:

```text
P711Test
```

ดังนั้น:

```python
f"{DATASET.name}_Seed"
```

จะได้:

```text
P711Test_Seed
```

ชื่อ Run ที่สร้างขึ้นจึงเป็น:

```text
P711Test_Seed
```

---

# 15. `verbose=False`

```python id="u9x4c7"
verbose=False
```

ลดรายละเอียดที่แสดงระหว่าง Training

แทนที่จะพิมพ์รายละเอียดทุกอย่างออกทาง Console

---

# 16. ผลลัพธ์จาก Training

หลัง Train เสร็จ:

```python id="k6p2s8"
results
```

จะมีข้อมูลของ Training Run

จากนั้นใช้:

```python id="z5n7a3"
results.save_dir
```

เพื่อหา Folder ที่เก็บผลลัพธ์

โดยทั่วไปจะอยู่ประมาณ:

```text
runs/detect/P711Test_Seed/
```

---

# 17. `best.pt`

```python id="q8v3m5"
Path(results.save_dir) / "weights" / "best.pt"
```

จะชี้ไปยัง:

```text
runs/detect/P711Test_Seed/
└── weights/
    └── best.pt
```

### `best.pt`

คือ Checkpoint ที่มี Performance ดีที่สุดตาม Metric ที่ใช้ระหว่าง Training

โดยทั่วไปจะใช้ `best.pt` สำหรับนำ Model ไป:

* Predict
* Validate
* Test
* Deploy

---

# 18. `last.pt`

ระหว่าง Training โดยทั่วไปจะมี:

```text
weights/
├── best.pt
└── last.pt
```

### `best.pt`

Model ที่ดีที่สุดระหว่างการ Train

### `last.pt`

Model จาก Epoch สุดท้าย

ตัวอย่าง:

```text
Epoch 73 → Best
Epoch 100 → Last
```

อาจได้:

```text
best.pt → Epoch 73
last.pt → Epoch 100
```

---

# 19. แสดง Train Summary

```python id="s4d9f1"
print("\n===== TRAIN SUMMARY =====")
print(
    "Model :",
    Path(results.save_dir)
    / "weights"
    / "best.pt"
)
print("=========================")
```

ตัวอย่าง:

```text
===== TRAIN SUMMARY =====
Model : C:\...\runs\detect\P711Test_Seed\weights\best.pt
=========================
```

ทำให้รู้ทันทีว่า `best.pt` อยู่ที่ไหน

---

# 20. โครงสร้างผลลัพธ์

หลัง Train เสร็จ จะได้ประมาณ:

```text
P711Test/
│
├── P711_Seed/
│   ├── images/
│   ├── labels/
│   └── data.yaml
│
└── ...
```

และโดยทั่วไปผลการ Train จะอยู่แยกใน:

```text
runs/
└── detect/
    └── P711Test_Seed/
        │
        ├── weights/
        │   ├── best.pt
        │   └── last.pt
        │
        ├── results.csv
        ├── results.png
        ├── confusion_matrix.png
        └── ...
```

---

# 21. ความสัมพันธ์กับ STEP ก่อนหน้า

## STEP 2.5

เลือก Empty:

```text
Images ไม่มี Label
        ↓
เลือก Empty
        ↓
P711_Empty_Selection.txt
```

## STEP 3.0

สร้าง Seed Dataset:

```text
Labeled Images
      +
Empty Images
      ↓
P711_Seed/
├── images/
└── labels/
```

## STEP 4.0

สร้าง Configuration:

```text
P711_Seed/
└── data.yaml
```

## STEP 5.0

Train:

```text
P711_Seed
    +
yolo11n.pt
    ↓
Training
    ↓
best.pt
```

---

# 22. Pipeline รวม

```text
STEP 1
CHECK DATASET
       ↓
ตรวจ Image / Label / Class
       ↓
STEP 2.0
VISUALIZE LABEL
       ↓
ตรวจ Bounding Box
       ↓
STEP 2.5
SELECT EMPTY
       ↓
เลือก Negative / Empty Images
       ↓
STEP 3.0
BUILD SEED
       ↓
Labeled + Empty
       ↓
STEP 4.0
BUILD DATA.YAML
       ↓
Dataset Configuration
       ↓
STEP 5.0
TRAIN SEED MODEL
       ↓
YOLO Training
       ↓
best.pt
```

---

# 23. จุดสำคัญของ STEP 5.0

ค่าที่กำหนดในโค้ดคือ:

| Parameter |                   ค่า | ความหมาย          |
| --------- | --------------------: | ----------------- |
| `epochs`  |                 `100` | Train 100 รอบ     |
| `imgsz`   |                 `960` | Input Size        |
| `batch`   |                   `8` | 8 Images / Batch  |
| `device`  |                   `0` | ใช้ GPU 0         |
| `name`    |       `P711Test_Seed` | ชื่อ Training Run |
| `data`    | `P711_Seed/data.yaml` | Dataset Config    |

---

# 24. สรุป

**STEP 5.0 = Train Seed Model**

หน้าที่หลัก:

```text
P711_Seed/data.yaml
        +
yolo11n.pt
        ↓
model.train()
        ↓
100 Epochs
        ↓
best.pt
```

ผลลัพธ์สำคัญที่สุดคือ:

```text
runs/detect/P711Test_Seed/weights/best.pt
```

> **STEP 5.0 คือขั้นตอนที่เปลี่ยน Seed Dataset ให้กลายเป็น YOLO Model รุ่นแรก (`best.pt`) เพื่อนำไปใช้ตรวจ/ทำนายกับข้อมูลชุดต่อไป**
