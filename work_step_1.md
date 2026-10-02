# P711 Dataset Checker

สคริปต์นี้ใช้สำหรับ **ตรวจสอบ Dataset สำหรับงาน Object Detection** โดยตรวจสอบว่า

* มี Image กี่ไฟล์
* มี Label กี่ไฟล์
* Image กับ Label จับคู่กันหรือไม่
* มี Object ทั้งหมดกี่ตัว
* มี Class อะไรบ้าง
* แต่ละ Class มี Object กี่ตัว
* มี Image ที่ไม่มี Label กี่รูป
* อ่านชื่อ Class จากไฟล์ `.yaml` / `.yml` ของ CVAT

---

## 1. Import Library

```python
from pathlib import Path
import yaml
```

### `Path`

ใช้จัดการ Path และค้นหาไฟล์ใน Folder

### `yaml`

ใช้สำหรับอ่านไฟล์ `.yaml` / `.yml`

---

# STEP 1: กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Folder หลักของ Dataset

ตัว `r` ด้านหน้า String ทำให้ Windows Path สามารถใช้ `\` ได้โดยไม่ต้อง escape

---

# STEP 2: หา Image

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
]
```

ใช้:

```python
DATASET.rglob("*")
```

เพื่อค้นหาไฟล์และ Folder **ทุกระดับ**

จากนั้นตรวจ Extension:

```python
p.suffix.lower()
```

ว่าตรงกับ:

```text
.jpg
.jpeg
.png
```

ผลลัพธ์ `images` จะเป็น List ของ Path ของรูปทั้งหมด

ตัวอย่าง:

```text
[
    P711Test/images/001.jpg,
    P711Test/images/002.jpg,
    P711Test/images/003.png
]
```

---

# STEP 3: สร้างชื่อ Image สำหรับจับคู่ Label

```python
image_names = {p.stem for p in images}
```

`stem` คือชื่อไฟล์โดย **ไม่รวม Extension**

เช่น:

```text
001.jpg → 001
002.png → 002
```

จึงได้:

```python
{
    "001",
    "002",
    "003"
}
```

ใช้ `set` เพราะต้องการตรวจสอบว่า Label มีชื่อเดียวกับ Image หรือไม่

---

# STEP 4: หา Label ที่มี Image คู่กัน

```python
labels = [
    p for p in DATASET.rglob("*.txt")
    if p.stem in image_names
]
```

ค้นหาไฟล์ `.txt` ทั้งหมด แล้วตรวจว่า:

```python
p.stem in image_names
```

หรือไม่

ตัวอย่าง:

```text
001.jpg
001.txt
```

ถือว่าเป็นคู่กัน

แต่:

```text
002.jpg
003.txt
```

ไม่ถือว่าเป็นคู่กัน

ดังนั้น `labels` จะเก็บเฉพาะ Label ที่มี Image ชื่อเดียวกัน

---

# STEP 5: อ่านชื่อ Class จาก YAML

```python
class_names = {}
```

สร้าง Dictionary สำหรับเก็บ:

```text
Class ID → Class Name
```

เช่น:

```python
{
    0: "person",
    1: "car",
    2: "truck"
}
```

---

## ค้นหา YAML

```python
for file in list(DATASET.rglob("*.yaml")) + list(DATASET.rglob("*.yml")):
```

ค้นหาไฟล์ทั้ง:

```text
.yaml
.yml
```

---

## อ่าน YAML

```python
data = yaml.safe_load(
    file.read_text(encoding="utf-8")
)
```

อ่าน YAML แล้วแปลงเป็น Python Dictionary

---

# STEP 6: อ่าน `names`

```python
names = data.get("names")
```

สมมติ YAML มี:

```yaml
names:
  0: person
  1: car
  2: truck
```

จะได้:

```python
names = {
    0: "person",
    1: "car",
    2: "truck"
}
```

---

## กรณี `names` เป็น Dictionary

```python
if isinstance(names, dict):
    class_names = {
        int(k): v for k, v in names.items()
    }
```

แปลง Key ให้เป็น `int`

จาก:

```python
{
    "0": "person",
    "1": "car"
}
```

เป็น:

```python
{
    0: "person",
    1: "car"
}
```

---

## กรณี `names` เป็น List

```python
elif isinstance(names, list):
    class_names = dict(enumerate(names))
```

ถ้า YAML เป็น:

```yaml
names:
  - person
  - car
  - truck
```

`enumerate()` จะสร้าง:

```python
{
    0: "person",
    1: "car",
    2: "truck"
}
```

---

# STEP 7: ตรวจสอบ Label

สร้างตัวแปร:

```python
class_count = {}
objects = 0
empty = 0
```

ใช้สำหรับ:

| Variable      | ความหมาย                    |
| ------------- | --------------------------- |
| `class_count` | จำนวน Object ของแต่ละ Class |
| `objects`     | จำนวน Object ทั้งหมด        |
| `empty`       | จำนวน Label ที่ไม่มีข้อมูล  |

---

# STEP 8: อ่าน Label ทีละไฟล์

```python
for label in labels:
```

วนอ่าน Label ทุกไฟล์

---

## อ่านแต่ละบรรทัด

```python
lines = [
    x.strip()
    for x in label.read_text(
        encoding="utf-8-sig"
    ).splitlines()
    if x.strip()
]
```

ทำ 3 อย่าง:

### 1. อ่านไฟล์

```python
label.read_text(...)
```

### 2. แยกเป็นบรรทัด

```python
.splitlines()
```

### 3. ตัดช่องว่าง

```python
x.strip()
```

และ:

```python
if x.strip()
```

