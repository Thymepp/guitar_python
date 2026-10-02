# STEP 9 — VERIFY FINAL DATASET

## 1. จุดประสงค์ของ Step นี้

Step 9 ใช้สำหรับตรวจสอบ **Final Dataset** ที่สร้างจาก Step 8

โดยจะอ่าน

```text
P711_Final/images/
P711_Final/labels/
```

แล้วนำ Label ของแต่ละภาพมาวาด Bounding Box ลงบนภาพ

จากนั้นแสดงผลครั้งละ 6 รูป เพื่อให้สามารถตรวจสอบด้วยสายตาได้ว่า

* Image มีอยู่จริง
* Label มีอยู่จริง
* Image กับ Label จับคู่กันถูกต้อง
* Bounding Box อยู่ในตำแหน่งที่ถูกต้อง
* Label Format ถูกต้อง
* ไม่มีกรอบที่ผิดตำแหน่งอย่างเห็นได้ชัด

แนวคิดหลักคือ

```text
P711_Final
    │
    ├── images/
    │      │
    │      ▼
    │   อ่าน Image
    │
    └── labels/
           │
           ▼
        อ่าน Label
           │
           ▼
   แปลง YOLO Coordinate
           │
           ▼
    วาด Bounding Box
           │
           ▼
     แสดง 6 รูป/หน้า
```

---

# 2. Code ทั้งหมดของ Step 9

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

FINAL = DATASET / "P711_Final"

images = sorted([
    p for p in (FINAL / "images").iterdir()
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
])

for n in range(0, len(images), 6):

    fig, ax = plt.subplots(2, 3, figsize=(15, 9))
    ax = ax.ravel()

    current = images[n:n+6]

    for i, img_path in enumerate(current):

        img = cv2.imread(str(img_path))

        if img is None:
            continue

        h, w = img.shape[:2]

        label = FINAL / "labels" / f"{img_path.stem}.txt"

        if label.exists():

            for line in label.read_text(
                encoding="utf-8-sig"
            ).splitlines():

                d = line.split()

                if len(d) != 5:
                    continue

                _, x, y, bw, bh = map(float, d)

                x1 = int((x - bw/2) * w)
                y1 = int((y - bh/2) * h)
                x2 = int((x + bw/2) * w)
                y2 = int((y + bh/2) * h)

                cv2.rectangle(
                    img,
                    (x1, y1),
                    (x2, y2),
                    (0, 255, 0),
                    2
                )

        ax[i].imshow(
            cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        )
        ax[i].axis("off")

    for a in ax[len(current):]:
        a.axis("off")

    plt.tight_layout()
    plt.show()
```

---

# 3. Import Library

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt
```

ใช้ 3 Library

### `Path`

ใช้จัดการ Path ของ Dataset และไฟล์

### `cv2`

ใช้ OpenCV สำหรับ

* อ่าน Image
* อ่านขนาด Image
* วาด Bounding Box
* แปลงสี BGR → RGB

### `matplotlib.pyplot`

ใช้แสดง Image เป็น Grid

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Folder หลักของ Dataset

ตัวอย่าง

```text
C:\Users\pawor\Desktop\P711Test
```

---

# 5. กำหนด Final Dataset

```python
FINAL = DATASET / "P711_Final"
```

สร้าง Path ไปยัง Final Dataset

จึงได้

```text
C:\Users\pawor\Desktop\P711Test\P711_Final
```

ภายในมี

```text
P711_Final/
├── images/
└── labels/
```

---

# 6. ค้นหา Image ใน Final Dataset

```python
images = sorted([
    p for p in (FINAL / "images").iterdir()
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
])
```

ส่วนนี้ค้นหา Image ภายใน

```text
P711_Final/images
```

โดยเลือกเฉพาะ

```text
.jpg
.jpeg
.png
```

---

# 7. `.iterdir()`

```python
(FINAL / "images").iterdir()
```

ใช้ดูไฟล์และ Folder ที่อยู่ **โดยตรง** ภายใน `images`

ต่างจาก `rglob()` ที่สามารถค้นหาเข้าไปใน Subfolder ได้

ใน Step นี้ต้องการดูเฉพาะไฟล์ใน

```text
P711_Final/images/
```

จึงใช้ `.iterdir()`

---

# 8. `.suffix.lower()`

```python
p.suffix.lower()
```

ใช้ดูนามสกุลไฟล์

ตัวอย่าง

```text
image001.JPG
```

จะได้

```text
.jpg
```

หลังจากใช้ `.lower()`

