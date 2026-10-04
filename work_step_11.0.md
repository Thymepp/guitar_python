# STEP 11.0 — SAVE & RENAME MODEL

## 1. หน้าที่ของ STEP 11.0

STEP 11.0 ใช้สำหรับ **Copy Final Model ที่ผ่านการ Train และ Test แล้ว มาเก็บไว้ในโฟลเดอร์สำหรับใช้งานจริง**

จาก

```text
runs
└── detect
    └── P711Test_Final
        └── weights
            └── best.pt
```

เปลี่ยนเป็น

```text
BTS_CountVision
└── model_count
    └── P711test.pt
```

### Flow

```text
STEP 9.0
Train Final Model
      ↓
best.pt
      ↓
STEP 10.0
Model Test
      ↓
ตรวจสอบ Model
      ↓
STEP 11.0
Save & Rename
      ↓
P711test.pt
      ↓
นำไปใช้กับระบบจริง
```

---

# 2. Import Library

```python
from pathlib import Path
import shutil
```

### `Path`

ใช้จัดการตำแหน่งไฟล์และ Folder

### `shutil`

ใช้ Copy ไฟล์

ใน STEP นี้ใช้

```python
shutil.copy2()
```

เพื่อ Copy Model พร้อม metadata ของไฟล์

---

# 3. กำหนด Dataset

```python
# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Dataset หลัก

```text
C:\Users\pawor\Desktop\P711Test
```

ข้อดีคือส่วนอื่นของ Code สามารถสร้าง Path ต่อจาก `DATASET` ได้

---

# 4. กำหนดชื่อ Model ใหม่

```python
MODEL_NAME = "P711test.pt"
```

ชื่อ Model ที่ต้องการเก็บ

จากเดิม

```text
best.pt
```

เป็น

```text
P711test.pt
```

ชื่อใหม่นี้เหมาะกับการนำไปใช้ใน Project มากกว่า `best.pt` เพราะสื่อว่าเป็น Model ของ Project `P711test`

---

# 5. ตำแหน่ง Best Model

```python
SOURCE = (
    DATASET.parent
    / "runs" / "detect"
    / f"{DATASET.name}_Final"
    / "weights"
    / "best.pt"
)
```

ส่วนนี้สร้าง Path ของ Model ต้นฉบับ

เมื่อ

```python
DATASET = C:\Users\pawor\Desktop\P711Test
```

จะได้

```text
DATASET.parent
```

เป็น

```text
C:\Users\pawor\Desktop
```

และ

```python
DATASET.name
```

เป็น

```text
P711Test
```

ดังนั้น

```python
f"{DATASET.name}_Final"
```

จะได้

```text
P711Test_Final
```

ตำแหน่งสุดท้ายคือ

```text
C:\Users\pawor\Desktop
└── runs
    └── detect
        └── P711Test_Final
            └── weights
                └── best.pt
```

---

# 6. สร้างโฟลเดอร์สำหรับ Model

```python
MODEL_DIR = (
    DATASET.parent
    / "BTS_CountVision"
    / "model_count"
)
```

กำหนด Folder สำหรับเก็บ Model

จะได้

```text
C:\Users\pawor\Desktop\BTS_CountVision\model_count
```

จากนั้น

```python
MODEL_DIR.mkdir(
    parents=True,
    exist_ok=True
)
```

ทำหน้าที่สร้าง Folder

### `parents=True`

ถ้า Folder ด้านบนยังไม่มี เช่น

```text
BTS_CountVision
```

ก็สร้างให้ด้วย

### `exist_ok=True`

ถ้า Folder มีอยู่แล้ว

```text
ไม่ Error
```

จึงสามารถรัน Script ซ้ำได้

---

# 7. กำหนดตำแหน่งปลายทาง

```python
DEST = MODEL_DIR / MODEL_NAME
```

นำ

```text
MODEL_DIR
```

มารวมกับ

```text
MODEL_NAME
```

จึงได้

```text
C:\Users\pawor\Desktop\BTS_CountVision\model_count\P711test.pt
```

ดังนั้น

```text
SOURCE
↓
best.pt

DEST
↓
P711test.pt
```

---

# 8. ตรวจสอบว่า Best Model มีอยู่จริง

```python
if not SOURCE.exists():
    raise FileNotFoundError(
        f"ไม่พบโมเดล:\n{SOURCE}"
    )
