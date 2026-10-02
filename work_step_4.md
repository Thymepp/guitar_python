# WORK STEP 4 — BUILD DATA.YAML

## 1. Import Library

```python
from pathlib import Path
import yaml
```

Step นี้ใช้ Library 2 ตัว:

### `Path`

```python
from pathlib import Path
```

ใช้สำหรับจัดการ Path ของไฟล์และ Folder

---

### `yaml`

```python
import yaml
```

ใช้สำหรับอ่านและเขียนไฟล์ `.yaml`

ใน Step นี้ใช้:

```python
yaml.safe_load()
```

สำหรับอ่าน YAML

และ:

```python
yaml.dump()
```

สำหรับสร้าง YAML

---

# 2. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset

```text
C:\Users\pawor\Desktop\P711Test
```

ถ้าต้องการใช้ Dataset อื่น เปลี่ยนเฉพาะบรรทัดนี้

```python
DATASET = Path(r"...")
```

---

# 3. กำหนดตำแหน่ง Seed

```python
SEED = DATASET / "P711_Seed"
```

สร้าง Path สำหรับ Folder `P711_Seed`

ถ้า:

```text
DATASET =
C:\Users\pawor\Desktop\P711Test
```

จะได้:

```text
SEED =
C:\Users\pawor\Desktop\P711Test\P711_Seed
```

เครื่องหมาย `/` ของ `Path` ใช้สำหรับต่อ Path

ดังนั้น:

```python
DATASET / "P711_Seed"
```

หมายถึง:

```text
P711Test\P711_Seed
```

---

# 4. เตรียมตัวแปรสำหรับ Class Names

```python
names = None
```

กำหนดค่าเริ่มต้นของ `names` เป็น:

```python
None
```

หมายความว่า:

> ตอนนี้ยังไม่พบ Class names

ภายหลังถ้าเจอ `names` ใน YAML จะเปลี่ยนค่าเป็นข้อมูล Class

ตัวอย่าง:

```yaml
names:
  - person
  - car
  - bus
```

จะได้:

```python
names = ["person", "car", "bus"]
```

---

# 5. ค้นหา YAML ของ Dataset

```python
for f in DATASET.glob("*.yaml"):
```

ใช้ `glob("*.yaml")` เพื่อค้นหาไฟล์ `.yaml` ที่อยู่ **โดยตรงใน Dataset**

ตัวอย่าง:

```text
P711Test/
├── data.yaml
├── classes.yaml
├── images/
└── labels/
```

จะค้นหา:

```text
data.yaml
classes.yaml
```

แต่จะไม่ค้นหา YAML ที่อยู่ใน Subfolder

เพราะใช้:

```python
glob()
```

ไม่ใช่:

```python
rglob()
```

---

# 6. อ่าน YAML

```python
try:
    data = yaml.safe_load(
        f.read_text(encoding="utf-8")
    )
```

อ่านข้อมูลจาก YAML

## `f.read_text()`

```python
f.read_text(encoding="utf-8")
```

อ่านข้อความจากไฟล์ YAML

---

## `yaml.safe_load()`

```python
yaml.safe_load(...)
```

แปลง YAML Text ให้เป็น Python Object

ตัวอย่าง YAML:

```yaml
names:
  - person
  - car
  - bus
```

หลัง `safe_load()` จะได้ประมาณ:

```python
{
    "names": [
        "person",
        "car",
        "bus"
    ]
}
```

---

# 7. ตรวจว่ามี `names` หรือไม่

```python
if "names" in data:
```

ตรวจว่า Dictionary ที่อ่านจาก YAML มี Key:

```text
names
```

หรือไม่

ตัวอย่าง:

```yaml
path: dataset
train: images/train
val: images/val
names:
  - person
  - car
```

จะพบ:

```python
"names" in data
```

เป็น:

```text
True
```

---

# 8. เก็บ Class Names

```python
names = data["names"]
```

นำ Class Names จาก YAML มาเก็บในตัวแปร:

```python
names
```

ตัวอย่าง:

```yaml
names:
  - person
  - car
  - bus
```

จะได้:

```python
names = [
    "person",
    "car",
    "bus"
]
```

---

# 9. หยุดค้นหา YAML

```python
break
```

เมื่อเจอ YAML ที่มี `names` แล้ว ให้หยุด Loop ทันที

ดังนั้นไม่จำเป็นต้องอ่าน YAML ที่เหลือต่อ

Flow:

```text
ค้นหา YAML
    ↓
อ่าน YAML
    ↓
มี "names" ?
    │
    ├── ไม่พบ → ไปไฟล์ถัดไป
    │
    └── พบ → names = data["names"]
                 ↓
               break
                 ↓
              จบ Loop
```

---

# 10. จัดการ Error

```python
except:
    pass
```

ถ้า YAML ไฟล์ใดอ่านไม่ได้ หรือเกิด Error:

```python
except:
```

ให้:

```python
pass
```

คือ:

> ไม่ต้องทำอะไร แล้วไปตรวจไฟล์ YAML ถัดไป

ตัวอย่าง:

```text
file1.yaml → อ่านไม่ได้ → ข้าม
file2.yaml → อ่านได้ → ตรวจ names
file3.yaml → ไม่ต้องอ่าน เพราะเจอแล้ว
```

---

# 11. ตรวจสอบว่าเจอ Class หรือไม่

```python
if names is None:
    raise RuntimeError("ไม่พบ Class names ใน data.yaml")
```

