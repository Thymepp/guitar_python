# STEP 11 — SAVE & RENAME MODEL

## 1. จุดประสงค์ของ Step นี้

Step 11 คือขั้นตอน **นำ Final Model ที่ Train เสร็จแล้วมาเก็บไว้ในตำแหน่งที่ต้องการ และเปลี่ยนชื่อโมเดล**

โดยโมเดลต้นทางคือ:

```text
runs/detect/P711Test_Final/weights/best.pt
```

และจะถูก Copy ไปเป็น:

```text
BTS_CountVision/model_count/P711test.pt
```

Flow:

```text
Final Training
     │
     ▼
best.pt
     │
     │ ตรวจสอบว่ามีหรือไม่
     ▼
Copy
     │
     │ เปลี่ยนชื่อ
     ▼
P711test.pt
     │
     ▼
BTS_CountVision/model_count/
```

---

# 2. โค้ดทั้งหมด

```python
from pathlib import Path
import shutil

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

# ชื่อโมเดลที่ต้องการ
MODEL_NAME = "P711test.pt"

# ตำแหน่ง Best Model
SOURCE = (
    DATASET.parent
    / "runs" / "detect"
    / f"{DATASET.name}_Final"
    / "weights"
    / "best.pt"
)

# โฟลเดอร์เก็บโมเดล
MODEL_DIR = DATASET.parent / "BTS_CountVision" / "model_count"
MODEL_DIR.mkdir(parents=True, exist_ok=True)

# ตำแหน่งปลายทาง
DEST = MODEL_DIR / MODEL_NAME

# ตรวจสอบ Model
if not SOURCE.exists():
    raise FileNotFoundError(
        f"ไม่พบโมเดล:\n{SOURCE}"
    )

# ถ้ามีชื่อซ้ำ
if DEST.exists():
    print(f"พบโมเดลเดิม: {DEST}")
    answer = input("ต้องการเขียนทับหรือไม่? (y/n): ").lower()

    if answer != "y":
        print("ยกเลิก")
        raise SystemExit

# Copy + เปลี่ยนชื่อ
shutil.copy2(SOURCE, DEST)

print("\n=================================")
print("STEP 11 — SAVE & RENAME MODEL")
print("=================================")
print("Source :", SOURCE)
print("Model  :", DEST)
print("=================================")
print("บันทึกโมเดลเรียบร้อย")
```

---

# 3. Import Library

```python
from pathlib import Path
import shutil
```

มี 2 Library หลัก

| Library  | หน้าที่                       |
| -------- | ----------------------------- |
| `Path`   | จัดการ Path ของไฟล์และ Folder |
| `shutil` | Copy ไฟล์                     |

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Dataset หลัก:

```text
C:\Users\pawor\Desktop\P711Test
```

จากตัวแปรนี้ โค้ดจะใช้หา:

* Final Model
* Folder สำหรับเก็บ Model

---

# 5. กำหนดชื่อ Model ใหม่

```python
MODEL_NAME = "P711test.pt"
```

กำหนดชื่อที่ต้องการใช้กับ Model Final

จากเดิม:

```text
best.pt
```

จะถูก Copy เป็น:

```text
P711test.pt
```

จุดนี้เป็นเพียงการ **เปลี่ยนชื่อไฟล์** ไม่ได้ Train โมเดลใหม่

---

# 6. กำหนดตำแหน่ง Source Model

```python
SOURCE = (
    DATASET.parent
    / "runs" / "detect"
    / f"{DATASET.name}_Final"
    / "weights"
    / "best.pt"
)
```

`SOURCE` คือ Path ของ Model ต้นฉบับ

จาก:

```python
DATASET
```

คือ:

```text
C:\Users\pawor\Desktop\P711Test
```

ดังนั้น:

```python
DATASET.parent
```

คือ:

```text
C:\Users\pawor\Desktop
```

และ:

```python
DATASET.name
```

คือ:

```text
P711Test
```

ดังนั้น:

```python
f"{DATASET.name}_Final"
```

จะได้:

```text
P711Test_Final
```

สุดท้าย `SOURCE` จะเป็น:

```text
C:\Users\pawor\Desktop\runs\detect\P711Test_Final\weights\best.pt
```

---

# 7. โครงสร้าง Source

โดยคาดหวังว่าไฟล์จาก Step 9 จะอยู่ในลักษณะ:

```text
Desktop/
│
├── P711Test/
│
├── runs/
│   └── detect/
│       └── P711Test_Final/
│           ├── weights/
│           │   ├── best.pt
│           │   └── last.pt
│           │
│           └── ...
│
└── ...
```

Step 11 จะเลือกเฉพาะ:

```text
best.pt
```

---

# 8. สร้าง Folder สำหรับเก็บ Model

```python
MODEL_DIR = DATASET.parent / "BTS_CountVision" / "model_count"
```

กำหนด Folder ปลายทาง:

```text
BTS_CountVision/
└── model_count/
```

โดยอยู่ใน Folder เดียวกับ `P711Test`

ตัวอย่าง:

```text
C:\Users\pawor\Desktop\
│
├── P711Test\
│
├── runs\
│
└── BTS_CountVision\
    └── model_count\
```

---

# 9. `mkdir()`

```python
MODEL_DIR.mkdir(parents=True, exist_ok=True)
```

ใช้สร้าง Folder

### `parents=True`

ถ้า Folder ชั้นบนยังไม่มี:

```text
BTS_CountVision
```

Python จะสร้างให้ด้วย

เช่น:

```text
Desktop/
└── BTS_CountVision/
    └── model_count/
```

สามารถสร้างทั้งโครงสร้างได้ในคำสั่งเดียว

### `exist_ok=True`

ถ้า Folder มีอยู่แล้ว:

```text
BTS_CountVision/model_count
```

จะไม่เกิด Error

ดังนั้นสามารถรัน Script ซ้ำได้

---

# 10. กำหนดตำแหน่งปลายทาง

```python
DEST = MODEL_DIR / MODEL_NAME
```

`MODEL_DIR`:

```text
C:\Users\pawor\Desktop\BTS_CountVision\model_count
```

`MODEL_NAME`:

```text
P711test.pt
```

ดังนั้น:

```python
DEST
```

จะเป็น:

```text
C:\Users\pawor\Desktop\BTS_CountVision\model_count\P711test.pt
```

---

# 11. ตรวจสอบว่า Source Model มีหรือไม่

```python
if not SOURCE.exists():
```

ตรวจสอบว่า:

```text
best.pt
```

มีอยู่จริงหรือไม่

---

# 12. ถ้าไม่พบ Model

```python
raise FileNotFoundError(
    f"ไม่พบโมเดล:\n{SOURCE}"
)
```

ถ้าไม่มี `best.pt` จะหยุดโปรแกรมทันที

ตัวอย่าง:

```text
FileNotFoundError:
ไม่พบโมเดล:
C:\Users\pawor\Desktop\runs\detect\P711Test_Final\weights\best.pt
```

ข้อดีคือรู้ทันทีว่า Python กำลังหา Model จากตำแหน่งใด

---

# 13. ตรวจสอบชื่อไฟล์ซ้ำ

```python
if DEST.exists():
```

ตรวจสอบว่า:

```text
P711test.pt
```

มีอยู่แล้วหรือไม่

ถ้ามี:

```text
BTS_CountVision/
└── model_count/
    └── P711test.pt
```

โค้ดจะไม่ Copy ทับทันที

แต่จะถามผู้ใช้ก่อน

---

# 14. แสดงข้อความเตือน

```python
print(f"พบโมเดลเดิม: {DEST}")
```

ตัวอย่าง:

```text
พบโมเดลเดิม:
C:\Users\pawor\Desktop\BTS_CountVision\model_count\P711test.pt
```

---

# 15. ถามว่าจะเขียนทับหรือไม่

