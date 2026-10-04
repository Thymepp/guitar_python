# P711 Dataset Check

## 1. วัตถุประสงค์

โค้ดนี้ใช้สำหรับ **ตรวจสอบ Dataset สำหรับงาน Object Detection** โดยตรวจสอบว่า

* มีรูปภาพกี่รูป
* มี Label กี่ไฟล์
* มี Object ทั้งหมดกี่ตัว
* มี Class อะไรบ้าง
* แต่ละ Class มี Object กี่ตัว
* มีรูปภาพที่ไม่มี Label หรือไม่
* อ่านชื่อ Class จากไฟล์ `.yaml` / `.yml`

---

# 2. Import Library

```python
from pathlib import Path
import yaml
```

### `Path`

ใช้จัดการ Path และค้นหาไฟล์ในโฟลเดอร์

### `yaml`

ใช้เปิดอ่านไฟล์ `.yaml` หรือ `.yml` เช่นไฟล์ที่เก็บชื่อ Class

---

# 3. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset

ตัวอย่างโครงสร้าง:

```text
P711Test/
├── images/
├── labels/
├── data.yaml
└── ...
```

---

# 4. หา Image ทั้งหมด

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
]
```

ใช้

```python
DATASET.rglob("*")
```

เพื่อค้นหาไฟล์ทั้งหมดใน Dataset แบบ **Recursive**

หมายถึงค้นหาทั้ง Folder หลักและ Folder ย่อย

รองรับ Image:

```text
.jpg
.jpeg
.png
```

---

# 5. เก็บชื่อ Image

```python
image_names = {p.stem for p in images}
```

`.stem` คือชื่อไฟล์โดยไม่เอานามสกุล

ตัวอย่าง:

```text
P711_001.jpg
```

จะได้

```text
P711_001
```

ดังนั้น

```python
image_names
```

จะเก็บชื่อ Image เอาไว้สำหรับตรวจหา Label คู่กัน

---

# 6. หา Label

```python
labels = [
    p for p in DATASET.rglob("*.txt")
    if p.stem in image_names
]
```

ค้นหาไฟล์ `.txt`

แต่จะเอาเฉพาะ Label ที่มี Image ชื่อเดียวกัน

ตัวอย่าง:

```text
P711_001.jpg
P711_001.txt
```

ถือว่าเป็นคู่กัน

แต่ถ้ามี

```text
P711_002.txt
```

โดยไม่มี

```text
P711_002.jpg
```

จะไม่ถูกนับใน `labels`

---

# 7. อ่านชื่อ Class จาก YAML

```python
class_names = {}
```

สร้าง Dictionary สำหรับเก็บชื่อ Class

ตัวอย่าง:

```python
{
    0: "person",
    1: "car",
    2: "helmet"
}
```

---

## ค้นหา YAML

```python
for file in list(DATASET.rglob("*.yaml")) + list(DATASET.rglob("*.yml")):
```

ค้นหาไฟล์

```text
.yaml
.yml
```

ทั้งหมดใน Dataset

---

## อ่าน YAML

```python
data = yaml.safe_load(
    file.read_text(encoding="utf-8")
)
```

อ่านข้อมูลจาก YAML แล้วแปลงเป็น Python Dictionary

---

# 8. ตรวจรูปแบบ `names`

```python
names = data.get("names")
```

ดึงข้อมูลชื่อ Class จาก Key:

```yaml
names:
```

---

## กรณี `names` เป็น Dictionary

ตัวอย่าง:

```yaml
names:
  0: person
  1: car
  2: helmet
```

โค้ด:

```python
if isinstance(names, dict):
    class_names = {
        int(k): v for k, v in names.items()
    }
```

ผลลัพธ์:

```python
{
    0: "person",
    1: "car",
    2: "helmet"
}
```

---

## กรณี `names` เป็น List

ตัวอย่าง:

```yaml
names:
  - person
  - car
  - helmet
```

โค้ด:

```python
elif isinstance(names, list):
    class_names = dict(enumerate(names))
```

จะกลายเป็น:

```python
{
    0: "person",
    1: "car",
    2: "helmet"
}
```

---

# 9. ตรวจสอบ Label

สร้างตัวแปรสำหรับนับข้อมูล

```python
class_count = {}
objects = 0
empty = 0
```

ความหมาย:

| ตัวแปร        | ความหมาย                    |
| ------------- | --------------------------- |
| `class_count` | จำนวน Object ของแต่ละ Class |
| `objects`     | จำนวน Object ทั้งหมด        |
| `empty`       | จำนวน Label ที่ไม่มีข้อมูล  |

---

# 10. อ่าน Label ทีละไฟล์

```python
for label in labels:
```

วนตรวจสอบ Label ทุกไฟล์

---

## อ่านข้อมูลใน Label

```python
lines = [
    x.strip()
    for x in label.read_text(
        encoding="utf-8-sig"
    ).splitlines()
    if x.strip()
]
```

ทำงานดังนี้:

1. อ่านไฟล์ `.txt`
2. รองรับ `UTF-8 BOM`
3. แยกเป็นแต่ละบรรทัด
4. ตัดช่องว่างด้วย `.strip()`
5. ไม่เอาบรรทัดว่าง

---

# 11. ตรวจ Label ว่าง

```python
if not lines:
    empty += 1
    continue
