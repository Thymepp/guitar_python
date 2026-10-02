# STEP 7 — VERIFY AUTO LABEL

## 1. จุดประสงค์ของ Step นี้

Step 7 ใช้สำหรับ **ตรวจสอบ Label ที่ถูกสร้างจาก Step 6**

โดยนำไฟล์ `.txt` ที่มีอยู่ใน Dataset มาอ่านค่า Bounding Box แล้ววาดกรอบลงบนรูปภาพด้วย OpenCV

แนวคิดหลักคือ

```text
Step 6
Auto Label
    │
    ▼
สร้าง .txt
    │
    ▼
Step 7
อ่าน .txt
    │
    ▼
อ่าน Image
    │
    ▼
แปลง YOLO Coordinate → Pixel Coordinate
    │
    ▼
วาด Bounding Box
    │
    ▼
แสดงภาพ 6 รูปต่อหน้า
```

จุดประสงค์สำคัญคือ **ดูด้วยตาว่า Auto Label ที่ Model สร้างขึ้นมานั้นวาง Bounding Box ถูกตำแหน่งหรือไม่**

---

# 2. Code ทั้งหมดของ Step 7

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt

DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

# รูปที่ Auto Label
labels = [
    p for p in DATASET.rglob("*.txt")
    if p.stem in {x.stem for x in DATASET.rglob("*")
                  if x.suffix.lower() in [".jpg", ".jpeg", ".png"]}
]

for n in range(0, len(labels), 6):

    fig, ax = plt.subplots(2, 3, figsize=(15, 9))
    ax = ax.ravel()

    for i, label in enumerate(labels[n:n+6]):

        img_path = next(
            p for p in DATASET.rglob("*")
            if p.stem == label.stem
            and p.suffix.lower() in [".jpg", ".jpeg", ".png"]
        )

        img = cv2.imread(str(img_path))
        h, w = img.shape[:2]

        for line in label.read_text().splitlines():
            d = line.split()

            if len(d) == 5:
                _, x, y, bw, bh = map(float, d)

                x1 = int((x - bw/2) * w)
                y1 = int((y - bh/2) * h)
                x2 = int((x + bw/2) * w)
                y2 = int((y + bh/2) * h)

                cv2.rectangle(img, (x1,y1), (x2,y2), (0,255,0), 2)

        ax[i].imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
        ax[i].axis("off")

    for a in ax[len(labels[n:n+6]):]:
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

ใช้ทั้งหมด 3 Library

### `Path`

ใช้จัดการ Folder และไฟล์

### `cv2`

คือ OpenCV ใช้สำหรับ

* อ่านรูป
* อ่านขนาดรูป
* วาด Bounding Box

### `matplotlib.pyplot`

ใช้สำหรับแสดงรูปภาพเป็น Grid

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset หลัก

ตัวอย่าง

```text
C:\Users\pawor\Desktop\P711Test
```

---

# 5. หา Label ที่ตรงกับรูปภาพ

```python
labels = [
    p for p in DATASET.rglob("*.txt")
    if p.stem in {x.stem for x in DATASET.rglob("*")
                  if x.suffix.lower() in [".jpg", ".jpeg", ".png"]}
]
```

ส่วนนี้ใช้ค้นหาไฟล์ `.txt` ที่มีชื่อเดียวกับ Image

ตัวอย่าง

```text
image001.jpg
image001.txt
```

เนื่องจาก

```python
p.stem
```

ของทั้งสองไฟล์คือ

```text
image001
```

จึงถือว่าเป็นคู่กัน

---

# 6. `DATASET.rglob("*.txt")`

```python
DATASET.rglob("*.txt")
```

ค้นหาไฟล์ `.txt` ทุกไฟล์ใน Dataset รวมถึง Subfolder

ตัวอย่าง

```text
P711Test/
├── image001.txt
├── image002.txt
└── folder/
    └── image003.txt
```

ทั้งหมดจะถูกค้นพบ

---

# 7. `p.stem`

```python
p.stem
```

ใช้เอาชื่อไฟล์โดยไม่เอานามสกุล