```python
answer = input("ต้องการเขียนทับหรือไม่? (y/n): ").lower()
```

โปรแกรมรอให้ผู้ใช้ป้อน:

```text
y
```

หรือ:

```text
n
```

---

# 16. `.lower()`

```python
.lower()
```

ใช้เปลี่ยนตัวอักษรให้เป็น lowercase

ดังนั้น:

```text
Y → y
N → n
```

ทำให้ตรวจสอบง่ายขึ้น

เช่นผู้ใช้พิมพ์:

```text
Y
```

จะกลายเป็น:

```text
y
```

---

# 17. กรณีไม่ต้องการเขียนทับ

```python
if answer != "y":
    print("ยกเลิก")
    raise SystemExit
```

ถ้าผู้ใช้ตอบอะไรก็ตามที่ไม่ใช่:

```text
y
```

โปรแกรมจะยกเลิก

ตัวอย่าง:

```text
พบโมเดลเดิม: ...
ต้องการเขียนทับหรือไม่? (y/n): n
ยกเลิก
```

---

# 18. `raise SystemExit`

```python
raise SystemExit
```

ใช้หยุดการทำงานของโปรแกรม

ดังนั้นเมื่อผู้ใช้ตอบ:

```text
n
```

โปรแกรมจะไม่ไปถึงคำสั่ง Copy

---

# 19. Copy Model

```python
shutil.copy2(SOURCE, DEST)
```

นี่คือคำสั่งสำคัญของ Step 11

ทำหน้าที่:

```text
SOURCE
  │
  │ copy2()
  ▼
DEST
```

จาก:

```text
runs/detect/P711Test_Final/weights/best.pt
```

ไปเป็น:

```text
BTS_CountVision/model_count/P711test.pt
```

---

# 20. `copy2()` คืออะไร

```python
shutil.copy2()
```

ใช้ Copy ไฟล์พร้อมพยายามรักษา Metadata ของไฟล์ เช่น Timestamp ต่าง ๆ

ต่างจากแนวคิดของการสร้างไฟล์ใหม่ด้วยการอ่านแล้วเขียนเอง เพราะ `copy2()` ถูกออกแบบมาสำหรับการ Copy ไฟล์โดยตรง

---

# 21. Copy + Rename ในคำสั่งเดียว

โค้ด:

```python
shutil.copy2(SOURCE, DEST)
```

ไม่ได้มีคำสั่ง Rename แยกต่างหาก

แต่เพราะกำหนด Destination เป็นชื่อใหม่:

```python
DEST = MODEL_DIR / "P711test.pt"
```

ผลลัพธ์จึงเป็น:

```text
best.pt
   │
   │ Copy
   ▼
P711test.pt
```

จึงเรียกได้ว่า:

```text
Copy + Rename
```

---

# 22. Source กับ Destination

สามารถมองเป็นตารางได้ดังนี้:

| รายการ             | ตำแหน่ง                                      |
| ------------------ | -------------------------------------------- |
| Source             | `runs/detect/P711Test_Final/weights/best.pt` |
| Destination Folder | `BTS_CountVision/model_count`                |
| Destination Name   | `P711test.pt`                                |
| Final Model        | `BTS_CountVision/model_count/P711test.pt`    |

---

# 23. Print Summary

```python
print("\n=================================")
print("STEP 11 — SAVE & RENAME MODEL")
print("=================================")
print("Source :", SOURCE)
print("Model  :", DEST)
print("=================================")
print("บันทึกโมเดลเรียบร้อย")
```

ใช้แสดงข้อมูลหลัง Copy สำเร็จ

ตัวอย่าง:

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

# 24. โครงสร้างหลังจบ Step 11

หลังจาก Script ทำงานสำเร็จ จะได้:

```text
Desktop/
│
├── P711Test/
│
├── runs/
│   └── detect/
│       └── P711Test_Final/
│           └── weights/
│               ├── best.pt
│               └── last.pt
│
└── BTS_CountVision/
    └── model_count/
        └── P711test.pt
```