ทำให้สามารถตรวจสอบนามสกุลได้โดยไม่สนใจตัวพิมพ์ใหญ่/เล็ก

---

# 9. `sorted()`

```python
sorted([...])
```

เรียงรายการ Image ก่อนนำไปแสดง

เช่น

```text
image001.jpg
image002.jpg
image003.jpg
...
```

ทำให้ลำดับการตรวจสอบมีความเป็นระเบียบ

---

# 10. แบ่ง Image ครั้งละ 6 รูป

```python
for n in range(0, len(images), 6):
```

วน Image ทีละ 6 รูป

ตัวอย่างถ้ามี 20 รูป

```text
รอบที่ 1 → รูป 1-6
รอบที่ 2 → รูป 7-12
รอบที่ 3 → รูป 13-18
รอบที่ 4 → รูป 19-20
```

---

# 11. สร้างพื้นที่แสดงผล 2 × 3

```python
fig, ax = plt.subplots(2, 3, figsize=(15, 9))
```

สร้าง Grid

```text
2 Rows × 3 Columns
```

รวมทั้งหมด 6 ช่อง

```text
┌──────────┬──────────┬──────────┐
│ Image 1  │ Image 2  │ Image 3  │
├──────────┼──────────┼──────────┤
│ Image 4  │ Image 5  │ Image 6  │
└──────────┴──────────┴──────────┘
```

---

# 12. `ax.ravel()`

```python
ax = ax.ravel()
```

เปลี่ยน `ax` จาก Array แบบ 2 มิติ

```text
2 × 3
```

เป็น Array แบบ 1 มิติ

```text
[0, 1, 2, 3, 4, 5]
```

ทำให้สามารถใช้

```python
ax[i]
```

ในการเลือกตำแหน่งของแต่ละภาพได้ง่าย

---

# 13. เลือก Image ชุดปัจจุบัน

```python
current = images[n:n+6]
```

เลือก Image ตั้งแต่ตำแหน่ง

```text
n
```

ถึงก่อน

```text
n + 6
```

ตัวอย่าง

```python
n = 0
```

จะได้

```text
images[0:6]
```

คือ 6 รูปแรก

เมื่อ

```python
n = 6
```

จะได้

```text
images[6:12]
```

คือรูปที่ 7-12

---

# 14. วนทีละ Image

```python
for i, img_path in enumerate(current):
```

`enumerate()` ทำให้ได้ทั้ง

```text
i
```

ตำแหน่งของภาพ

และ

```text
img_path
```

Path ของ Image

ตัวอย่าง

```text
i = 0 → image001.jpg
i = 1 → image002.jpg
i = 2 → image003.jpg
```

---

# 15. อ่าน Image

```python
img = cv2.imread(str(img_path))
```

ใช้ OpenCV อ่าน Image

ตัวอย่าง

```text
P711_Final/images/image001.jpg
```

จะถูกโหลดมาเก็บไว้ใน

```python
img
```

---

# 16. ตรวจสอบว่าอ่าน Image สำเร็จหรือไม่

```python
if img is None:
    continue
```

ถ้า OpenCV อ่าน Image ไม่สำเร็จ

```python
img is None
```

จะเป็น `True`

โปรแกรมจะข้าม Image นั้นด้วย

```python
continue
```

ทำให้ไม่เกิด Error จากการใช้ Image ที่อ่านไม่ได้

---

# 17. อ่านขนาด Image

```python
h, w = img.shape[:2]
```

ดึงขนาดของ Image

```text
h = Height
w = Width
```

เช่น

```text
Image = 1920 × 1080
```

จะได้

```text
w = 1920
h = 1080
```

ข้อมูลนี้ใช้ในการแปลง Coordinate จาก YOLO Format เป็น Pixel

---

# 18. หา Label คู่กับ Image

```python
label = FINAL / "labels" / f"{img_path.stem}.txt"
```

สร้าง Path ของ Label จากชื่อ Image

ตัวอย่าง

```text
image001.jpg
```

มี

```python
img_path.stem
```

เป็น

```text
image001
```

จึงสร้างเป็น

```text
P711_Final/labels/image001.txt
```

ทำให้ Image และ Label ถูกจับคู่ตามชื่อ

```text
image001.jpg
      ↕
image001.txt
```

---

# 19. ตรวจสอบว่า Label มีอยู่หรือไม่

```python
if label.exists():
```

ตรวจสอบว่าไฟล์ Label มีอยู่จริง

ถ้ามี

```text
image001.txt
```

