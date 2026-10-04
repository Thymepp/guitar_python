# 3.0 Build Seed Dataset

## 1. วัตถุประสงค์

โค้ดนี้ใช้สำหรับสร้าง **Seed Dataset** จาก Dataset หลัก

โดยนำ:

1. ภาพที่มี YOLO Label จริง
2. ภาพ Empty ที่เลือกจาก `P711_Empty_Selection.txt`

มารวมกันเป็น Dataset ใหม่ที่:

```text
P711_Seed/
├── images/
└── labels/
```

---

# 2. Flow การทำงาน

```text
P711Test
   │
   ├── Images
   │
   ├── YOLO Labels
   │
   └── P711_Empty_Selection.txt
   │
   ▼
คัดแยก
   │
   ├── Labeled Images
   │
   └── Empty Images
   │
   ▼
สร้าง P711_Seed
   │
   ├── images/
   │
   └── labels/
```

---

# 3. Import Library

```python id="8q2v8a"
from pathlib import Path
import shutil
```

### `Path`

ใช้จัดการ Path ของไฟล์และ Folder

### `shutil`

ใช้สำหรับ:

* Copy ไฟล์
* ลบ Folder
* จัดการไฟล์

ในโค้ดนี้ใช้:

```python id="j6jpl2"
shutil.rmtree()
shutil.copy2()
```

---

# 4. กำหนด Dataset

```python id="yfr6p4"
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Dataset หลัก

---

# 5. กำหนด Seed Folder

```python id="t6s1ab"
SEED = DATASET / "P711_Seed"
```

ผลลัพธ์:

```text id="gr1ykn"
P711Test/
└── P711_Seed/
```

---

# 6. ลบ Seed เดิม

```python id="l0c0bq"
if SEED.exists():
    shutil.rmtree(SEED)
```

ถ้ามี `P711_Seed` อยู่แล้ว จะลบทิ้งทั้งหมด

### จุดประสงค์

ทำให้ทุกครั้งที่ Run Script:

```text
P711_Seed
```

ถูกสร้างใหม่จากข้อมูลปัจจุบัน

### ⚠️ สำคัญ

คำสั่ง:

```python id="9f8ahm"
shutil.rmtree(SEED)
```

จะลบ Folder และไฟล์ภายในทั้งหมด

---

# 7. สร้างโครงสร้าง Seed Dataset

```python id="n4rcwm"
(SEED / "images").mkdir(parents=True)
(SEED / "labels").mkdir(parents=True)
```

สร้าง:

```text id="36n1tw"
P711_Seed/
├── images/
└── labels/
```

---

# 8. อ่านรายการ Empty Images

กำหนดไฟล์:

```python id="cr9s5k"
empty_file = DATASET / "P711_Empty_Selection.txt"
```

ไฟล์นี้ถูกสร้างจาก **STEP 2.5**

---

# 9. สร้าง `empty_names`

```python id="1w9q4p"
empty_names = set()
```

ใช้เก็บชื่อของภาพที่ถูกเลือกให้เป็น Empty

ตัวอย่าง:

```python id="b9v39v"
{
    "P711_010",
    "P711_015",
    "P711_023"
}
```

---

# 10. อ่าน Empty Selection

```python id="2prkpy"
if empty_file.exists():
```

ตรวจสอบก่อนว่าไฟล์:

```text id="t5v0ke"
P711_Empty_Selection.txt
```

มีอยู่หรือไม่

---

# 11. แปลง Path เป็นชื่อ Image

```python id="gk5j6c"
empty_names = {
    Path(x).stem
    for x in empty_file.read_text(
        encoding="utf-8"
    ).splitlines()
    if x.strip()
}
```

สมมติในไฟล์มี:

```text id="2m4h5a"
C:\Users\pawor\Desktop\P711Test\P711_010.jpg
C:\Users\pawor\Desktop\P711Test\P711_015.jpg
```

`.stem` จะเปลี่ยนเป็น:

```text id="2h9xob"
P711_010
P711_015
```

---

# 12. ค้นหา Images ทั้งหมด

```python id="b9u1zi"
images = {
    p.stem: p
    for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and SEED not in p.parents
}
```

สร้าง Dictionary:

```text id="p5z5oa"
ชื่อ Image → Path
```

ตัวอย่าง:

```python id="l0x0gi"
{
    "P711_001": Path(".../P711_001.jpg"),
    "P711_002": Path(".../P711_002.jpg")
}
```

---

# 13. ป้องกันการอ่าน Image จาก Seed

```python id="uj0n8g"
and SEED not in p.parents
```

สำคัญมาก เพราะ:

```python id="yw29hs"
DATASET.rglob("*")
```

สามารถค้นเข้าไปใน:

```text id="yq0qv1"
P711_Seed/
```

ได้

เงื่อนไขนี้จึงบอกว่า:

> ไม่เอา Image ที่อยู่ภายใน `P711_Seed`

เพื่อป้องกันข้อมูลถูกนำกลับมา Copy ซ้ำ

---

# 14. สร้างรายการ Labeled

```python id="x5n4jw"
labeled = []
```

เก็บ Label ที่ผ่านเงื่อนไขว่าเป็น Label จริง

---

# 15. ค้นหา Label

```python id="g6j6k0"
for label in DATASET.rglob("*.txt"):
```

ค้นหาไฟล์ `.txt` ทั้งหมด

---

# 16. ไม่อ่าน Label ใน Seed

```python id="j6a3qz"
if SEED in label.parents:
    continue
