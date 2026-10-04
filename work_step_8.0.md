# STEP 8.0 — BUILD FINAL DATASET

## 1. หน้าที่ของ STEP 8.0

STEP 8.0 ทำหน้าที่สร้าง Dataset ใหม่ชื่อ

```text
P711_Final
```

โดยนำข้อมูล 2 ส่วนมารวมกัน

```text
P711_Seed
     +
Auto Label
     ↓
P711_Final
```

โครงสร้างที่ได้คือ

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

จุดประสงค์คือสร้าง **Final Dataset** ที่รวม Label เดิมและ Label ที่ Auto Label สร้างขึ้น เพื่อเตรียมใช้ Train Model รอบต่อไป

---

# 2. Import Library

```python
from pathlib import Path
import shutil
```

### `Path`

ใช้จัดการ Path ของไฟล์และ Folder

เช่น

```python
DATASET / "P711_Seed"
```

### `shutil`

ใช้สำหรับ Copy และลบ Folder

ในโค้ดนี้ใช้

```python
shutil.copy2()
```

และ

```python
shutil.rmtree()
```

---

# 3. กำหนด Dataset

```python
# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

ตำแหน่ง Dataset หลัก

```text
C:\Users\pawor\Desktop\P711Test
```

---

# 4. กำหนด Seed และ Final

```python
SEED = DATASET / "P711_Seed"
FINAL = DATASET / "P711_Final"
```

จะได้

```text
P711Test/
├── P711_Seed/
└── P711_Final/
```

โดย

### `SEED`

คือ Dataset ที่สร้างจาก STEP 3.0

### `FINAL`

คือ Dataset ใหม่ที่ STEP 8.0 กำลังจะสร้าง

---

# 5. ลบ Final เดิม

```python
if FINAL.exists():
    shutil.rmtree(FINAL)
```

ถ้ามี `P711_Final` อยู่แล้ว จะลบทิ้งทั้งหมด

ตัวอย่าง

```text
P711_Final/
├── images/
├── labels/
└── data.yaml
```

จะถูกลบทั้ง Folder

แล้วจึงสร้างใหม่

> **ข้อควรระวัง:** `shutil.rmtree()` เป็นการลบแบบ Recursive ดังนั้นข้อมูลทั้งหมดใน `P711_Final` จะหายและไม่สามารถกู้คืนได้ง่าย ๆ

ข้อดีคือทุกครั้งที่รัน STEP 8.0 จะได้ Final Dataset ที่สร้างใหม่จากข้อมูลต้นทาง ไม่เกิดไฟล์เก่าค้างอยู่

---

# 6. สร้างโครงสร้าง Final

```python
(FINAL / "images").mkdir(parents=True)
(FINAL / "labels").mkdir(parents=True)
```

สร้าง

```text
P711_Final/
├── images/
└── labels/
```

`parents=True` หมายถึงถ้า Parent Folder ยังไม่มี Python สามารถสร้างให้ด้วย

---

# 7. ส่วนที่ 1 — Copy Seed

```python
for img in (SEED / "images").iterdir():
```

เข้าไปดูไฟล์ใน

```text
P711_Seed/images
```

โดยใช้

```python
.iterdir()
```

ซึ่งจะดูไฟล์และ Folder **โดยตรงในระดับนั้น**

ต่างจาก

```python
.rglob("*")
```

ที่ค้นหาลงไปทุกระดับ

---

# 8. ตรวจสอบว่าเป็น Image

```python
if img.suffix.lower() not in [".jpg", ".jpeg", ".png"]:
    continue
```

ตรวจว่าไฟล์เป็น Image หรือไม่

รองรับ

```text
.jpg
.jpeg
.png
```

ถ้าไม่ใช่จะใช้

```python
continue
```

เพื่อข้ามไฟล์นั้น

---

# 9. Copy Seed Image

```python
shutil.copy2(img, FINAL / "images" / img.name)
```

Copy รูปจาก

```text
P711_Seed/images
```

ไปยัง

```text
P711_Final/images
```

ตัวอย่าง

```text
P711_Seed/images/car001.jpg
              ↓