โปรแกรมจะอ่าน Label และวาด Bounding Box

ถ้าไม่มี Label โปรแกรมจะยังแสดง Image แต่จะไม่มี Bounding Box

---

# 20. อ่าน Label ทีละบรรทัด

```python
for line in label.read_text(
    encoding="utf-8-sig"
).splitlines():
```

อ่านไฟล์ `.txt`

โดยใช้

```text
encoding="utf-8-sig"
```

จากนั้น

```python
.splitlines()
```

แบ่งข้อมูลออกเป็นทีละบรรทัด

ตัวอย่าง

```text
0 0.500000 0.500000 0.300000 0.400000
1 0.700000 0.600000 0.150000 0.200000
```

จะถูกอ่านทีละบรรทัด

---

# 21. แยกข้อมูลด้วย `split()`

```python
d = line.split()
```

ตัวอย่าง

```text
0 0.500000 0.500000 0.300000 0.400000
```

จะกลายเป็น

```python
[
    "0",
    "0.500000",
    "0.500000",
    "0.300000",
    "0.400000"
]
```

---

# 22. ตรวจสอบ YOLO Format

```python
if len(d) != 5:
    continue
```

แต่ละ Label ของ YOLO Detection ต้องมี 5 ค่า

```text
Class
X
Y
Width
Height
```

ดังนั้นถ้าไม่ครบ 5 ค่า

```python
continue
```

เพื่อข้ามข้อมูลบรรทัดนั้น

---

# 23. อ่าน Class และ Bounding Box

```python
_, x, y, bw, bh = map(float, d)
```

แปลงข้อมูลทั้งหมดเป็น `float`

ข้อมูลคือ

```text
_  = Class
x  = Center X
y  = Center Y
bw = Box Width
bh = Box Height
```

ตัวแปร `_` รับ Class ID แต่ Step นี้ไม่ได้ใช้ Class ในการแสดงผล

---

# 24. แปลง YOLO Coordinate เป็น Pixel

YOLO เก็บ Bounding Box แบบ Normalized

```text
x = Center X
y = Center Y
bw = Width
bh = Height
```

โดยค่าอยู่ในช่วงประมาณ

```text
0.0 - 1.0
```

แต่ OpenCV ต้องการ Pixel Coordinate

จึงต้องแปลง

---

# 25. คำนวณมุมซ้ายบน

```python
x1 = int((x - bw/2) * w)
y1 = int((y - bh/2) * h)
```

สูตร

```text
x1 = (Center X - Width / 2) × Image Width

y1 = (Center Y - Height / 2) × Image Height
```

ผลลัพธ์คือ

```text
(x1, y1)
```

ซึ่งเป็นมุมซ้ายบนของ Bounding Box

---

# 26. คำนวณมุมขวาล่าง

```python
x2 = int((x + bw/2) * w)
y2 = int((y + bh/2) * h)
```

สูตร

```text
x2 = (Center X + Width / 2) × Image Width

y2 = (Center Y + Height / 2) × Image Height
```

ผลลัพธ์คือ

```text
(x2, y2)
```

ซึ่งเป็นมุมขวาล่างของ Bounding Box

---

# 27. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (x1, y1),
    (x2, y2),
    (0, 255, 0),
    2
)
```

ใช้ OpenCV วาดกรอบบน Image

ค่าต่าง ๆ คือ

```text
img
```

Image ที่ต้องการวาด

```text
(x1, y1)
```

มุมซ้ายบน

```text
(x2, y2)
```

มุมขวาล่าง

```text
(0, 255, 0)
```

สีเขียวในระบบ BGR

```text
2
```

ความหนาของเส้น Bounding Box

---

# 28. แสดง Image

```python
ax[i].imshow(
    cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
)
```

OpenCV อ่าน Image เป็น

```text
BGR
```

แต่ Matplotlib ใช้

```text
RGB
```

จึงต้องแปลงด้วย

```python
cv2.cvtColor(
    img,
    cv2.COLOR_BGR2RGB
)
```

ก่อนแสดงผล

---

# 29. ซ่อน Axis

```python
ax[i].axis("off")
```

ซ่อนแกน X/Y

ทำให้หน้าจอแสดงเฉพาะ Image และ Bounding Box

---

# 30. ซ่อนช่องที่ไม่มี Image

```python
for a in ax[len(current):]:
    a.axis("off")