```

ก่อน Copy จะตรวจสอบก่อนว่า

```text
best.pt
```

มีอยู่จริงหรือไม่

ถ้าไม่มี จะหยุดโปรแกรมทันที

ตัวอย่าง Error

```text
FileNotFoundError:
ไม่พบโมเดล:
C:\Users\pawor\Desktop\runs\detect\P711Test_Final\weights\best.pt
```

ช่วยป้องกันการ Copy จากตำแหน่งที่ผิด

---

# 9. ตรวจสอบชื่อซ้ำ

```python
if DEST.exists():
```

ตรวจสอบว่า

```text
P711test.pt
```

มีอยู่แล้วหรือไม่

ถ้ามีจะแสดง

```python
print(f"พบโมเดลเดิม: {DEST}")
```

ตัวอย่าง

```text
พบโมเดลเดิม:
C:\Users\pawor\Desktop\BTS_CountVision\model_count\P711test.pt
```

---

# 10. ถามก่อนเขียนทับ

```python
answer = input(
    "ต้องการเขียนทับหรือไม่? (y/n): "
).lower()
```

ให้ผู้ใช้เลือก

```text
y
```

หรือ

```text
n
```

`.lower()` ทำให้

```text
Y
```

กลายเป็น

```text
y
```

ดังนั้นทั้ง

```text
Y
y
```

ถือว่าเป็นคำตอบเดียวกัน

---

# 11. กรณีไม่ต้องการเขียนทับ

```python
if answer != "y":
    print("ยกเลิก")
    raise SystemExit
```

ถ้าผู้ใช้ตอบอย่างอื่นนอกจาก

```text
y
```

Script จะยกเลิก

ตัวอย่าง

```text
พบโมเดลเดิม: ...
ต้องการเขียนทับหรือไม่? (y/n): n
ยกเลิก
```

Model เดิมจะไม่ถูกแก้ไข

---

# 12. Copy + เปลี่ยนชื่อ

```python
shutil.copy2(SOURCE, DEST)
```

นี่คือหัวใจของ STEP 11.0

ทำการ Copy

```text
best.pt
```

ไปยัง

```text
P711test.pt
```

สำคัญคือ **ไม่ได้เปลี่ยนแปลง Model ภายในไฟล์**

เป็นเพียงการ Copy ไฟล์แล้วตั้งชื่อใหม่

```text
best.pt
   │
   │ copy2()
   ↓
P711test.pt
```

ดังนั้น Model Architecture, Weights และข้อมูลภายในยังเป็น Model เดิม

---

# 13. ทำไมใช้ `copy2()`?

```python
shutil.copy2(SOURCE, DEST)
```

`copy2()` ใช้ Copy ไฟล์พร้อมพยายามรักษา metadata ของไฟล์ เช่น timestamp

เมื่อเทียบกับ

```python
shutil.copy()
```

`copy2()` เหมาะกับกรณีที่ต้องการ Copy ไฟล์ Model โดยรักษาข้อมูลไฟล์ให้ใกล้เคียงต้นฉบับมากกว่า

---

# 14. แสดงผลหลัง Copy

```python
print("\n=================================")
print("STEP 11 — SAVE & RENAME MODEL")
print("=================================")
print("Source :", SOURCE)
print("Model  :", DEST)
print("=================================")
print("บันทึกโมเดลเรียบร้อย")
```

ตัวอย่าง Output

```text
=================================
STEP 11 — SAVE & RENAME MODEL
=================================
Source : C:\Users\pawor\Desktop\runs\detect\P711Test_Final\weights\best.pt
Model  : C:\Users\pawor\Desktop\BTS_CountVision\model_count\P711test.pt
=================================
บันทึกโมเดลเรียบร้อย
```

---

# 15. โครงสร้างไฟล์หลัง STEP 11.0

หลังจากทำงานเสร็จ จะมีโครงสร้างประมาณนี้

```text
C:\Users\pawor\Desktop
│
├── P711Test
│   ├── P711_Seed
│   ├── P711_Final
│   └── ...
│
├── runs
│   └── detect
│       ├── P711Test_Seed
│       │   └── weights
│       │       ├── best.pt
│       │       └── last.pt
│       │
│       └── P711Test_Final
│           └── weights
│               ├── best.pt
│               └── last.pt
│
└── BTS_CountVision
    └── model_count
        └── P711test.pt
```

ดังนั้น

### Model สำหรับ Training

ยังอยู่ที่

```text
runs/detect/P711Test_Final/weights/best.pt
```

### Model สำหรับใช้งาน

ถูก Copy มาไว้ที่

```text
BTS_CountVision/model_count/P711test.pt
```

---

# 16. STEP 11.0 ไม่ได้ Train Model

จุดสำคัญคือ STEP นี้

**ไม่ได้ Train**

**ไม่ได้ Predict**

**ไม่ได้แก้ Weight**

แต่ทำเพียง

```text
ค้นหา
   ↓