P711_Final/images/car001.jpg
```

### `copy2()`

มีพฤติกรรมคล้าย `copy()` แต่พยายามรักษา metadata ของไฟล์ด้วย

---

# 10. หา Label คู่กับ Image

```python
label = SEED / "labels" / f"{img.stem}.txt"
```

ถ้ารูปคือ

```text
car001.jpg
```

ค่า

```python
img.stem
```

คือ

```text
car001
```

ดังนั้น Python จะหา

```text
P711_Seed/labels/car001.txt
```

---

# 11. Copy Label

```python
if label.exists():
    shutil.copy2(label, FINAL / "labels" / label.name)
```

ถ้ามี Label คู่กับ Image

```text
car001.jpg
car001.txt
```

ก็ Copy Label ไปด้วย

```text
P711_Seed
    ↓
P711_Final

images/car001.jpg
labels/car001.txt
```

---

# 12. ทำไมต้อง `if label.exists()`?

เพราะใน Seed Dataset อาจมี **Empty Image**

ตัวอย่าง

```text
images/
├── object001.jpg
├── object002.jpg
└── empty001.jpg

labels/
├── object001.txt
└── object002.txt
```

`empty001.jpg` ไม่มี `.txt`

โค้ดจึงสามารถ Copy รูปได้ แต่ไม่จำเป็นต้อง Copy Label

นี่เป็นลักษณะของ **Negative / Empty Image**

---

# 13. ส่วนที่ 2 — Copy Auto Label

```python
for img in DATASET.rglob("*"):
```

ค้นหาไฟล์ทั้งหมดภายใน Dataset แบบ Recursive

ตัวอย่าง

```text
P711Test/
├── image001.jpg
├── image002.jpg
├── folder1/
│   ├── image003.jpg
│   └── image003.txt
├── P711_Seed/
└── P711_Final/
```

`rglob("*")` สามารถค้นหาได้หลายระดับ

---

# 14. ข้าม Seed และ Final

```python
if SEED in img.parents or FINAL in img.parents:
    continue