```

กรณีหน้าสุดท้ายมี Image ไม่ครบ 6 รูป

ตัวอย่างมีเหลือเพียง 2 รูป

```text
┌──────────┬──────────┬──────────┐
│ Image 1  │ Image 2  │          │
├──────────┼──────────┼──────────┤
│          │          │          │
└──────────┴──────────┴──────────┘
```

ช่องที่ไม่มี Image จะถูกปิดไว้

---

# 31. จัด Layout

```python
plt.tight_layout()
```

ปรับระยะห่างของแต่ละช่องให้เหมาะสม

---

# 32. แสดงผล

```python
plt.show()
```

แสดงภาพชุดปัจจุบัน

จากนั้นเมื่อปิดหรือจบการแสดงผล โปรแกรมจะทำ Batch ถัดไป

```text
6 รูปแรก
   ↓
6 รูปถัดไป
   ↓
6 รูปถัดไป
   ↓
...
```

จนกว่าจะครบทั้งหมด

---

# 33. ตัวอย่างการทำงาน

สมมติ Final Dataset มี

```text
P711_Final/
├── images/
│   ├── A.jpg
│   ├── B.jpg
│   └── C.jpg
│
└── labels/
    ├── A.txt
    ├── B.txt
    └── C.txt
```

โปรแกรมจะจับคู่

```text
A.jpg ←→ A.txt
B.jpg ←→ B.txt
C.jpg ←→ C.txt
```

จากนั้นอ่านข้อมูลใน Label เช่น

```text
0 0.500000 0.500000 0.300000 0.400000
```

แล้ววาดกรอบบน Image

---

# 34. สิ่งที่ Step 9 ใช้ตรวจสอบ

Step 9 สามารถใช้ตรวจสอบปัญหาที่มองเห็นได้ เช่น

### Bounding Box ถูกตำแหน่ง

```text
        ┌───────────────┐
        │    Object     │
        │               │
        └───────────────┘
```

### Bounding Box เล็กหรือใหญ่เกินไป

```text
┌───────────────────────┐
│       Object          │
│   ┌─────┐             │
│   │ Box │             │
│   └─────┘             │
└───────────────────────┘
```

### Bounding Box อยู่ผิดตำแหน่ง

```text
Object

          ┌──────────┐
          │   Box    │
          └──────────┘
```

กรณีนี้ควรกลับไปตรวจสอบ Label

---

# 35. ความแตกต่างระหว่าง Step 7 และ Step 9

### Step 7

ตรวจสอบ **Auto Label**

```text
Dataset
   ↓
Auto Label
   ↓
ตรวจ Bounding Box
```

เป็นการตรวจผลที่ Model สร้าง Label ให้

### Step 9

ตรวจสอบ **Final Dataset**

```text
Seed
  +
Auto Label
  ↓
P711_Final
  ↓
ตรวจ Bounding Box
```

ดังนั้น Step 9 เป็นการตรวจสอบข้อมูลหลังจากที่นำข้อมูลทั้งหมดมารวมเป็น Final Dataset แล้ว

---

# 36. Flow ของ Step 9

```text
P711_Final
     │
     ├── images/
     │
     └── labels/
            │
            ▼
       จับคู่ชื่อไฟล์
            │
            ▼
        อ่าน Image
            │
            ▼
        อ่าน Label
            │
            ▼
    Class X Y W H
            │
            ▼
   แปลงเป็น Pixel
            │
            ▼
   วาด Bounding Box
            │
            ▼
    แสดงผล 2 × 3
            │
            ▼
      ตรวจด้วยตา
```

---

# 37. สรุป Step 9

Step 9 ทำหน้าที่

1. เปิด `P711_Final`
2. ค้นหา Image ใน `images/`
3. เรียงลำดับ Image
4. แบ่ง Image ครั้งละ 6 รูป
5. อ่าน Image ด้วย OpenCV
6. หา Label ที่ชื่อเดียวกับ Image
7. อ่าน YOLO Label
8. ตรวจสอบว่ามีข้อมูลครบ 5 ค่า
9. แปลง Normalized Coordinate เป็น Pixel Coordinate
10. คำนวณมุมของ Bounding Box
11. วาด Bounding Box สีเขียว
12. แสดง Image ด้วย Matplotlib
13. แสดง 6 รูปต่อหน้า
14. ใช้สำหรับ Visual Verification ของ Final Dataset

**เป้าหมายของ Step 9 คือ**

```text
P711_Final
     ↓
ตรวจ Image + Label
     ↓
วาด Bounding Box
     ↓
Visual Inspection
     ↓
ตรวจสอบความถูกต้องก่อนนำ Dataset ไปใช้ต่อ
```