```

ถ้า Label อยู่ใน:

```text id="9ppru7"
P711_Seed/labels/
```

จะข้ามไป

---

# 17. ไม่อ่าน Empty Selection เป็น YOLO Label

```python id="2s9jv6"
if label.name == "P711_Empty_Selection.txt":
    continue
```

เพราะไฟล์นี้:

```text id="0u7zxi"
P711_Empty_Selection.txt
```

ไม่ใช่ YOLO Label

แต่เป็นไฟล์รายชื่อ Empty Images

---

# 18. ตรวจว่า Label เป็นของ Image จริง

```python id="kg2p0v"
if label.stem in images and label.stem not in empty_names:
    labeled.append(label)
```

มี 2 เงื่อนไข:

### เงื่อนไขที่ 1

```python id="h9j9dd"
label.stem in images
```

ต้องมี Image คู่กัน

เช่น:

```text id="4f1jqu"
P711_001.jpg
P711_001.txt
```

### เงื่อนไขที่ 2

```python id="k0h4rd"
label.stem not in empty_names
```

ต้องไม่ใช่ภาพที่ผู้ใช้เลือกเป็น Empty

---

# 19. Copy Labeled Dataset

```python id="c6v3n9"
for label in labeled:

    img = images[label.stem]
```

นำ Label แต่ละตัวมาหา Image ที่มีชื่อเดียวกัน

---

# 20. Copy Image

```python id="x8y2gn"
shutil.copy2(
    img,
    SEED / "images" / img.name
)
```

Copy Image ไป:

```text id="m6t2r9"
P711_Seed/images/
```

---

# 21. Copy Label

```python id="9udq7e"
shutil.copy2(
    label,
    SEED / "labels" / label.name
)
```

Copy Label ไป:

```text id="s0n5be"
P711_Seed/labels/
```

ดังนั้น Labeled Dataset จะมีคู่:

```text id="z5n4n3"
images/
└── P711_001.jpg

labels/
└── P711_001.txt
```

---

# 22. Copy Empty Images

สร้างตัวนับ:

```python id="g7h3a8"
empty_count = 0
```

ใช้สำหรับนับจำนวน Empty Images ที่ Copy สำเร็จ

---

# 23. วน Empty Images

```python id="6f5x3k"
for stem in empty_names:
```

วนชื่อภาพ Empty ทั้งหมด

---

# 24. ตรวจว่า Image มีอยู่จริง

```python id="2a4i9v"
if stem in images:
```

ป้องกันกรณีใน `P711_Empty_Selection.txt` มีชื่อ Image ที่หาไม่เจอ

---

# 25. Copy Empty Image

```python id="m8l7g0"
shutil.copy2(
    images[stem],
    SEED / "images" / images[stem].name
)
```

นำ Empty Image ไปไว้ใน:

```text id="ihx5kw"
P711_Seed/images/
```

### จุดสำคัญ

Empty Image **ไม่มี Label**

ดังนั้นจะมีเฉพาะ:

```text id="n3m4h5"
images/P711_010.jpg
```

แต่ไม่มี:

```text id="7h8k9l"
labels/P711_010.txt
```

---

# 26. นับ Empty Images

```python id="2h9t8c"
empty_count += 1
```

ทุกครั้งที่ Copy สำเร็จ จะเพิ่ม:

```text id="3t4m5n"
+1
```

---

# 27. สรุปจำนวนไฟล์

## จำนวน Images

```python id="xv8d0w"
total_images = len(
    list((SEED / "images").iterdir())
)
```

นับจำนวนไฟล์ใน:

```text id="f9d2s1"
P711_Seed/images/
```

---

## จำนวน Labels

```python id="a1b2c3"
total_labels = len(
    list((SEED / "labels").glob("*.txt"))
)
```

นับเฉพาะ `.txt` ใน:

```text id="d4e5f6"
P711_Seed/labels/
```

---

# 28. แสดง Summary

```python id="z7x8c9"
print("\n=================================")
print("STEP 3 — BUILD SEED DATASET")
print("=================================")
```

แสดงหัวข้อ:

```text id="v1b2n3"
STEP 3 — BUILD SEED DATASET
```

---

## Labeled Images

```python id="m4k5l6"
print("Labeled Images :", len(labeled))
```

จำนวน Image ที่มี Label จริง

---

## Empty Images

```python id="q7w8e9"
print("Empty Images   :", empty_count)
```

จำนวน Empty Image ที่ Copy สำเร็จ

---

## Total Images

```python id="r1t2y3"
print("Total Images   :", total_images)
```

จำนวน Image ทั้งหมดใน Seed

สูตรโดยหลัก:

```text id="u4i5o6"
Total Images
=
Labeled Images
+
Empty Images
```

---

## Total Labels

```python id="p7a8s9"
print("Total Labels   :", total_labels)
```

จำนวน YOLO Label ทั้งหมด

โดย Empty Images จะไม่มี Label

ดังนั้นโดยปกติ:

```text id="d1f2g3"
Total Labels = Labeled Images
```

---

# 29. ตัวอย่างผลลัพธ์

```text id="h4j5k6"
=================================
STEP 3 — BUILD SEED DATASET
=================================
Labeled Images : 80
Empty Images   : 20
Total Images   : 100
Total Labels   : 80
=================================
```

หมายความว่า:

```text id="l7z8x9"
Labeled = 80
Empty   = 20
รวม     = 100 Images
```

---

# 30. โครงสร้างผลลัพธ์

หลังจาก Run จะได้:

```text id="q1w2e3"
P711Test/
│
├── images / labels / ...
│
├── P711_Empty_Selection.txt
│
└── P711_Seed/
    │
    ├── images/
    │   ├── P711_001.jpg
    │   ├── P711_002.jpg
    │   ├── P711_010.jpg    ← Empty
    │   └── ...
    │
    └── labels/
        ├── P711_001.txt
        ├── P711_002.txt
        └── ...
