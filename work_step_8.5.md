# STEP 8.5 — ตรวจสอบ P711_Final Dataset

## 1. หน้าที่ของ STEP 8.5

STEP 8.5 ใช้สำหรับ **ตรวจสอบ Dataset ที่สร้างเสร็จจาก STEP 8.0**

โดยอ่านข้อมูลจาก

```text
P711_Final/
├── images/
└── labels/
```

แล้วนำ Label ของแต่ละ Image มาวาด Bounding Box เพื่อให้ตรวจสอบด้วยสายตาอีกครั้ง

ลำดับการทำงานคือ

```text
STEP 8.0
Build Final Dataset
       ↓
P711_Final
       ↓
STEP 8.5
Visual Check Final Dataset
       ↓
ตรวจ Image + Label
       ↓
พร้อมสำหรับ Train Final
```

แตกต่างจาก STEP 7.0 ตรงที่

```text
STEP 7.0 → ตรวจ Auto Label
STEP 8.5 → ตรวจ Final Dataset ทั้งหมด
```

---

# 2. Import Library

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt
```

### `Path`

ใช้จัดการ Folder และ File

### `cv2`

ใช้

* อ่าน Image
* อ่านขนาด Image
* วาด Bounding Box
* แปลงสี BGR → RGB

### `matplotlib.pyplot`

ใช้แสดง Image ครั้งละ 6 รูป

---

# 3. กำหนด Dataset

```python
# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Dataset หลัก

```text
C:\Users\pawor\Desktop\P711Test
```

---

# 4. กำหนด Final Dataset

```python
FINAL = DATASET / "P711_Final"
```

จะได้

```text
C:\Users\pawor\Desktop\P711Test\P711_Final
```

STEP 8.5 จะตรวจเฉพาะข้อมูลภายใน `P711_Final`

---

# 5. ค้นหา Image

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

โดยรองรับ

```text
.jpg
.jpeg
.png
```

---

## 5.1 `iterdir()`

```python
(FINAL / "images").iterdir()
```

หมายถึงอ่านไฟล์/Folder **เฉพาะระดับโดยตรง**

ตัวอย่าง

```text
images/
├── A.jpg       ← เจอ
├── B.jpg       ← เจอ
├── C.png       ← เจอ
└── folder/
    └── D.jpg   ← ไม่ค้นลงไป
```

เหมาะกับ Final Dataset เพราะเราตั้งใจให้ Image อยู่โดยตรงใน `images/`

---

## 5.2 `sorted()`

```python
sorted([...])
```

ใช้เรียงลำดับชื่อไฟล์

เช่น

```text
image001.jpg
image002.jpg
image003.jpg
```

ทำให้การเปิดดูแต่ละรอบมีลำดับที่คงที่มากขึ้น

---

# 6. แบ่ง Image ครั้งละ 6 รูป

```python
for n in range(0, len(images), 6):
```

แบ่ง Image เป็นชุดละ 6

เช่นมี 14 รูป

```text
รอบที่ 1 → 1-6
รอบที่ 2 → 7-12
รอบที่ 3 → 13-14
```

---

# 7. สร้างหน้าต่าง 2 × 3

```python
fig, ax = plt.subplots(2, 3, figsize=(15, 9))
```

สร้างพื้นที่แสดงผล 6 ช่อง

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │ Image 6 │
└─────────┴─────────┴─────────┘
```

---

# 8. แปลง Axes เป็น 1 มิติ

```python
ax = ax.ravel()
```

ทำให้สามารถเรียกช่องต่าง ๆ เป็น

```python
ax[0]
ax[1]
ax[2]
ax[3]
ax[4]
ax[5]
```

แทนการใช้

```python
ax[0][0]
ax[0][1]
...
```

---

# 9. เลือก Image ปัจจุบัน

```python
current = images[n:n+6]
```

เลือก Image สูงสุด 6 รูปสำหรับรอบปัจจุบัน

ตัวอย่าง

```python
images[0:6]
```

คือ 6 รูปแรก

และ

```python
images[6:12]
```

คือชุดถัดไป

---

# 10. วนอ่าน Image

```python
for i, img_path in enumerate(current):
```

ได้ข้อมูล 2 ตัว

```text
i
↓
ตำแหน่งของช่อง