```

ส่วนนี้สำคัญมาก

เพราะ `DATASET.rglob("*")` สามารถค้นหาเข้าไปใน

```text
P711_Seed
```

และ

```text
P711_Final
```

ได้ด้วย

แต่เราไม่ต้องการให้ STEP 8.0 นำข้อมูลจากสอง Folder นี้กลับมา Copy ซ้ำ

จึงใช้

```python
continue
```

เพื่อข้าม

```text
P711_Seed/*
P711_Final/*
```

---

# 15. ตรวจว่าเป็น Image

```python
if img.suffix.lower() not in [".jpg", ".jpeg", ".png"]:
    continue
```

เลือกเฉพาะ Image

ดังนั้นไฟล์ประเภทอื่น เช่น

```text
.yaml
.txt
.py
```

จะถูกข้าม

---

# 16. หา Auto Label

```python
label = img.with_suffix(".txt")
```

ถ้า Image คือ

```text
image100.jpg
```

จะเปลี่ยนเป็น

```text
image100.txt
```

ตัวอย่าง

```text
image100.jpg
     ↓
with_suffix(".txt")
     ↓
image100.txt
```

นี่เป็นวิธีจับคู่ Image กับ Label โดยใช้ชื่อเดียวกัน

---

# 17. ตรวจว่ามี Auto Label หรือไม่

```python
if label.exists():
```

ถ้ามี

```text
image100.jpg
image100.txt
```

ถือว่า Image นี้มี Label แล้ว

จึง Copy ทั้งสองไฟล์

---

# 18. Copy Auto Label Image

```python
shutil.copy2(img, FINAL / "images" / img.name)
```

นำ Image ไปยัง

```text
P711_Final/images
```

---

# 19. Copy Auto Label

```python
shutil.copy2(label, FINAL / "labels" / label.name)
```

นำ Label ไปยัง

```text
P711_Final/labels
```

ดังนั้น

```text
image100.jpg
image100.txt
```

จะกลายเป็น

```text
P711_Final/
├── images/
│   └── image100.jpg
│
└── labels/
    └── image100.txt
```

---

# 20. ภาพรวมการรวมข้อมูล

STEP 8.0 มีการ Copy สองรอบ

### รอบที่ 1

```text
P711_Seed
    ↓
P711_Final
```

นำ Label เดิม + Empty Image

### รอบที่ 2

```text
Dataset หลัก
    ↓
ค้นหา Image ที่มี .txt
    ↓
Auto Label
    ↓
P711_Final
```

สุดท้าย

```text
Seed Images
     +
Seed Labels
     +
Auto Label Images
     +
Auto Labels
     ↓
P711_Final
```

---

# 21. Summary

```python
images = list((FINAL / "images").iterdir())
labels = list((FINAL / "labels").glob("*.txt"))
```

ตรวจสอบจำนวนไฟล์ใน Final

### จำนวน Images

```python
len(images)
```

### จำนวน Labels

```python
len(labels)
```

---

# 22. คำนวณ Empty

```python
len(images) - len(labels)
```

ในโค้ดนี้ใช้จำนวน Image ลบจำนวน Label เพื่อประมาณจำนวน Image ที่ไม่มี Label

ตัวอย่าง

```text
Images = 1,000
Labels =   850

Empty = 1,000 - 850
      = 150
```

จึงแสดง

```text
Empty : 150
```

---

# 23. แสดงผล

```python
print("\n=================================")
print("STEP 8 — BUILD FINAL DATASET")
print("=================================")
print("Images :", len(images))
print("Labels :", len(labels))
print("Empty  :", len(images) - len(labels))
print("Path   :", FINAL)
print("=================================")
```

ตัวอย่าง

```text
=================================
STEP 8 — BUILD FINAL DATASET
=================================
Images : 1500
Labels : 1350
Empty  : 150
Path   : C:\Users\pawor\Desktop\P711Test\P711_Final
=================================
```

---

# 24. ความสัมพันธ์กับ STEP 7.0

ก่อน STEP 8.0 เรามี

```text
STEP 6.0
Auto Label
      ↓
สร้าง .txt
      ↓
STEP 7.0
Visual Check
      ↓
ตรวจ Bounding Box
      ↓
STEP 8.0
Build Final Dataset
```

ดังนั้น STEP 7.0 เป็นการ **ตรวจสอบ**

ส่วน STEP 8.0 เป็นการ **รวบรวม**

---

# 25. จุดสำคัญของโค้ดนี้

มีแนวคิดสำคัญ 2 อย่าง

### 1. Seed เป็นข้อมูลตั้งต้น

```text
P711_Seed
```

มีข้อมูลที่เราคัดเลือกไว้ตั้งแต่ต้น

### 2. Auto Label เป็นข้อมูลที่ Model สร้างเพิ่ม

```text
Auto Label
```

จึงสามารถนำทั้งสองส่วนมารวมกันเป็น

```text
P711_Final
```

เพื่อเตรียม Train Model รอบถัดไป

---

# 26. ข้อควรระวังเรื่อง `Empty`

บรรทัดนี้

```python
print("Empty  :", len(images) - len(labels))
```

เป็นเพียง **การประมาณจากจำนวนไฟล์**

ไม่ได้ตรวจว่า Label `.txt` เป็นไฟล์ว่างจริงหรือไม่

ตัวอย่าง

```text
images/
├── A.jpg
├── B.jpg
└── C.jpg

labels/
├── A.txt
├── B.txt
└── C.txt
```

ถ้า `C.txt` เป็นไฟล์ว่าง

```text
Images = 3
Labels = 3
```

โค้ดจะรายงาน

```text
Empty = 0
```

ทั้งที่จริง `C.jpg` อาจเป็น Empty Image

ดังนั้น

> `Images - Labels` หมายถึง **Image ที่ไม่มีไฟล์ Label** ไม่ได้หมายถึง **Image ที่ไม่มี Object**

นี่เป็นจุดสำคัญมาก

---

# 27. จุดที่ควรระวังอีกอย่าง — ชื่อไฟล์ซ้ำ

โค้ดใช้

```python
FINAL / "images" / img.name
```

ดังนั้นถ้ามีรูปชื่อเหมือนกันอยู่คนละ Folder เช่น

```text
folderA/image001.jpg
folderB/image001.jpg
```

ทั้งสองไฟล์จะถูก Copy ไปที่

```text
P711_Final/images/image001.jpg
```

จึงมีโอกาส **ชื่อชนกันและไฟล์หนึ่งเขียนทับอีกไฟล์**

สำหรับ Dataset ที่มีชื่อ Image ไม่ซ้ำกัน ปัญหานี้จะไม่เกิด

---

# 28. Final Dataset หลัง STEP 8.0

โครงสร้างโดยรวมจะเป็น

```text
P711Test/
│
├── P711_Seed/
│   ├── images/
│   ├── labels/
│   └── data.yaml
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
├── P711_Empty_Selection.txt
└── ...
```

---

# 29. Pipeline ทั้งหมดถึง STEP 8

```text
┌─────────────────────────────┐
│ STEP 1                      │
│ Check Dataset               │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ STEP 2.0                    │
│ Visualize Original Labels   │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ STEP 2.5                    │
│ Select Empty Images         │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ STEP 3.0                    │
│ Build Seed Dataset          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ STEP 4.0                    │
│ Build data.yaml             │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ STEP 5.0                    │
│ Train Seed Model            │
└──────────────┬──────────────┘
               ↓
          best.pt
               ↓
┌─────────────────────────────┐
│ STEP 6.0                    │
│ Auto Label                  │
└──────────────┬──────────────┘
               ↓
          Auto Labels
               ↓
┌─────────────────────────────┐
│ STEP 7.0                    │
│ Visual Check                │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ STEP 8.0                    │
│ Build Final Dataset         │
└──────────────┬──────────────┘
               ↓
        P711_Final
               ↓
       Train รอบถัดไป
```

# 30. สรุป STEP 8.0

| ส่วน                   | หน้าที่                        |
| ---------------------- | ------------------------------ |
| `SEED`                 | Dataset ตั้งต้น                |
| `FINAL`                | Dataset ปลายทาง                |
| `rmtree()`             | ลบ Final เดิม                  |
| `mkdir()`              | สร้าง `images/labels`          |
| `iterdir()`            | อ่านไฟล์ใน Seed                |
| `rglob("*")`           | ค้นหา Image ใน Dataset         |
| `SEED in img.parents`  | ข้าม Seed                      |
| `FINAL in img.parents` | ข้าม Final                     |
| `with_suffix(".txt")`  | หา Label คู่กับ Image          |
| `copy2()`              | Copy Image/Label               |
| `len(images)`          | จำนวน Image                    |
| `len(labels)`          | จำนวน Label                    |
| `Images - Labels`      | จำนวน Image ที่ไม่มีไฟล์ Label |

**สรุปสั้น ๆ:** STEP 8.0 คือขั้นตอน **รวม Dataset** โดยนำ `P711_Seed` และผลจาก Auto Label มาสร้างเป็น `P711_Final` เพื่อใช้เป็น Dataset สำหรับการ Train Model รอบถัดไป

จุดที่ต้องจำคือ **STEP 7.0 = ตรวจสอบ Label** และ **STEP 8.0 = รวมข้อมูลหลังตรวจสอบ**.
