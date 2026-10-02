# STEP 8 — BUILD FINAL DATASET

## 1. จุดประสงค์ของ Step นี้

Step 8 เป็นขั้นตอนสำหรับสร้าง **Final Dataset** หลังจากผ่านขั้นตอน Seed และ Auto Label มาแล้ว

โดยนำข้อมูลจาก

```text
P711_Seed
```

และรูปที่ได้รับ Auto Label จาก Step 6 มารวมกันไว้ที่

```text
P711_Final
```

โครงสร้างสุดท้ายจะเป็น

```text
P711_Final/
├── images/
│   ├── image001.jpg
│   ├── image002.jpg
│   └── ...
│
└── labels/
    ├── image001.txt
    ├── image002.txt
    └── ...
```

แนวคิดหลักคือ

```text
P711_Seed
     │
     │ Copy
     ▼
P711_Final
     ▲
     │ Copy
     │
Auto Label Images
```

---

# 2. Code ทั้งหมดของ Step 8

```python
from pathlib import Path
import shutil

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

SEED = DATASET / "P711_Seed"
FINAL = DATASET / "P711_Final"

# สร้าง Final ใหม่
if FINAL.exists():
    shutil.rmtree(FINAL)

(FINAL / "images").mkdir(parents=True)
(FINAL / "labels").mkdir(parents=True)

# =========================
# 1. Copy Seed
# =========================
for img in (SEED / "images").iterdir():

    if img.suffix.lower() not in [".jpg", ".jpeg", ".png"]:
        continue

    shutil.copy2(img, FINAL / "images" / img.name)

    label = SEED / "labels" / f"{img.stem}.txt"

    if label.exists():
        shutil.copy2(label, FINAL / "labels" / label.name)


# =========================
# 2. Copy Auto Label
# =========================
for img in DATASET.rglob("*"):

    # ข้าม Seed และ Final
    if SEED in img.parents or FINAL in img.parents:
        continue

    if img.suffix.lower() not in [".jpg", ".jpeg", ".png"]:
        continue

    label = img.with_suffix(".txt")

    if label.exists():
        shutil.copy2(img, FINAL / "images" / img.name)
        shutil.copy2(label, FINAL / "labels" / label.name)


# =========================
# Summary
# =========================
images = list((FINAL / "images").iterdir())
labels = list((FINAL / "labels").glob("*.txt"))

print("\n=================================")
print("STEP 8 — BUILD FINAL DATASET")
print("=================================")
print("Images :", len(images))
print("Labels :", len(labels))
print("Empty  :", len(images) - len(labels))
print("Path   :", FINAL)
print("=================================")
```

---

# 3. Import Library

```python
from pathlib import Path
import shutil
```

ใช้ 2 Library

### `Path`

ใช้จัดการ Path ของ Dataset และไฟล์ต่าง ๆ

### `shutil`

ใช้สำหรับจัดการไฟล์และ Folder เช่น

```python
shutil.copy2()
```

สำหรับ Copy ไฟล์

และ

```python
shutil.rmtree()
```

สำหรับลบ Folder พร้อมข้อมูลภายใน

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Dataset หลัก

```text
C:\Users\pawor\Desktop\P711Test
```

---

# 5. กำหนด Seed และ Final

```python
SEED = DATASET / "P711_Seed"
FINAL = DATASET / "P711_Final"
```

สร้าง Path ของ 2 Folder

### Seed

```text
P711_Seed
```

คือ Dataset ที่ใช้สำหรับ Train ใน Step 5

### Final

```text
P711_Final
```

คือ Dataset ที่จะสร้างขึ้นใน Step นี้

โครงสร้าง

```text
P711Test/
├── P711_Seed/
└── P711_Final/
```

---

# 6. ลบ Final เดิม

```python
if FINAL.exists():
    shutil.rmtree(FINAL)
```

ตรวจสอบว่า `P711_Final` มีอยู่แล้วหรือไม่

ถ้ามี

```python
FINAL.exists()
```

จะเป็น `True`

จากนั้น

```python
shutil.rmtree(FINAL)
```

จะลบ Folder `P711_Final` ทั้งหมด รวมถึงไฟล์ภายใน

เหตุผลคือ Step 8 ต้องการสร้าง Final Dataset ใหม่ทุกครั้ง เพื่อไม่ให้มีไฟล์เก่าค้างอยู่

---

# 7. สร้าง Folder Images และ Labels

```python
(FINAL / "images").mkdir(parents=True)
(FINAL / "labels").mkdir(parents=True)
```

สร้างโครงสร้าง

```text
P711_Final/
├── images/
└── labels/
```

`parents=True` ทำให้ Python สามารถสร้าง Folder ที่อยู่ก่อนหน้าด้วย หากยังไม่มี

---

# 8. ส่วนที่ 1 — Copy Seed

```python
for img in (SEED / "images").iterdir():
```

เข้าไปดูไฟล์ทั้งหมดภายใน

```text
P711_Seed/images
```

โดยใช้