img_path
↓
Path ของ Image
```

ตัวอย่าง

```text
i = 0 → Image 1
i = 1 → Image 2
i = 2 → Image 3
```

---

# 11. อ่าน Image ด้วย OpenCV

```python
img = cv2.imread(str(img_path))
```

OpenCV อ่าน Image เข้าเป็น NumPy Array

จากนั้นตรวจสอบว่าอ่านสำเร็จหรือไม่

```python
if img is None:
    continue
```

ถ้าอ่าน Image ไม่สำเร็จ จะข้ามรูปนั้นทันที

---

# 12. อ่านขนาด Image

```python
h, w = img.shape[:2]
```

ได้

```text
h = Height
w = Width
```

ตัวอย่าง

```text
Image = 1920 × 1080

w = 1920
h = 1080
```

ข้อมูลนี้จำเป็นสำหรับแปลง YOLO Coordinate กลับเป็น Pixel

---

# 13. หา Label คู่กับ Image

```python
label = FINAL / "labels" / f"{img_path.stem}.txt"
```

สมมติ Image คือ

```text
ABC001.jpg
```

`img_path.stem` คือ

```text
ABC001
```

ดังนั้น Label ที่จะค้นหาคือ

```text
P711_Final/labels/ABC001.txt
```

โครงสร้างคู่กันคือ

```text
images/
└── ABC001.jpg

labels/
└── ABC001.txt
```

---

# 14. ตรวจสอบว่ามี Label หรือไม่

```python
if label.exists():
```

ถ้ามี Label จึงอ่าน Bounding Box

ถ้าไม่มี Label โค้ดจะยังแสดง Image ได้ แต่จะไม่มี Bounding Box

นี่มีประโยชน์สำหรับ **Empty / Negative Image**

เช่น

```text
empty001.jpg
```

ไม่มี

```text
empty001.txt
```

ก็ยังสามารถนำรูปมาแสดงเพื่อตรวจสอบได้

---

# 15. อ่าน Label

```python
for line in label.read_text(
    encoding="utf-8-sig"
).splitlines():
```

อ่านไฟล์ Label ทีละบรรทัด

ตัวอย่าง

```text
0 0.500000 0.400000 0.200000 0.300000
```

---

## ทำไมใช้ `utf-8-sig`

```python
encoding="utf-8-sig"
```

ช่วยรองรับไฟล์ UTF-8 ที่มี BOM

ทำให้การอ่าน Label มีความปลอดภัยมากขึ้น โดยเฉพาะไฟล์ที่อาจถูกสร้างจากโปรแกรม Windows บางชนิด

---

# 16. แยกข้อมูล

```python
d = line.split()
```

ตัวอย่าง

```text
0 0.5 0.4 0.2 0.3
```

จะกลายเป็น

```python
[
    "0",
    "0.5",
    "0.4",
    "0.2",
    "0.3"
]
```

---

# 17. ตรวจสอบรูปแบบ Label

```python
if len(d) != 5:
    continue
```

YOLO Detection Label ต้องมี 5 ค่า

```text
class
x
y
width
height
```

ถ้าไม่ครบ 5 ค่า จะข้ามบรรทัดนั้น

---

# 18. แปลงข้อมูลเป็นตัวเลข

```python
_, x, y, bw, bh = map(float, d)
```

ตัวอย่าง

```text
0 0.5 0.4 0.2 0.3
```

ได้

```text
_  = 0
x  = 0.5
y  = 0.4
bw = 0.2
bh = 0.3
```

`_` คือ Class ID แต่ STEP 8.5 ยังไม่ได้ใช้ในการแสดงชื่อ Class

---

# 19. แปลง YOLO Coordinate เป็น Pixel

YOLO ใช้รูปแบบ

```text
class x_center y_center width height
```

และค่าตำแหน่งเป็น Normalized Coordinate

ดังนั้นต้องแปลงเป็น

```text
x1 y1 x2 y2
```

สำหรับ OpenCV

---

## 19.1 จุดซ้ายบน

```python
x1 = int((x - bw/2) * w)
y1 = int((y - bh/2) * h)
```

สูตร

```text
x1 = (x_center - width/2) × image_width