```

ถ้า Label ไม่มีข้อมูล

ตัวอย่าง:

```text
P711_001.txt
```

แต่ภายในไม่มีอะไร

จะเพิ่ม:

```python
empty += 1
```

---

# 12. อ่าน Class ID

ตัวอย่าง Label YOLO:

```text
0 0.512 0.431 0.120 0.220
```

ค่าตัวแรกคือ

```text
0
```

ดังนั้นใช้:

```python
class_id = int(line.split()[0])
```

เพื่อดึง Class ID

---

# 13. นับจำนวน Object

```python
class_count[class_id] = (
    class_count.get(class_id, 0) + 1
)

objects += 1
```

ตัวอย่าง:

```text
0 ...
0 ...
1 ...
2 ...
0 ...
```

ผลลัพธ์:

```python
{
    0: 3,
    1: 1,
    2: 1
}
```

และ

```python
objects = 5
```

---

# 14. แสดงผล

```python
print("=" * 45)
print("P711 DATASET CHECK")
print("=" * 45)
```

สร้าง Header สำหรับแสดงผล

---

## จำนวน Image

```python
print(f"Images : {len(images)}")
```

แสดงจำนวน Image ทั้งหมด

---

## จำนวน Label

```python
print(f"Labels : {len(labels)}")
```

แสดงจำนวน Label ที่มี Image คู่กัน

---

## จำนวน Object

```python
print(f"Objects : {objects}")
```

แสดงจำนวน Object ทั้งหมดใน Dataset

---

## จำนวน Class

```python
print(f"Classes : {len(class_count)}")
```

แสดงจำนวน Class ที่พบจริงใน Label

**หมายเหตุ:** ค่านี้คือจำนวน Class ที่มี Object อยู่ใน Label ไม่ใช่จำนวน Class ทั้งหมดที่ประกาศใน YAML

---

# 15. แสดงจำนวน Object ของแต่ละ Class

```python
for class_id, count in sorted(class_count.items()):

    name = class_names.get(
        class_id,
        f"Class {class_id}"
    )

    print(f"  {name} : {count}")
```

ตัวอย่างผลลัพธ์:

```text
Class:

  person : 1250
  helmet : 980
  car : 430
```

ถ้าไม่พบชื่อ Class ใน YAML จะใช้:

```text
Class 5
```

แทน

---

# 16. ตรวจ Image ที่ไม่มี Label

```python
print(f"Images without label : {len(images) - len(labels)}")
```

คำนวณ:

```text
จำนวน Image - จำนวน Label ที่จับคู่ได้
```

ตัวอย่าง:

```text
Images = 1000
Labels = 950
```

จะได้:

```text
Images without label : 50
```

---

# 17. ตัวอย่างผลลัพธ์

```text
=============================================
P711 DATASET CHECK
=============================================
Images : 1000
Labels : 950
Objects : 3250
Classes : 3

Class:
  person : 1500
  helmet : 1200
  car : 550

Images without label : 50
=============================================
```

---

# 18. สรุปการทำงานทั้งหมด

```text
Dataset
   │
   ├── หา Image
   │      └── .jpg / .jpeg / .png
   │
   ├── เก็บชื่อ Image
   │      └── .stem
   │
   ├── หา Label
   │      └── .txt ที่มีชื่อคู่กับ Image
   │
   ├── หา YAML
   │      └── อ่านชื่อ Class
   │
   ├── อ่าน Label
   │      ├── Class ID
   │      └── จำนวน Object
   │
   └── แสดงผล
          ├── Images
          ├── Labels
          ├── Objects
          ├── Classes
          ├── จำนวนแต่ละ Class
          └── Images without label
```

---

# 19. จุดสำคัญที่ควรเข้าใจ

## `rglob("*")`

ค้นหาไฟล์ทุกชนิดในทุก Folder ย่อย

```python
DATASET.rglob("*")
```

---

## `.stem`

เอาชื่อไฟล์โดยไม่เอานามสกุล

```python
Path("cat001.jpg").stem
```

ผล:

```text
cat001
```

---

## `.suffix`

เอานามสกุลไฟล์

```python
Path("cat001.jpg").suffix
```

ผล:

```text
.jpg
```

---

## `class_count.get()`

ใช้เพิ่มจำนวน Class

```python
class_count.get(class_id, 0) + 1
```

ถ้ายังไม่เคยมี Class:

```text
0
```

ถ้ามีแล้ว:

```text
จำนวนเดิม + 1
```

---

## `class_names.get()`

ใช้ค้นหาชื่อ Class จาก Class ID

```python
class_names.get(class_id, f"Class {class_id}")
```

ถ้าพบ:

```text
0 → person
```

ถ้าไม่พบ:

```text
Class 0
```

---

# 20. สรุปสั้นที่สุด

โค้ดนี้คือ **Dataset Checker สำหรับ YOLO**

```text
Image
  ↓
หา Label ที่จับคู่กัน
  ↓
อ่าน YAML
  ↓
อ่าน Class ID จาก Label
  ↓
นับ Object
  ↓
สรุป Dataset
```

โดยผลลัพธ์หลักคือ:

```text
Images
Labels
Objects
Classes
Class แต่ละตัวมีกี่ Object
Images ที่ไม่มี Label
```