จุดสำคัญคือ:

```text
best.pt
```

ยังคงอยู่ที่ตำแหน่งเดิม

เพราะ Step 11 ใช้:

```python
shutil.copy2()
```

ไม่ใช่:

```python
shutil.move()
```

ดังนั้นเป็นการ **Copy** ไม่ใช่การย้ายไฟล์

---

# 25. `copy2()` กับ `move()` ต่างกันอย่างไร

### `copy2()`

```python
shutil.copy2(SOURCE, DEST)
```

ผล:

```text
Source      → ยังอยู่
Destination → มีไฟล์ใหม่
```

### `move()`

```python
shutil.move(SOURCE, DEST)
```

ผล:

```text
Source      → ถูกย้ายออก
Destination → มีไฟล์
```

Step 11 เลือกใช้ `copy2()` เพราะต้องการเก็บ `best.pt` ต้นฉบับไว้ด้วย

---

# 26. Flow การทำงานทั้งหมด

```text
                    STEP 11
                       │
                       ▼
          หา Final Model: best.pt
                       │
                       ▼
              SOURCE.exists() ?
                 │           │
                NO          YES
                 │           │
                 ▼           ▼
             Error       สร้าง Folder
                             │
                             ▼
                       P711test.pt
                       มีอยู่แล้ว ?
                         │       │
                        YES      NO
                         │       │
                         ▼       │
                    ถาม y/n      │
                     │   │       │
                     y   n       │
                     │   │       │
                     │   └──→ ยกเลิก
                     │
                     ▼
                  copy2()
                     │
                     ▼
              P711test.pt
```

---

# 27. ความสัมพันธ์กับ Step 9 และ Step 10

Pipeline ช่วงท้าย:

```text
STEP 9
Train Final Model
      │
      ▼
best.pt
      │
      ├──────────────┐
      │              │
      ▼              ▼
STEP 10          STEP 11
Test Model       Save Model
      │              │
      ▼              ▼
ตรวจสอบ          P711test.pt
Model
```

### Step 9

สร้าง:

```text
best.pt
```

### Step 10

ตรวจสอบว่า Model สามารถ Predict ภาพได้อย่างไร

### Step 11

นำ Model ที่ผ่านการ Train มาเก็บในตำแหน่งสำหรับใช้งาน:

```text
BTS_CountVision/model_count/P711test.pt
```

---

# 28. สรุป Step 11

| Code              | หน้าที่                    |
| ----------------- | -------------------------- |
| `Path`            | จัดการ Path                |
| `shutil`          | Copy ไฟล์                  |
| `MODEL_NAME`      | กำหนดชื่อ Model ใหม่       |
| `SOURCE`          | ตำแหน่ง `best.pt`          |
| `MODEL_DIR`       | Folder เก็บ Model          |
| `mkdir()`         | สร้าง Folder               |
| `DEST`            | ตำแหน่ง Model ใหม่         |
| `SOURCE.exists()` | ตรวจสอบว่ามี Model หรือไม่ |
| `DEST.exists()`   | ตรวจสอบชื่อซ้ำ             |
| `input()`         | ถามผู้ใช้                  |
| `.lower()`        | แปลงคำตอบเป็น lowercase    |
| `SystemExit`      | ยกเลิกโปรแกรม              |
| `shutil.copy2()`  | Copy Model                 |
| `print()`         | แสดงผลลัพธ์                |

---

# 29. ผลลัพธ์สุดท้าย

เป้าหมายของ Step 11 คือ:

```text
Final Model
     │
     ▼
best.pt
     │
     │ Copy + Rename
     ▼
P711test.pt
```

และได้ไฟล์:

```text
C:\Users\pawor\Desktop\BTS_CountVision\model_count\P711test.pt
```

**ไฟล์ `P711test.pt` คือ Final Model ที่ถูกเตรียมไว้สำหรับนำไปใช้งานต่อในโปรเจกต์ `BTS_CountVision`**