y1 = (y_center - height/2) × image_height
```

---

## 19.2 จุดขวาล่าง

```python
x2 = int((x + bw/2) * w)
y2 = int((y + bh/2) * h)
```

สูตร

```text
x2 = (x_center + width/2) × image_width

y2 = (y_center + height/2) × image_height
```

สุดท้ายได้

```text
(x1,y1)
      ↓
┌────────────┐
│   OBJECT   │
└────────────┘
      ↑
(x2,y2)
```

---

# 20. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (x1, y1),
    (x2, y2),
    (0, 255, 0),
    2
)
```

ใช้ OpenCV วาดกรอบ

องค์ประกอบคือ

```text
img
↓
รูปภาพ

(x1,y1)
↓
มุมซ้ายบน

(x2,y2)
↓
มุมขวาล่าง

(0,255,0)
↓
สีเขียว

2
↓
ความหนา 2 Pixel
```

---

# 21. แสดง Image

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

จึงต้องแปลงด้วย

```python
cv2.cvtColor(
    img,
    cv2.COLOR_BGR2RGB
)
```

ก่อนแสดง

---

# 22. ปิด Axis

```python
ax[i].axis("off")
```

ซ่อนแกน X/Y เพื่อให้เห็นเฉพาะ Image และ Bounding Box

---

# 23. ซ่อนช่องที่ไม่ได้ใช้

```python
for a in ax[len(current):]:
    a.axis("off")
```

กรณีชุดสุดท้ายมี Image ไม่ครบ 6 รูป

เช่นเหลือ 2 รูป

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │         │
├─────────┼─────────┼─────────┤
│         │         │         │
└─────────┴─────────┴─────────┘
```

ช่องที่เหลือจะถูกซ่อน

---

# 24. จัด Layout และแสดงผล

```python
plt.tight_layout()
plt.show()
```

`tight_layout()` ปรับระยะห่างของแต่ละช่อง

จากนั้น

```python
plt.show()
```

แสดงผลออกมา

---

# 25. STEP 8.5 ต่างจาก STEP 7.0 อย่างไร?

นี่เป็นจุดสำคัญของ Pipeline

| ขั้นตอน  | ตรวจอะไร                 |
| -------- | ------------------------ |
| STEP 7.0 | ตรวจ Auto Label          |
| STEP 8.0 | รวม Seed + Auto Label    |
| STEP 8.5 | ตรวจ Final Dataset       |
| STEP 9.0 | เตรียม/Train Final Model |

### STEP 7.0

ตรวจว่า

```text
Model → Auto Label
```

สร้าง Bounding Box ถูกหรือไม่

### STEP 8.0

นำข้อมูลที่ผ่านขั้นตอนก่อนหน้ามารวมเป็น

```text
P711_Final
```

### STEP 8.5

ตรวจอีกครั้งว่า

```text
P711_Final
```

มี Image และ Label ที่ถูกต้อง

---

# 26. สิ่งที่ควรตรวจใน STEP 8.5

ขณะดูภาพควรตรวจ 4 เรื่องหลัก

### ① Bounding Box ครอบวัตถุหรือไม่

```text
ถูกต้อง

┌──────────────┐
│    Object    │
└──────────────┘
```

### ② Bounding Box หลุดวัตถุหรือไม่

```text
ผิด

┌──────┐
│      │
│ Object ─────
└──────┘
```

### ③ มีวัตถุที่ไม่มี Label หรือไม่

เช่นภาพมี 3 Object

```text
Object A ✓
Object B ✓
Object C ✗
```

แสดงว่า Label ยังไม่ครบ

### ④ มี False Positive หรือไม่

เช่นมี Bounding Box อยู่ในพื้นที่ที่ไม่มี Object

```text
┌────────┐
│        │
│        │ ← ไม่มี Object
└────────┘
```

ควรแก้ Label ก่อนนำไป Train

---

# 27. จุดสำคัญเรื่อง Empty Image

STEP 8.0 สามารถมี Image ที่ไม่มี Label ได้ เช่น

```text
P711_Final/
├── images/
│   └── empty001.jpg
│
└── labels/
    └── ไม่มี empty001.txt