เช่น

```text
image001.txt
```

จะได้

```text
image001
```

ส่วน

```text
image001.jpg
```

ก็จะได้

```text
image001
```

ดังนั้นจึงใช้จับคู่ Image กับ Label ได้

---

# 8. ค้นหาเฉพาะ Image

ส่วนนี้

```python
{
    x.stem for x in DATASET.rglob("*")
    if x.suffix.lower() in [".jpg", ".jpeg", ".png"]
}
```

สร้าง Set ที่เก็บชื่อของ Image ทั้งหมด

เช่น

```text
{
    "image001",
    "image002",
    "image003"
}
```

จากนั้นนำไปตรวจสอบกับ

```python
p.stem
```

ของ Label

---

# 9. ผลของ `labels`

สุดท้ายตัวแปร

```python
labels
```

จะเก็บ Path ของ `.txt` ที่สามารถจับคู่กับ Image ได้

เช่น

```text
image001.txt
image002.txt
image003.txt
```

แนวคิดคือ

```text
Image                    Label

image001.jpg    ←→       image001.txt
image002.jpg    ←→       image002.txt
image003.jpg    ←→       image003.txt
```

---

# 10. แบ่งภาพครั้งละ 6 รูป

```python
for n in range(0, len(labels), 6):
```

วนข้อมูลทีละ 6 Label

ถ้ามี 18 รูป

```text
รอบที่ 1 → รูป 1-6
รอบที่ 2 → รูป 7-12
รอบที่ 3 → รูป 13-18
```

ถ้ามี 20 รูป

```text
รอบที่ 1 → รูป 1-6
รอบที่ 2 → รูป 7-12
รอบที่ 3 → รูป 13-18
รอบที่ 4 → รูป 19-20
```

---

# 11. สร้าง Grid 2 × 3

```python
fig, ax = plt.subplots(2, 3, figsize=(15, 9))
```

สร้างพื้นที่แสดงรูปจำนวน

```text
2 Rows × 3 Columns
```

หรือทั้งหมด

```text
6 ช่อง
```

หน้าตาประมาณ

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

เปลี่ยน Array แบบ 2 มิติ

```text
2 × 3
```

ให้กลายเป็น Array แบบ 1 มิติ

```text
[0, 1, 2, 3, 4, 5]
```

ทำให้สามารถใช้

```python
ax[i]
```

ได้ง่าย

---

# 13. วนทีละ Label

```python
for i, label in enumerate(labels[n:n+6]):
```

เลือก Label ชุดปัจจุบันสูงสุด 6 ไฟล์

เช่น

```text
labels[0:6]
```

แล้วใช้ `enumerate()` เพื่อให้ได้

```text
i      = ตำแหน่ง
label  = Path ของ Label
```

ตัวอย่าง

```text
i = 0 → image001.txt
i = 1 → image002.txt
i = 2 → image003.txt
```

---

# 14. หา Image ที่คู่กับ Label

```python
img_path = next(
    p for p in DATASET.rglob("*")
    if p.stem == label.stem
    and p.suffix.lower() in [".jpg", ".jpeg", ".png"]
)
```

ส่วนนี้ใช้หา Image ที่มีชื่อเดียวกับ Label

ถ้า Label คือ

```text
image003.txt
```

ค่า

```python
label.stem
```

คือ

```text
image003
```

โปรแกรมจึงค้นหา Image ที่มี

```text
stem == image003
```

และต้องมีนามสกุล

```text
.jpg
.jpeg
.png
```

---

# 15. อ่าน Image

```python
img = cv2.imread(str(img_path))
```

ใช้ OpenCV อ่านรูปเข้าไปเก็บใน

```python
img
```

---

# 16. อ่านขนาด Image

```python
h, w = img.shape[:2]
```

ดึง

```text
h = Height
w = Width
```

เช่น Image ขนาด

```text
1920 × 1080
```

จะได้ประมาณ

```text
w = 1920
h = 1080
```

ข้อมูลนี้จำเป็นสำหรับแปลง YOLO Coordinate ให้เป็น Pixel Coordinate