```python
.iterdir()
```

ซึ่งจะวนดูสิ่งที่อยู่โดยตรงใน Folder นั้น

---

# 9. ตรวจสอบว่าเป็น Image

```python
if img.suffix.lower() not in [".jpg", ".jpeg", ".png"]:
    continue
```

เลือกเฉพาะไฟล์รูป

```text
.jpg
.jpeg
.png
```

ถ้าไม่ใช่ Image

```python
continue
```

จะข้ามไฟล์นั้นทันที

---

# 10. Copy Image จาก Seed

```python
shutil.copy2(
    img,
    FINAL / "images" / img.name
)
```

Copy Image จาก

```text
P711_Seed/images
```

ไปยัง

```text
P711_Final/images
```

ตัวอย่าง

```text
P711_Seed/images/image001.jpg
```

จะถูก Copy เป็น

```text
P711_Final/images/image001.jpg
```

---

# 11. หา Label ของ Seed

```python
label = SEED / "labels" / f"{img.stem}.txt"
```

สมมติ Image คือ

```text
image001.jpg
```

ค่า

```python
img.stem
```

คือ

```text
image001
```

จึงสร้าง Path เป็น

```text
P711_Seed/labels/image001.txt
```

---

# 12. ตรวจสอบว่า Label มีอยู่หรือไม่

```python
if label.exists():
    shutil.copy2(label, FINAL / "labels" / label.name)
```

ถ้ามี Label

```text
image001.txt
```

ก็ Copy ไปยัง

```text
P711_Final/labels/
```

ดังนั้น Image และ Label จาก Seed จะถูกย้ายมาเป็นคู่กัน

```text
images/
└── image001.jpg

labels/
└── image001.txt
```

---

# 13. ส่วนที่ 2 — Copy Auto Label

```python
for img in DATASET.rglob("*"):
```

ค้นหาไฟล์ทั้งหมดใน Dataset รวมถึง Subfolder

ซึ่งจะครอบคลุมข้อมูลที่ Auto Label จาก Step 6 สร้างไว้

---

# 14. ข้าม Seed และ Final

```python
if SEED in img.parents or FINAL in img.parents:
    continue
```

ส่วนนี้สำคัญมาก

เพราะ `DATASET.rglob("*")` จะค้นพบทั้ง

```text
P711_Seed
P711_Final
```

ด้วย

ถ้าไม่ข้าม อาจทำให้ข้อมูลจาก Seed หรือ Final ถูกนำมาประมวลผลซ้ำ

ดังนั้น

```python
SEED in img.parents
```

หมายถึง

> ไฟล์นี้อยู่ภายใน `P711_Seed` หรือไม่

และ

```python
FINAL in img.parents
```

หมายถึง

> ไฟล์นี้อยู่ภายใน `P711_Final` หรือไม่

ถ้าใช่อย่างใดอย่างหนึ่ง

```python
continue
```

เพื่อข้าม

---

# 15. ตรวจสอบ Image

```python
if img.suffix.lower() not in [".jpg", ".jpeg", ".png"]:
    continue
```

เลือกเฉพาะ Image

```text
.jpg
.jpeg
.png
```

---

# 16. หา Label ที่อยู่คู่กับ Image

```python
label = img.with_suffix(".txt")
```

เปลี่ยนนามสกุลของ Image เป็น `.txt`

ตัวอย่าง

```text
image010.jpg
```

จะกลายเป็น

```text
image010.txt
```

โดยใช้

```python
img.with_suffix(".txt")
```

---

# 17. ตรวจสอบว่า Auto Label มีอยู่

```python
if label.exists():
```

ตรวจสอบว่าไฟล์ `.txt` ของ Image นั้นมีอยู่จริงหรือไม่

ถ้ามี แสดงว่า Image นี้มี Label

ตัวอย่าง

```text
image010.jpg
image010.txt
```

จะผ่านเงื่อนไข

---

# 18. Copy Auto Label Image

```python
shutil.copy2(
    img,
    FINAL / "images" / img.name
)
```

Copy Image ที่มี Label ไปยัง

```text
P711_Final/images
```

---

# 19. Copy Auto Label

```python
shutil.copy2(
    label,
    FINAL / "labels" / label.name
)
```

Copy Label คู่กันไปยัง

```text
P711_Final/labels
```

ดังนั้นข้อมูลจะถูกเก็บเป็นคู่

```text
Image
   │
   └── image010.jpg

Label
   │
   └── image010.txt
```

---

# 20. ทำไมต้องเช็ค `label.exists()`?

เพราะ Step 8 ต้องการเอาเฉพาะ Image ที่มี Label

ถ้ามี

```text
image010.jpg
```

แต่ไม่มี

```text
image010.txt
```

จะไม่ถูก Copy

ทำให้ Final Dataset ลดโอกาสมี Image ที่ไม่มี Label

---

# 21. สร้างรายการ Images

```python
images = list((FINAL / "images").iterdir())
```

อ่านไฟล์ทั้งหมดใน