```

STEP 8.5 จะยังเปิด

```text
empty001.jpg
```

ได้ เพราะการหา Image กับการหา Label แยกจากกัน

```python
if label.exists():
```

ถ้าไม่มี Label ก็แค่ไม่วาด Bounding Box

นี่ทำให้สามารถตรวจ **Negative Image** ได้ด้วย

---

# 28. จุดที่ควรระวัง

STEP 8.5 ใช้

```python
label = FINAL / "labels" / f"{img_path.stem}.txt"
```

ดังนั้นระบบจับคู่ด้วย **ชื่อไฟล์**

ตัวอย่าง

```text
abc123.jpg
abc123.txt
```

จับคู่กัน

แต่

```text
abc123.jpg
ABC123.txt
```

บน Windows อาจมีพฤติกรรมเรื่องชื่อไฟล์ที่แตกต่างจากระบบที่ Case-sensitive ดังนั้นควรตั้งชื่อ Image/Label ให้ตรงกันเสมอ

---

# 29. ข้อจำกัดของ STEP 8.5

โค้ดนี้ตรวจเฉพาะว่า

```text
Image → Label
```

และวาด Bounding Box

ยังไม่ได้ตรวจเชิงโครงสร้าง เช่น

* Class ID มีอยู่จริงหรือไม่
* ค่า `x,y,w,h` อยู่ในช่วง `0–1` หรือไม่
* Bounding Box เกินขอบ Image หรือไม่
* Label ที่ไม่มี Image มีหรือไม่
* Image ที่ไม่มี Label มีกี่รูป
* Label เสีย/ผิด Format กี่ไฟล์

ดังนั้น STEP 8.5 เป็น **Visual Check** มากกว่า **Dataset Validation**

---

# 30. Pipeline ถึง STEP 8.5

```text
STEP 1
Check Dataset
       ↓
STEP 2.0
Visualize Original Labels
       ↓
STEP 2.5
Select Empty Images
       ↓
STEP 3.0
Build Seed Dataset
       ↓
STEP 4.0
Build data.yaml
       ↓
STEP 5.0
Train Seed Model
       ↓
STEP 6.0
Auto Label
       ↓
STEP 7.0
Check Auto Label
       ↓
STEP 8.0
Build Final Dataset
       ↓
STEP 8.5
Check Final Dataset
       ↓
P711_Final
       ↓
Train Final Model
```

---

# 31. สรุป STEP 8.5

| Code                  | หน้าที่                    |
| --------------------- | -------------------------- |
| `FINAL`               | กำหนด Final Dataset        |
| `iterdir()`           | หา Image ใน `Final/images` |
| `sorted()`            | เรียง Image                |
| `range(..., 6)`       | แบ่งครั้งละ 6 รูป          |
| `imread()`            | อ่าน Image                 |
| `shape[:2]`           | อ่าน Height/Width          |
| `with_suffix(".txt")` | หา Label คู่กับ Image      |
| `read_text()`         | อ่าน Label                 |
| `split()`             | แยกข้อมูล YOLO             |
| `map(float)`          | แปลงเป็นตัวเลข             |
| `x1,y1,x2,y2`         | แปลง Normalized → Pixel    |
| `rectangle()`         | วาด Bounding Box           |
| `cvtColor()`          | BGR → RGB                  |
| `imshow()`            | แสดง Image                 |
| `show()`              | เปิดผลลัพธ์                |

## สรุปสั้นที่สุด

```text
STEP 8.0
สร้าง P711_Final
       ↓
STEP 8.5
เปิดทุก Image ใน P711_Final
       ↓
อ่าน Label ที่ตรงกับ Image
       ↓
แปลง YOLO Coordinate
       ↓
วาด Bounding Box
       ↓
ตรวจด้วยสายตา
       ↓
พร้อมเข้าสู่ Final Training
```

ดังนั้น **STEP 8.5 = Final Visual Quality Check** ก่อนนำ `P711_Final` ไป Train Model รอบสุดท้าย