จะเอาเฉพาะบรรทัดที่ไม่ว่าง

---

# STEP 9: ตรวจ Empty Label

```python
if not lines:
    empty += 1
    continue
```

ถ้า Label ไม่มีข้อมูล:

```text
001.txt
```

แต่ข้างในว่าง:

```text
(empty)
```

จะเพิ่ม:

```python
empty += 1
```

แล้วข้ามไป Label ถัดไปด้วย:

```python
continue
```

---

# STEP 10: อ่าน Class ID

Label ของ YOLO มักมีรูปแบบ:

```text
class_id x_center y_center width height
```

ตัวอย่าง:

```text
0 0.512 0.432 0.200 0.300
```

โค้ด:

```python
class_id = int(line.split()[0])
```

ทำงานดังนี้:

```python
line.split()
```

ได้:

```python
["0", "0.512", "0.432", "0.200", "0.300"]
```

เลือกตัวแรก:

```python
[0]
```

ได้:

```text
"0"
```

แล้วแปลงเป็น:

```python
int("0")
```

ได้:

```text
0
```

---

# STEP 11: นับจำนวน Object ต่อ Class

```python
class_count[class_id] = (
    class_count.get(class_id, 0) + 1
)
```

เป็น Pattern สำหรับ **นับจำนวน**

ตัวอย่าง:

```python
class_count = {}
```

เจอ Class `0` ครั้งแรก:

```python
class_count.get(0, 0)
```

ได้ `0`

จึง:

```python
class_count[0] = 0 + 1
```

ผล:

```python
{
    0: 1
}
```

ถ้าเจอ Class `0` อีกครั้ง:

```python
class_count[0] = 1 + 1
```

ผล:

```python
{
    0: 2
}
```

---

# STEP 12: นับ Object ทั้งหมด

```python
objects += 1
```

ทุกครั้งที่เจอ Label Object ที่ถูกต้อง จะเพิ่มจำนวน Object ทั้งหมด 1

เช่น:

```text
Class 0 → 3 objects
Class 1 → 5 objects
Class 2 → 2 objects
```

จะได้:

```text
Objects = 10
```

---

# STEP 13: แสดงผล

```python
print(f"Images : {len(images)}")
print(f"Labels : {len(labels)}")
print(f"Objects : {objects}")
print(f"Classes : {len(class_count)}")
```

แสดง:

```text
Images  : จำนวนรูป
Labels  : จำนวน Label ที่จับคู่กับรูปได้
Objects : จำนวน Object ทั้งหมด
Classes : จำนวน Class ที่พบใน Label
```

---

# STEP 14: แสดงจำนวน Object ของแต่ละ Class

```python
for class_id, count in sorted(class_count.items()):
```

`class_count.items()` ได้:

```python
class_id, count
```

เช่น:

```python
0, 150
1, 80
2, 30
```

`sorted()` ทำให้เรียงตาม Class ID

---

## แปลง Class ID เป็นชื่อ

```python
name = class_names.get(
    class_id,
    f"Class {class_id}"
)
```

ถ้ามีชื่อใน YAML:

```python
0 → person
```

จะแสดง:

```text
person
```

ถ้าไม่มีชื่อ:

```text
Class 5
```

---

# STEP 15: แสดง Image ที่ไม่มี Label

```python
print(
    f"Images without label : {len(images) - len(labels)}"
)
```

คำนวณ:

```text
จำนวน Image ทั้งหมด - จำนวน Label ที่จับคู่ได้
```

ตัวอย่าง:

```text
Images = 100
Labels = 95
```

จะได้:

```text
Images without label = 5
```

---

# ภาพรวมการทำงาน

```text
Dataset
   │
   ├── หา Images
   │      └── .jpg / .jpeg / .png
   │
   ├── เก็บ Image Stem
   │      └── 001.jpg → 001
   │
   ├── หา Labels
   │      └── .txt ที่มีชื่อเดียวกับ Image
   │
   ├── อ่าน YAML
   │      └── Class ID → Class Name
   │
   ├── อ่าน Label
   │      ├── ตรวจ Empty
   │      ├── อ่าน Class ID
   │      └── นับ Object
   │
   └── แสดง Report
          ├── Images
          ├── Labels
          ├── Objects
          ├── Classes
          ├── จำนวนแต่ละ Class
          └── Images without label
```

---

# ตัวอย่าง Output

```text
=============================================
P711 DATASET CHECK
=============================================

Images : 100
Labels : 95
Objects : 523
Classes : 3

Class:
  person : 250
  car : 180
  truck : 93

Images without label : 5

=============================================
```

---

# สรุปสั้น ๆ

Code นี้มีหน้าที่หลัก 4 เรื่อง:

```text
1. Find
   หา Image และ Label

2. Match
   ตรวจ Image ↔ Label

3. Count
   นับ Object และ Class

4. Report
   แสดงสรุป Dataset
```

โดยใช้ Library หลัก:

```python
pathlib → จัดการไฟล์และ Folder
yaml    → อ่าน Class จาก YAML
```

และ Python Concept สำคัญที่ใช้คือ:

```python
rglob()          # ค้นหาไฟล์ทุกระดับ
suffix           # นามสกุลไฟล์
stem             # ชื่อไฟล์ไม่รวม extension
set              # เก็บชื่อ Image
dict.get()       # ใช้สำหรับนับจำนวน
splitlines()     # แยกข้อความเป็นบรรทัด
split()          # แยกข้อมูลใน Label
isinstance()     # ตรวจชนิดข้อมูล
enumerate()      # สร้าง index
sorted()         # เรียงข้อมูล
continue         # ข้ามรอบปัจจุบัน
```