```text
P711_Final/images
```

แล้วเปลี่ยนเป็น List

ตัวอย่าง

```python
images = [
    image001.jpg,
    image002.jpg,
    image003.jpg
]
```

---

# 22. สร้างรายการ Labels

```python
labels = list((FINAL / "labels").glob("*.txt"))
```

อ่านเฉพาะไฟล์ `.txt` ใน

```text
P711_Final/labels
```

เช่น

```text
image001.txt
image002.txt
image003.txt
```

---

# 23. นับจำนวน Images

```python
len(images)
```

ใช้ดูว่ามี Image ใน Final Dataset ทั้งหมดกี่รูป

เช่น

```text
Images : 1000
```

---

# 24. นับจำนวน Labels

```python
len(labels)
```

ใช้ดูว่ามี Label `.txt` ทั้งหมดกี่ไฟล์

เช่น

```text
Labels : 1000
```

---

# 25. คำนวณ Empty

```python
len(images) - len(labels)
```

คำนวณจำนวน Image ที่ไม่มี Label

เช่น

```text
Images = 1000
Labels = 980
```

จะได้

```text
Empty = 20
```

หมายถึงมี Image 20 รูปที่ไม่มี Label คู่กัน

ถ้า

```text
Images = 1000
Labels = 1000
```

จะได้

```text
Empty = 0
```

---

# 26. แสดง Path ของ Final Dataset

```python
print("Path   :", FINAL)
```

แสดงตำแหน่งของ Final Dataset

ตัวอย่าง

```text
Path : C:\Users\pawor\Desktop\P711Test\P711_Final
```

---

# 27. ตัวอย่าง Output

หลังจากทำงานเสร็จอาจแสดง

```text
=================================
STEP 8 — BUILD FINAL DATASET
=================================
Images : 1250
Labels : 1250
Empty  : 0
Path   : C:\Users\pawor\Desktop\P711Test\P711_Final
=================================
```

สามารถอ่านได้ว่า

```text
Images = 1250 รูป
Labels = 1250 ไฟล์
Empty  = 0 รูป
```

แสดงว่าใน Final Dataset มี Image และ Label จำนวนเท่ากัน

---

# 28. โครงสร้างก่อน Step 8

ก่อนสร้าง Final Dataset อาจมีลักษณะ

```text
P711Test/
│
├── P711_Seed/
│   ├── images/
│   └── labels/
│
├── image001.jpg
├── image001.txt
├── image002.jpg
├── image002.txt
│
└── ...
```

---

# 29. โครงสร้างหลัง Step 8

หลังทำงานเสร็จจะได้

```text
P711Test/
│
├── P711_Seed/
│   ├── images/
│   └── labels/
│
├── P711_Final/
│   ├── images/
│   │   ├── image001.jpg
│   │   ├── image002.jpg
│   │   └── ...
│   │
│   └── labels/
│       ├── image001.txt
│       ├── image002.txt
│       └── ...
│
└── ...
```

---

# 30. Flow ของ Step 8

```text
                P711Test
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
     P711_Seed          Auto Label
          │                 │
          │                 │
          ▼                 ▼
       Images             Images
       Labels             Labels
          │                 │
          └────────┬────────┘
                   │
                   ▼
             P711_Final
                   │
          ┌────────┴────────┐
          ▼                 ▼
       images/           labels/
          │                 │
          └────────┬────────┘
                   ▼
             Final Dataset
```

---

# 31. หลักการสำคัญของ Step 8

Step นี้แบ่งการ Copy เป็น 2 ส่วน

### ส่วนที่ 1 — Seed

```text
P711_Seed
     ↓
Copy Image + Label
     ↓
P711_Final
```

### ส่วนที่ 2 — Auto Label

```text
Dataset Images
     ↓
ตรวจว่ามี .txt
     ↓
มี Label
     ↓
Copy Image + Label
     ↓
P711_Final
```

สุดท้ายทั้งสองส่วนจะถูกรวมกันใน

```text
P711_Final
```

---

# 32. สรุป Step 8

Step 8 ทำหน้าที่

1. กำหนด `P711_Seed`
2. กำหนด `P711_Final`
3. ลบ `P711_Final` เดิมถ้ามี
4. สร้าง `images/`
5. สร้าง `labels/`
6. Copy Image จาก Seed
7. Copy Label จาก Seed
8. ค้นหา Image ที่มี Auto Label
9. ข้าม `P711_Seed`
10. ข้าม `P711_Final`
11. Copy Image ที่มี Label
12. Copy Label ที่คู่กับ Image
13. นับจำนวน Image
14. นับจำนวน Label
15. คำนวณจำนวน Image ที่ไม่มี Label
16. แสดงตำแหน่ง Final Dataset

**เป้าหมายของ Step 8 คือ**

```text
Seed Dataset
      +
Auto Labeled Dataset
      ↓
P711_Final
      ↓
images + labels
      ↓
Dataset พร้อมสำหรับขั้นตอนถัดไป
```