---

# 17. อ่านข้อมูลจาก Label

```python
for line in label.read_text().splitlines():
```

อ่านไฟล์ `.txt` ทีละบรรทัด

ตัวอย่าง Label

```text
0 0.512500 0.430000 0.250000 0.300000
1 0.700000 0.600000 0.150000 0.200000
```

แต่ละบรรทัดแทน Bounding Box หนึ่งอัน

---

# 18. แยกข้อมูลแต่ละช่อง

```python
d = line.split()
```

ตัวอย่าง

```text
0 0.512500 0.430000 0.250000 0.300000
```

จะกลายเป็น

```python
[
    "0",
    "0.512500",
    "0.430000",
    "0.250000",
    "0.300000"
]
```

---

# 19. ตรวจสอบว่ามีข้อมูลครบ 5 ค่า

```python
if len(d) == 5:
```

YOLO Detection Label ต้องมี

```text
Class
X
Y
Width
Height
```

รวมทั้งหมด 5 ค่า

ดังนั้นถ้าไม่ใช่ 5 ค่า โปรแกรมจะไม่วาด Bounding Box บรรทัดนั้น

---

# 20. แปลงข้อมูลเป็น Float

```python
_, x, y, bw, bh = map(float, d)
```

ข้อมูลเดิมเป็น String

จึงใช้

```python
map(float, d)
```

แปลงเป็นตัวเลข

ส่วน

```python
_
```

ใช้รับค่า Class ID แต่ใน Step นี้ไม่ได้ใช้งาน จึงไม่จำเป็นต้องเก็บชื่อ

ดังนั้นเหลือ

```text
x
y
bw
bh
```

คือ

```text
x  = Center X
y  = Center Y
bw = Box Width
bh = Box Height
```

---

# 21. แปลง Center Coordinate เป็นมุมซ้ายบน

```python
x1 = int((x - bw/2) * w)
y1 = int((y - bh/2) * h)
```

YOLO เก็บ Bounding Box แบบ

```text
Center X
Center Y
Width
Height
```

แต่ OpenCV ต้องการพิกัด

```text
ซ้ายบน
ขวาล่าง
```

ดังนั้นต้องคำนวณใหม่

สูตร

```text
x1 = (x - width/2) × image width
y1 = (y - height/2) × image height
```

---

# 22. คำนวณมุมขวาล่าง

```python
x2 = int((x + bw/2) * w)
y2 = int((y + bh/2) * h)
```

สูตร

```text
x2 = (x + width/2) × image width
y2 = (y + height/2) × image height
```

ดังนั้น Bounding Box จะได้

```text
(x1, y1)
     ┌─────────────────┐
     │                 │
     │     Object      │
     │                 │
     └─────────────────┘
                 (x2,y2)
```

---

# 23. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (x1,y1),
    (x2,y2),
    (0,255,0),
    2
)
```

ใช้ OpenCV วาดสี่เหลี่ยมบน Image

ค่าต่าง ๆ คือ

```text
img
```

รูปภาพที่จะวาด

```text
(x1,y1)
```

มุมซ้ายบน

```text
(x2,y2)
```

มุมขวาล่าง

```text
(0,255,0)
```

สีเขียวในรูปแบบ BGR

```text
2
```

ความหนาของเส้น

---

# 24. แปลง BGR → RGB

```python
ax[i].imshow(
    cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
)
```

OpenCV อ่านรูปเป็น

```text
BGR
```

แต่ Matplotlib ใช้

```text
RGB
```

ดังนั้นต้องแปลงด้วย

```python
cv2.cvtColor(...)
```

ถ้าไม่แปลง สีของภาพอาจแสดงผิด

---

# 25. ปิดแกนของกราฟ

```python
ax[i].axis("off")
```

ซ่อนแกน X/Y และตัวเลขรอบภาพ

ทำให้เห็นเฉพาะ Image และ Bounding Box

---

# 26. ซ่อนช่องที่ไม่มีรูป

```python
for a in ax[len(labels[n:n+6]):]:
    a.axis("off")