หลังจากค้นหา YAML ทั้งหมดแล้ว ตรวจว่า `names` ยังเป็น:

```python
None
```

หรือไม่

ถ้ายังเป็น `None` แสดงว่า:

> ไม่พบ `names` ใน YAML

จึงหยุดโปรแกรมด้วย:

```python
raise RuntimeError(...)
```

และแสดงข้อความ:

```text
ไม่พบ Class names ใน data.yaml
```

---

# 12. สร้างข้อมูลสำหรับ Seed `data.yaml`

```python
data = {
    "path": str(SEED),
    "train": "images",
    "val": "images",
    "names": names
}
```

สร้าง Python Dictionary ที่จะนำไปเขียนเป็น `data.yaml`

โครงสร้างคือ:

```yaml
path: ...
train: images
val: images
names: ...
```

---

# 13. `path`

```python
"path": str(SEED)
```

กำหนดตำแหน่ง Dataset ของ Seed

ตัวอย่าง:

```text
C:\Users\pawor\Desktop\P711Test\P711_Seed
```

ใช้:

```python
str(SEED)
```

เพื่อแปลง `Path` เป็น String ก่อนนำไปเขียน YAML

---

# 14. `train`

```python
"train": "images"
```

บอกว่า Training Images อยู่ใน Folder:

```text
images
```

ภายใต้ `SEED`

ดังนั้น:

```text
P711_Seed/
└── images/
```

---

# 15. `val`

```python
"val": "images"
```

กำหนด Validation Images ให้ใช้ Folder เดียวกับ:

```text
images
```

ดังนั้นใน Step นี้:

```text
train → images
val   → images
```

หมายความว่า Train และ Validation ชี้ไปที่ Image Folder เดียวกัน

---

# 16. `names`

```python
"names": names
```

นำ Class Names ที่อ่านมาจาก Dataset เดิมมาใช้กับ Seed Dataset

ตัวอย่าง:

```python
names = [
    "person",
    "car",
    "bus"
]
```

จะถูกเขียนลง YAML

---

# 17. เปิดไฟล์ `data.yaml`

```python
with open(
    SEED / "data.yaml",
    "w",
    encoding="utf-8"
) as f:
```

สร้าง/เปิดไฟล์:

```text
P711_Seed/data.yaml
```

โหมด:

```text
w
```

หมายถึง Write

ถ้าไฟล์มีอยู่แล้ว เนื้อหาเดิมจะถูกเขียนทับ

---

# 18. เขียน Dictionary เป็น YAML

```python
yaml.dump(
    data,
    f,
    allow_unicode=True,
    sort_keys=False
)
```

แปลง Python Dictionary:

```python
data
```

ให้เป็น YAML แล้วเขียนลง:

```python
f
```

---

## `allow_unicode=True`

```python
allow_unicode=True
```

อนุญาตให้ YAML เก็บ Unicode ได้โดยตรง

เช่น Class ภาษาไทย:

```yaml
names:
  - คน
  - รถ
```

จะไม่ถูกแปลงเป็น Unicode Escape ที่อ่านยาก

---

## `sort_keys=False`

```python
sort_keys=False
```

ไม่เรียง Key ใหม่ตามตัวอักษร

ดังนั้นลำดับจะยังเป็นตามที่กำหนด:

```python
data = {
    "path": ...,
    "train": ...,
    "val": ...,
    "names": ...
}
```

ผลลัพธ์จึงยังเป็น:

```yaml
path: ...
train: images
val: images
names: ...
```

---

# 19. แสดงผลทาง CMD

```python
print("\n=================================")
print("STEP 4 — BUILD DATA.YAML")
print("=================================")
print("Seed  :", SEED)
print("Class :", names)
print("=================================")
```

ใช้แสดงผลเพื่อบอกว่า Step 4 ทำงานเสร็จแล้ว

ตัวอย่าง:

```text
=================================
STEP 4 — BUILD DATA.YAML
=================================
Seed  : C:\Users\pawor\Desktop\P711Test\P711_Seed
Class : ['person', 'car', 'bus']
=================================
```

---

# สรุปการทำงานของ WORK STEP 4

```text
Dataset
   │
   ▼
กำหนด P711_Seed
   │
   ▼
ค้นหา *.yaml
   │
   ▼
อ่าน YAML
   │
   ▼
หา "names"
   │
   ├── ไม่พบ → ข้ามไฟล์
   │
   └── พบ
        │
        ▼
   names = data["names"]
        │
        ▼
   สร้างข้อมูล Seed
        │
        ├── path  → P711_Seed
        ├── train → images
        ├── val   → images
        └── names → Class เดิม
        │
        ▼
สร้าง P711_Seed/data.yaml
        │
        ▼
แสดงผล
```

# ตัวอย่างผลลัพธ์

สมมติ Dataset เดิมมี Class:

```yaml
names:
  - person
  - car
  - bus
```

Step 4 จะสร้าง:

```text
P711_Seed/
└── data.yaml
```

โดย `data.yaml` จะมีโครงสร้างประมาณ:

```yaml
path: C:\Users\pawor\Desktop\P711Test\P711_Seed
train: images
val: images
names:
  - person
  - car
  - bus
```

## เป้าหมายของ Step 4

**สร้าง `data.yaml` สำหรับ `P711_Seed` โดยนำ Class Names จาก Dataset เดิมมาใช้ เพื่อให้ Seed Dataset สามารถนำไปใช้กับ YOLO ได้**
