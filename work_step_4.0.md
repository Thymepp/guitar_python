# 4.0 Build `data.yaml`

## 1. วัตถุประสงค์

โค้ดนี้ใช้สำหรับสร้างไฟล์:

```text
P711_Seed/data.yaml
```

โดยนำข้อมูล Class จาก YAML เดิมของ Dataset มาใช้

ไฟล์ `data.yaml` จะบอก YOLO ว่า:

* Dataset อยู่ที่ไหน
* Training Images อยู่ที่ไหน
* Validation Images อยู่ที่ไหน
* Dataset มี Class อะไรบ้าง

---

# 2. Flow การทำงาน

```text
P711Test
   │
   ├── data.yaml เดิม
   │       │
   │       └── names
   │
   ▼
อ่าน Class names
   │
   ▼
สร้าง P711_Seed/data.yaml
   │
   ├── path
   ├── train
   ├── val
   └── names
```

---

# 3. Import Library

```python
from pathlib import Path
import yaml
```

### `Path`

ใช้จัดการ Path ของ Dataset

### `yaml`

ใช้:

* อ่าน YAML เดิม
* สร้าง YAML ใหม่

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Dataset หลัก

---

# 5. กำหนด Seed

```python
SEED = DATASET / "P711_Seed"
```

จะได้:

```text
P711Test/
└── P711_Seed/
```

ดังนั้นไฟล์ที่ต้องการสร้างคือ:

```text
P711_Seed/data.yaml
```

---

# 6. เตรียมตัวแปร Class

```python
names = None
```

เริ่มต้นยังไม่มีข้อมูล Class

ภายหลังจะเก็บข้อมูลจาก YAML เช่น:

```python
names = [
    "person",
    "helmet",
    "car"
]
```

หรือในบาง Dataset อาจเป็น Dictionary:

```python
names = {
    0: "person",
    1: "helmet",
    2: "car"
}
```

---

# 7. ค้นหา YAML ของ Dataset

```python
for f in DATASET.glob("*.yaml"):
```

ใช้:

```python
.glob("*.yaml")
```

เพื่อค้นหาไฟล์ `.yaml` ที่อยู่ **โดยตรงใน `DATASET`**

ตัวอย่าง:

```text
P711Test/
├── data.yaml
└── P711_Seed/
```

จะเจอ:

```text
P711Test/data.yaml
```

### ต่างจาก `rglob()`

```python
DATASET.glob("*.yaml")
```

ค้นหาเฉพาะ Folder หลัก

ส่วน:

```python
DATASET.rglob("*.yaml")
```

จะค้นหา Folder ย่อยด้วย

---

# 8. อ่าน YAML

```python
data = yaml.safe_load(
    f.read_text(encoding="utf-8")
)
```

อ่านไฟล์ YAML และแปลงเป็น Python Object

ตัวอย่าง `data.yaml`:

```yaml
path: C:\Users\pawor\Desktop\P711Test
train: images
val: images

names:
  0: person
  1: helmet
  2: car
```

หลังอ่านแล้ว `data` จะมีข้อมูลประมาณ:

```python
{
    "path": "...",
    "train": "images",
    "val": "images",
    "names": {
        0: "person",
        1: "helmet",
        2: "car"
    }
}
```

---

# 9. ตรวจว่ามี `names` หรือไม่

```python
if "names" in data:
```

ตรวจสอบว่า YAML มี Key:

```text
names
```

หรือไม่

ถ้ามี:

```python
names = data["names"]
```

นำ Class names มาเก็บไว้

---

# 10. หยุดเมื่อเจอ YAML ที่มี Class

```python
break
```

เมื่อพบ YAML ที่มี:

```yaml
names:
```

แล้ว จะหยุด Loop ทันที

ดังนั้นไม่ต้องอ่าน YAML อื่นต่อ

---

# 11. Error Handling

```python
except:
    pass
```

ถ้าอ่าน YAML ไม่สำเร็จ จะข้ามไฟล์นั้น

แล้วไปลองไฟล์ถัดไป

---

# 12. ตรวจว่าเจอ Class หรือไม่