```

กรณีหน้าสุดท้ายมีรูปไม่ครบ 6 รูป

เช่นเหลือแค่ 2 รูป

```text
┌──────────┬──────────┬──────────┐
│ Image 1  │ Image 2  │          │
├──────────┼──────────┼──────────┤
│          │          │          │
└──────────┴──────────┴──────────┘
```

ช่องที่ไม่มีรูปจะถูกซ่อน

---

# 27. จัด Layout

```python
plt.tight_layout()
```

ปรับระยะห่างระหว่างภาพให้เหมาะสม

ช่วยลดปัญหาภาพหรือพื้นที่ของแต่ละช่องทับกัน

---

# 28. แสดงผล

```python
plt.show()
```

แสดงภาพทั้งหมด 6 รูปของ Batch ปัจจุบัน

ผู้ใช้สามารถดูว่า Bounding Box ที่ Auto Label สร้างขึ้นมานั้นอยู่ตรง Object หรือไม่

---

# 29. ตัวอย่างการตรวจสอบ

สมมติ Model สร้าง Label

```text
image001.txt
```

ภายใน

```text
0 0.500000 0.500000 0.300000 0.400000
```

Step 7 จะ

```text
image001.txt
       │
       ▼
อ่าน x,y,w,h
       │
       ▼
คำนวณ Pixel Coordinate
       │
       ▼
วาดกรอบสีเขียว
       │
       ▼
แสดง image001.jpg
```

ทำให้สามารถตรวจสอบด้วยสายตาได้ว่า

```text
       ┌─────────────┐
       │   Object    │
       │             │
       └─────────────┘
```

Bounding Box ครอบ Object ถูกต้องหรือไม่

---

# 30. Flow ของ Step 7

```text
Dataset
   │
   ├── Image
   │
   └── Label .txt
          │
          ▼
จับคู่ Image ↔ Label
          │
          ▼
อ่าน YOLO Label
          │
          ▼
Class X Y Width Height
          │
          ▼
แปลง Normalized Coordinate
เป็น Pixel Coordinate
          │
          ▼
OpenCV วาด Bounding Box
          │
          ▼
แสดงด้วย Matplotlib
          │
          ▼
6 รูปต่อหน้า
```

---

# 31. Step 7 สำคัญอย่างไร?

Step 6 เป็นการให้ Model **สร้าง Label โดยอัตโนมัติ**

ดังนั้น Label ที่ได้อาจมี

* Bounding Box ไม่ตรง Object
* ตรวจ Object ผิด
* ตรวจ Object เกิน
* ตรวจ Object ขาด
* Confidence ต่ำแต่ยังผ่าน Threshold
* Object บางประเภทถูกตรวจผิด Class

Step 7 จึงเป็นขั้นตอนสำหรับ **Visual Verification**

กล่าวง่าย ๆ คือ

```text
Step 6
"Model คิดว่าตรงนี้คือ Object"

          ↓

Step 7
"เปิดภาพดูว่า Model วาดถูกจริงหรือไม่"
```

---

# 32. สรุป Step 7

Step 7 ทำหน้าที่

1. ค้นหาไฟล์ `.txt`
2. ตรวจสอบว่า Label มี Image คู่กัน
3. แบ่งภาพครั้งละ 6 รูป
4. อ่าน Image
5. อ่าน YOLO Label
6. อ่าน `Class X Y Width Height`
7. แปลง Normalized Coordinate เป็น Pixel Coordinate
8. คำนวณมุมซ้ายบนและขวาล่าง
9. วาด Bounding Box ด้วย OpenCV
10. แปลง BGR เป็น RGB
11. แสดงผลด้วย Matplotlib
12. แสดงภาพเป็น Grid ขนาด 2×3
13. ใช้ตรวจสอบความถูกต้องของ Auto Label

**เป้าหมายของ Step 7 คือ**

```text
Auto Label จาก Step 6
          ↓
   Visual Inspection
          ↓
ตรวจว่ากรอบถูกต้องหรือไม่
          ↓
พร้อมสำหรับขั้นตอน Dataset ต่อไป
```