ตรวจสอบ
   ↓
Copy
   ↓
Rename
```

หรือพูดง่าย ๆ คือ

```text
best.pt
   ↓
Copy
   ↓
P711test.pt
```

---

# 17. ทำไมต้องแยก Model สำหรับใช้งานจริง?

ในช่วง Training จะมีไฟล์จำนวนมาก เช่น

```text
runs/
└── detect/
    └── P711Test_Final/
        ├── args.yaml
        ├── results.csv
        ├── results.png
        ├── confusion_matrix.png
        ├── labels.jpg
        └── weights/
            ├── best.pt
            └── last.pt
```

แต่โปรแกรมใช้งานจริงอาจต้องการแค่

```text
P711test.pt
```

จึงแยก Model ออกจาก Training Run

ทำให้ Project ใช้งานง่ายขึ้น

---

# 18. ความหมายของ `best.pt` และ `P711test.pt`

สองไฟล์นี้มาจาก Model เดียวกัน

```text
best.pt
   │
   │ Copy + Rename
   ↓
P711test.pt
```

ดังนั้น

```text
best.pt
=
P711test.pt
```

ในแง่ของ Model Weight

แตกต่างกันเพียง **ชื่อและตำแหน่งไฟล์**

---

# 19. ความสัมพันธ์กับ STEP ก่อนหน้า

| STEP          | หน้าที่                 |
| ------------- | ----------------------- |
| STEP 8.0      | สร้าง Final Dataset     |
| STEP 8.5      | ตรวจสอบ Final Dataset   |
| STEP 9.0      | Train Final Model       |
| STEP 10.0     | Test Final Model        |
| **STEP 11.0** | **Save + Rename Model** |

ดังนั้น STEP 11.0 เป็นขั้นตอนหลังจาก Model ผ่านการ Train และ Test แล้ว

---

# 20. Pipeline ตั้งแต่ Dataset ถึง Model ใช้งาน

```text
Raw Dataset
     ↓
STEP 1
Check Dataset
     ↓
STEP 2.0
Visualize Labels
     ↓
STEP 2.5
Select Empty Images
     ↓
STEP 3.0
Build Seed
     ↓
STEP 4.0
Build Seed YAML
     ↓
STEP 5.0
Train Seed
     ↓
Seed best.pt
     ↓
STEP 6.0
Auto Label
     ↓
STEP 7.0
Check Auto Label
     ↓
STEP 8.0
Build Final
     ↓
STEP 8.5
Check Final
     ↓
STEP 9.0
Train Final
     ↓
Final best.pt
     ↓
STEP 10.0
Model Test
     ↓
STEP 11.0
Save & Rename
     ↓
P711test.pt
     ↓
BTS_CountVision
     ↓
นำ Model ไปใช้งานจริง
```

---

# 21. สรุป Code แต่ละส่วน

| Code              | หน้าที่                 |
| ----------------- | ----------------------- |
| `Path`            | จัดการ Path             |
| `shutil`          | Copy ไฟล์               |
| `DATASET`         | Dataset หลัก            |
| `MODEL_NAME`      | ชื่อ Model ใหม่         |
| `SOURCE`          | ตำแหน่ง `best.pt`       |
| `MODEL_DIR`       | Folder เก็บ Model       |
| `mkdir()`         | สร้าง Folder            |
| `DEST`            | ตำแหน่ง Model ใหม่      |
| `SOURCE.exists()` | ตรวจสอบ Model ต้นฉบับ   |
| `DEST.exists()`   | ตรวจสอบชื่อซ้ำ          |
| `input()`         | ถามว่าจะเขียนทับหรือไม่ |
| `SystemExit`      | ยกเลิกโปรแกรม           |
| `shutil.copy2()`  | Copy Model              |
| `print()`         | แสดงผล                  |

---

# 22. สรุป STEP 11.0 แบบสั้น

```text
STEP 9.0
   ↓
P711Test_Final/weights/best.pt
   ↓
STEP 10.0
   ↓
ทดสอบ Model
   ↓
STEP 11.0
   ↓
Copy + Rename
   ↓
BTS_CountVision/model_count/P711test.pt
```

**STEP 11.0 = นำ Final `best.pt` มาเก็บเป็น Model สำหรับใช้งานจริง**

โดยไฟล์

```text
P711test.pt
```

คือ Final Model ที่ผ่านขั้นตอน Training และสามารถนำไปใช้ต่อในโปรแกรม `BTS_CountVision` ได้