```python
if names is None:
    raise RuntimeError(
        "ไม่พบ Class names ใน data.yaml"
    )
```

ถ้าไม่พบ `names`

โปรแกรมจะหยุดและแจ้ง:

```text
RuntimeError:
ไม่พบ Class names ใน data.yaml
```

เพื่อป้องกันการสร้าง `data.yaml` ที่ไม่มี Class

---

# 13. สร้างข้อมูลสำหรับ Seed YAML

```python
data = {
    "path": str(SEED),
    "train": "images",
    "val": "images",
    "names": names
}
```

สร้าง Dictionary ใหม่สำหรับ:

```text
P711_Seed/data.yaml
```

ประกอบด้วย 4 ส่วนหลัก

---

# 14. `path`

```python
"path": str(SEED)
```

กำหนดตำแหน่ง Dataset

ตัวอย่าง:

```yaml
path: C:\Users\pawor\Desktop\P711Test\P711_Seed
```

---

# 15. `train`

```python
"train": "images"
```

หมายความว่า Training Images อยู่ที่:

```text
P711_Seed/images/
```

เมื่อรวมกับ:

```yaml
path:
```

จะได้:

```text
P711_Seed/images/
```

---

# 16. `val`

```python
"val": "images"
```

กำหนด Validation Dataset ให้ใช้:

```text
P711_Seed/images/
```

เช่นเดียวกับ Training

ดังนั้นตอนนี้:

```text
train → images/
val   → images/
```

---

# 17. `names`

```python
"names": names
```

นำ Class จาก Dataset เดิมมาใช้

ตัวอย่าง:

```yaml
names:
  0: person
  1: helmet
  2: car
```

ทำให้ Class ID ใน Seed Dataset ตรงกับ Dataset เดิม

---

# 18. สร้าง `data.yaml`

```python
with open(
    SEED / "data.yaml",
    "w",
    encoding="utf-8"
) as f:
```

เปิด:

```text
P711_Seed/data.yaml
```

ด้วยโหมด:

```text
w = write
```

ถ้ามีไฟล์เดิมอยู่แล้ว จะเขียนทับ

---

# 19. เขียน YAML

```python
yaml.dump(
    data,
    f,
    allow_unicode=True,
    sort_keys=False
)
```

### `data`

ข้อมูลที่ต้องการเขียน

### `f`

ไฟล์ปลายทาง

### `allow_unicode=True`

อนุญาตให้เขียนภาษา Unicode เช่นภาษาไทย

### `sort_keys=False`

รักษาลำดับ Key ตามที่กำหนดไว้

ดังนั้นจะได้ลำดับ:

```yaml
path:
train:
val:
names:
```

แทนที่จะเรียงตามตัวอักษร

---

# 20. ตัวอย่าง `data.yaml` ที่ได้

สมมติ:

```text
P711_Seed
```

อยู่ที่:

```text
C:\Users\pawor\Desktop\P711Test\P711_Seed
```

ไฟล์จะมีลักษณะ:

```yaml
path: C:\Users\pawor\Desktop\P711Test\P711_Seed
train: images
val: images
names:
  0: person
  1: helmet
  2: car
```

---

# 21. โครงสร้าง Seed หลัง STEP 4.0

```text
P711_Seed/
│
├── images/
│   ├── P711_001.jpg
│   ├── P711_002.jpg
│   ├── P711_010.jpg
│   └── ...
│
├── labels/
│   ├── P711_001.txt
│   ├── P711_002.txt
│   └── ...
│
└── data.yaml
```

---

# 22. ทำไมต้องมี `data.yaml`

YOLO ต้องรู้ว่า Dataset มีโครงสร้างอย่างไร

`data.yaml` ทำหน้าที่เป็น **Dataset Configuration**

บอก:

```text
Dataset อยู่ที่ไหน
       ↓
Train อยู่ที่ไหน
       ↓
Validation อยู่ที่ไหน
       ↓
มี Class อะไรบ้าง
```

---

# 23. ความสัมพันธ์กับ STEP 3.0

### STEP 3.0

สร้าง:

```text
P711_Seed/
├── images/
└── labels/
```

### STEP 4.0

เพิ่ม:

```text
P711_Seed/
├── images/
├── labels/
└── data.yaml
```

ดังนั้น:

```text
STEP 3.0
สร้าง Dataset
       ↓
STEP 4.0
สร้าง Dataset Configuration
```

---

# 24. จุดสำคัญของ `names`

Class ID ต้องสอดคล้องกับ Label

ตัวอย่าง:

```yaml
names:
  0: person
  1: helmet
  2: car
```

ถ้า Label มี:

```text
0 0.5 0.5 0.2 0.3
```

หมายความว่า:

```text
Class ID = 0
       ↓
person
```

ถ้า:

```text
1 0.4 0.5 0.2 0.3
```

หมายถึง:

```text
Class ID = 1
       ↓
helmet
```

ดังนั้นการนำ `names` จาก Dataset เดิมมาใช้ ช่วยรักษา Mapping ของ Class ID

---

# 25. จุดที่ต้องระวัง: `val: images`

โค้ดนี้กำหนด:

```yaml
train: images
val: images
```

หมายความว่า **Train และ Validation ชี้ไปยัง Folder เดียวกัน**

```text
train ──┐
        ├──> images/
val ────┘
```

ดังนั้นนี่เหมาะสำหรับ **Seed / ขั้นตอนเริ่มต้นที่ต้องการ Dataset เดียว** แต่ไม่ใช่การแบ่ง Train/Validation แบบมาตรฐาน เพราะภาพชุดเดียวกันจะถูกใช้ทั้งสองบทบาท

ถ้าภายหลังต้องการ Train จริงแบบแยก Validation ควรมีโครงสร้างประมาณ:

```text
P711_Seed/
├── images/
│   ├── train/
│   └── val/
│
├── labels/
│   ├── train/
│   └── val/
│
└── data.yaml
```

และ:

```yaml
train: images/train
val: images/val
```

---

# 26. แสดงผล

```python
print("\n=================================")
print("STEP 4 — BUILD DATA.YAML")
print("=================================")
print("Seed  :", SEED)
print("Class :", names)
print("=================================")
```

ตัวอย่าง:

```text
=================================
STEP 4 — BUILD DATA.YAML
=================================
Seed  : C:\Users\pawor\Desktop\P711Test\P711_Seed
Class : {0: 'person', 1: 'helmet', 2: 'car'}
=================================
```

---

# 27. Flow รวม STEP 1 → STEP 4

```text
STEP 1
CHECK DATASET
    │
    ├── ตรวจ Images
    ├── ตรวจ Labels
    ├── นับ Objects
    └── ตรวจ Classes
    │
    ▼
STEP 2.0
VISUALIZE LABELS
    │
    └── ตรวจ Bounding Box ด้วยภาพ
    │
    ▼
STEP 2.5
SELECT EMPTY IMAGES
    │
    └── เลือกภาพที่เป็น Empty
    │
    ▼
STEP 3.0
BUILD SEED DATASET
    │
    ├── Labeled Images
    └── Empty Images
    │
    ▼
STEP 4.0
BUILD DATA.YAML
    │
    └── Dataset Configuration
    │
    ▼
P711_Seed
├── images/
├── labels/
└── data.yaml
```

---

# 28. สรุป STEP 4.0

**STEP 4.0 = สร้าง Dataset Configuration**

หน้าที่หลักคือ:

```text
หา Class จาก YAML เดิม
        ↓
สร้าง data Dictionary
        ↓
กำหนด Dataset Path
        ↓
กำหนด Train
        ↓
กำหนด Validation
        ↓
กำหนด Class Names
        ↓
เขียน P711_Seed/data.yaml
```

ผลลัพธ์สุดท้าย:

```text
P711_Seed/
├── images/
├── labels/
└── data.yaml
```

> **STEP 3.0 สร้างข้อมูล Dataset ส่วน STEP 4.0 สร้างไฟล์ Configuration เพื่อบอก YOLO ว่าจะใช้ Dataset นั้นอย่างไร**