```

---

# 31. ความแตกต่างระหว่าง Labeled และ Empty

## Labeled Image

มีทั้ง:

```text id="a1s2d3"
Image
+
Label
```

ตัวอย่าง:

```text id="f4g5h6"
images/P711_001.jpg
labels/P711_001.txt
```

---

## Empty Image

มีเฉพาะ:

```text id="j7k8l9"
Image
```

ไม่มี Label

ตัวอย่าง:

```text id="z1x2c3"
images/P711_010.jpg
```

---

# 32. ทำไม Empty Image ถึงไม่มี `.txt`

สำหรับ YOLO:

```text id="v4b5n6"
Image ที่ไม่มี Object
```

สามารถใช้เป็น **Negative / Empty Sample**

ดังนั้นไม่มี Bounding Box และไม่จำเป็นต้องมี Annotation Object

---

# 33. ความสัมพันธ์กับ STEP 2.5

STEP 2.5:

```text id="m7q8w9"
ภาพที่ไม่มี Label
       ↓
ผู้ใช้เลือกภาพ Empty
       ↓
P711_Empty_Selection.txt
```

STEP 3.0:

```text id="e1r2t3"
P711_Empty_Selection.txt
       ↓
Copy Empty Images
       ↓
P711_Seed/images/
```

ดังนั้น:

> **STEP 2.5 = เลือก Empty**

> **STEP 3.0 = สร้าง Seed Dataset**

---

# 34. จุดสำคัญของ `shutil.copy2()`

```python id="y4u5i6"
shutil.copy2(source, destination)
```

ใช้ Copy ไฟล์พร้อม Metadata บางส่วนของไฟล์

ต่างจากการย้ายไฟล์ เพราะไฟล์ต้นฉบับยังอยู่ที่เดิม

```text id="o7p8a9"
Original
   │
   ├──────────────► Seed
   │
ยังอยู่ที่เดิม
```

---

# 35. จุดสำคัญของ `shutil.rmtree()`

```python id="s1d2f3"
shutil.rmtree(SEED)
```

ใช้ลบ Folder ทั้ง Folder พร้อมไฟล์ภายใน

ดังนั้น:

```text id="g4h5j6"
P711_Seed เดิม
      ↓
ลบทิ้ง
      ↓
สร้างใหม่
      ↓
Copy Dataset ใหม่
```

---

# 36. สรุป STEP 3.0

```text id="k7l8m9"
STEP 3.0
BUILD SEED DATASET
────────────────────────

P711Test
   │
   ├── Image + Label
   │       ↓
   │   Labeled Dataset
   │
   └── Empty Selection
           ↓
       Empty Images
           │
           ▼
      P711_Seed
       ├── images/
       └── labels/
```

### หน้าที่หลัก

> **สร้าง Dataset ชุดเริ่มต้น (Seed Dataset) ที่ประกอบด้วย Labeled Images + Empty Images พร้อมนำไปใช้ในขั้นตอน Training ต่อไป**
