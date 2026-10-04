# STEP 7.0 — ตรวจสอบผล Auto Label

## 1. หน้าที่ของ STEP 7.0

STEP 7.0 ใช้สำหรับ **ตรวจสอบ Label ที่ถูกสร้างโดย STEP 6.0 (Auto Label)** ด้วยการนำไฟล์ `.txt` ที่มีอยู่มาอ่านค่า Bounding Box แล้ววาดกรอบลงบนรูปภาพ

ลำดับการทำงานคือ

```text
STEP 6.0
    ↓
YOLO Model
    ↓
สร้างไฟล์ .txt
    ↓
STEP 7.0
    ↓
อ่าน .txt
    ↓
แปลง YOLO Coordinate
    ↓
วาด Bounding Box
    ↓
แสดงรูป 6 รูปต่อหน้า
    ↓
ตรวจสอบด้วยสายตา
```

จุดประสงค์สำคัญคือ **ตรวจว่า Auto Label วาดตำแหน่งวัตถุถูกต้องหรือไม่**

---

# 2. Import Library

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt
```

### `Path`

ใช้จัดการไฟล์และโฟลเดอร์

```python
Path(...)
```

ทำให้สามารถใช้คำสั่ง เช่น

```python
.rglob()
.stem
.suffix
.read_text()
```

ได้สะดวก

### `cv2`

ใช้สำหรับอ่านรูปภาพและวาด Bounding Box

```python
cv2.imread()
cv2.rectangle()
cv2.cvtColor()
```

### `matplotlib`

ใช้แสดงรูปภาพหลายรูปพร้อมกัน

```python
plt.subplots()
plt.imshow()
plt.show()
```

---

# 3. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset หลัก

```text
C:\Users\pawor\Desktop\P711Test
```

โดย STEP 7.0 จะค้นหาไฟล์ภายใน Dataset แบบ Recursive ด้วย `rglob()`.

---

# 4. ค้นหา Label ที่จับคู่กับ Image

```python
labels = [
    p for p in DATASET.rglob("*.txt")
    if p.stem in {x.stem for x in DATASET.rglob("*")
                  if x.suffix.lower() in [".jpg", ".jpeg", ".png"]}
]
```

ส่วนนี้ทำหน้าที่หาไฟล์ `.txt` ที่มีชื่อเดียวกับรูปภาพ

ตัวอย่าง

```text
image001.jpg
image001.txt
```

จะถือว่าเป็นคู่กัน เพราะ

```python
p.stem
```

ของทั้งสองไฟล์คือ

```text
image001
```

---

## 4.1 `rglob("*.txt")`

```python
DATASET.rglob("*.txt")
```

ค้นหาไฟล์ `.txt` ทุกระดับภายใน Dataset

เช่น

```text
P711Test/
├── image001.jpg
├── image001.txt
├── folder1/
│   ├── image002.jpg
│   └── image002.txt
└── folder2/
    └── image003.txt
```

จะสามารถค้นหา `.txt` ทั้งหมดได้

---

# 5. ตรวจสอบว่า Label มี Image คู่กันหรือไม่

ส่วนนี้

```python
{x.stem for x in DATASET.rglob("*")
 if x.suffix.lower() in [".jpg", ".jpeg", ".png"]}
```

สร้าง `set` ของชื่อรูปภาพ

ตัวอย่าง

```text
image001.jpg
image002.jpg
image003.jpg
```

จะได้

```python
{
    "image001",
    "image002",
    "image003"
}
```

จากนั้น

```python
if p.stem in ...
```

จะเลือกเฉพาะ `.txt` ที่มีรูปภาพคู่กัน

---

# 6. แบ่งรูปทีละ 6 รูป

```python
for n in range(0, len(labels), 6):
```

แบ่ง Label ออกเป็นชุดละ 6

ตัวอย่างมี 20 Labels

```text
รอบที่ 1 → 0 - 5
รอบที่ 2 → 6 - 11
รอบที่ 3 → 12 - 17
รอบที่ 4 → 18 - 19
```

เหตุผลที่ใช้ 6 เพราะด้านล่างสร้าง Grid ขนาด

```text
2 × 3
```

หรือทั้งหมด 6 ช่อง

---

# 7. สร้างพื้นที่แสดงรูป

```python
fig, ax = plt.subplots(2, 3, figsize=(15, 9))
```

สร้างหน้าต่างสำหรับแสดงรูป

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │ Image 6 │
└─────────┴─────────┴─────────┘
```

### `2, 3`

หมายถึง

```text
2 แถว
3 คอลัมน์
```

รวม

```text
2 × 3 = 6 รูป
```

### `figsize=(15, 9)`

กำหนดขนาด Figure

```text
กว้าง = 15
สูง   = 9
```

---

# 8. แปลง Array ของ Axes ให้เป็น 1 มิติ

```python
ax = ax.ravel()
```

ก่อนใช้ `ravel()` จะมีลักษณะคล้าย

```python
ax[0][0]
ax[0][1]
ax[0][2]
ax[1][0]
ax[1][1]
ax[1][2]
```

หลังใช้

```python
ax = ax.ravel()
```

จะใช้ง่ายเป็น

```python
ax[0]
ax[1]
ax[2]
ax[3]
ax[4]
ax[5]
```

ทำให้สามารถใช้

```python
for i, label in enumerate(...):
```

ได้สะดวก

---

# 9. เลือก Label ครั้งละ 6

```python
for i, label in enumerate(labels[n:n+6]):
```

ตัวอย่าง

```python
labels[0:6]
```

คือรูปที่ 1–6

รอบต่อไป

```python
labels[6:12]
```

คือรูปที่ 7–12

`enumerate()` ทำให้ได้ทั้ง

```text
i     = ตำแหน่งช่อง
label = ไฟล์ Label
```

เช่น

```text
i = 0 → label รูปที่ 1
i = 1 → label รูปที่ 2
i = 2 → label รูปที่ 3
```

---

# 10. ค้นหา Image ที่ตรงกับ Label

```python
img_path = next(
    p for p in DATASET.rglob("*")
    if p.stem == label.stem
    and p.suffix.lower() in [".jpg", ".jpeg", ".png"]
)
```

สมมติ Label คือ

```text
ABC123.txt
```

ดังนั้น

```python
label.stem
```

คือ

```text
ABC123
```

โค้ดจะค้นหารูปที่มี

```text
ABC123.jpg
ABC123.jpeg
ABC123.png
```

และเลือกไฟล์แรกที่พบด้วย

```python
next(...)
```

---

# 11. อ่าน Image

```python
img = cv2.imread(str(img_path))
```

OpenCV อ่านรูปเข้ามาเป็น NumPy Array

เช่น

```text
Height × Width × Channel
```

จากนั้น

```python
h, w = img.shape[:2]
```

จะได้

```text
h = ความสูงของรูป
w = ความกว้างของรูป
```

ตัวอย่าง

```text
Image = 1920 × 1080

h = 1080
w = 1920
```

---

# 12. อ่านข้อมูล YOLO Label

```python
for line in label.read_text().splitlines():
```

อ่านไฟล์ `.txt` ทีละบรรทัด

ตัวอย่าง YOLO Label

```text
0 0.500000 0.400000 0.200000 0.300000
```

โครงสร้างคือ

```text
class_id
x_center
y_center
width
height
```

โดยค่า Coordinate ของ YOLO เป็น **Normalized Coordinate**

อยู่ในช่วงประมาณ

```text
0.0 - 1.0
```

---

# 13. แยกข้อมูลแต่ละบรรทัด

```python
d = line.split()
```

จาก

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

# 14. ตรวจสอบว่ามีข้อมูลครบ 5 ค่า

```python
if len(d) == 5:
```

YOLO Detection Label ต้องมี 5 ค่า

```text
class
x
y
width
height
```

ดังนั้นถ้าไม่ครบ 5 ค่า จะไม่เข้าไปวาด Bounding Box

---

# 15. แปลง String เป็น Float

```python
_, x, y, bw, bh = map(float, d)
```

ตัวอย่าง

```text
0 0.5 0.4 0.2 0.3
```

จะได้

```text
_  = 0
x  = 0.5
y  = 0.4
bw = 0.2
bh = 0.3
```

ตัวแปร `_` คือ Class ID

แต่ใน STEP 7.0 ยังไม่ได้ใช้ Class ID ในการแสดงผล

---

# 16. แปลง YOLO Coordinate เป็น Pixel

YOLO เก็บ Bounding Box เป็น

```text
x_center
y_center
width
height
```

และเป็นค่าที่ Normalize แล้ว

แต่ OpenCV ต้องการ

```text
x1
y1
x2
y2
```

เป็น Pixel

จึงต้องแปลง

---

## 16.1 หาจุดซ้ายบน

```python
x1 = int((x - bw/2) * w)
y1 = int((y - bh/2) * h)
```

หลักการคือ

```text
x1 = x_center - width/2
y1 = y_center - height/2
```

แล้วคูณด้วยขนาดรูป

```text
x1 × image width
y1 × image height
```

---

## 16.2 หาจุดขวาล่าง

```python
x2 = int((x + bw/2) * w)
y2 = int((y + bh/2) * h)
```

หลักการคือ

```text
x2 = x_center + width/2
y2 = y_center + height/2
```

แล้วแปลงเป็น Pixel

ดังนั้น

```text
YOLO

x y w h
  ↓
Pixel

x1 y1 x2 y2
```

---

# 17. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (x1,y1),
    (x2,y2),
    (0,255,0),
    2
)
```

ใช้ OpenCV วาดสี่เหลี่ยม

รูปแบบคือ

```python
cv2.rectangle(
    image,
    จุดซ้ายบน,
    จุดขวาล่าง,
    สี,
    ความหนา
)
```

ในโค้ดนี้

```python
(0,255,0)
```

คือสีเขียวในระบบสี BGR ของ OpenCV

และ

```python
2
```

คือความหนาของเส้น 2 Pixel

---

# 18. แสดง Image ใน Matplotlib

```python
ax[i].imshow(
    cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
)
```

OpenCV อ่านภาพเป็น

```text
BGR
```

แต่ Matplotlib ใช้

```text
RGB
```

ดังนั้นต้องแปลงด้วย

```python
cv2.cvtColor(
    img,
    cv2.COLOR_BGR2RGB
)
```

ก่อนแสดง

---

# 19. ปิด Axis

```python
ax[i].axis("off")
```

ซ่อนแกน X/Y

ทำให้ภาพดูสะอาดขึ้น

แทนที่จะเห็น

```text
0 100 200 300 ...
```

รอบรูป ก็จะแสดงเฉพาะภาพ

---

# 20. ซ่อนช่องที่ไม่มีรูป

```python
for a in ax[len(labels[n:n+6]):]:
    a.axis("off")
```

กรณีหน้าสุดท้ายมีไม่ครบ 6 รูป

เช่นมีเหลือแค่ 2 รูป

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │         │
├─────────┼─────────┼─────────┤
│         │         │         │
└─────────┴─────────┴─────────┘
```

โค้ดจะซ่อนช่องที่เหลือ

ทำให้ไม่เห็นช่องว่างที่ไม่จำเป็น

---

# 21. จัด Layout

```python
plt.tight_layout()
```

ปรับระยะห่างระหว่างรูปอัตโนมัติ

ช่วยลดปัญหารูปหรือข้อความชนกัน

---

# 22. แสดงผล

```python
plt.show()
```

เปิดหน้าต่างแสดงภาพ

ผู้ใช้สามารถตรวจสอบ Bounding Box ด้วยสายตาได้

---

# 23. ตัวอย่างผลลัพธ์

สมมติ STEP 6.0 สร้าง

```text
image001.txt
image002.txt
image003.txt
```

STEP 7.0 จะนำ Label เหล่านี้มาอ่าน แล้วแสดงประมาณ

```text
┌──────────────┬──────────────┬──────────────┐
│   Image 001  │   Image 002  │   Image 003  │
│    ┌────┐    │       ┌───┐  │    ┌─────┐   │
│    │Obj │    │       │Obj│  │    │ Obj │   │
│    └────┘    │       └───┘  │    └─────┘   │
├──────────────┼──────────────┼──────────────┤
│   Image 004  │   Image 005  │   Image 006  │
└──────────────┴──────────────┴──────────────┘
```

---

# 24. ความสัมพันธ์กับ STEP 6.0

STEP 6.0 ทำหน้าที่

```text
Image
  ↓
YOLO Model
  ↓
Prediction
  ↓
YOLO .txt
```

STEP 7.0 ทำหน้าที่

```text
YOLO .txt
  ↓
อ่าน Coordinate
  ↓
แปลงกลับเป็น Pixel
  ↓
วาด Bounding Box
  ↓
ตรวจสอบผล
```

ดังนั้น STEP 7.0 เป็นขั้นตอน **Quality Check ของ Auto Label**

---

# 25. จุดสำคัญที่ควรตรวจสอบ

หลังจากเปิดภาพ ให้ดูว่า

### ① Bounding Box ครอบวัตถุถูกต้องหรือไม่

```text
ถูกต้อง
┌──────────┐
│  OBJECT  │
└──────────┘
```

ถ้ากรอบหลุดออกจากวัตถุ อาจต้องแก้ Label

### ② มีวัตถุที่ Model ตรวจไม่เจอหรือไม่

เช่นในภาพมี 3 วัตถุ

```text
Object A ✓
Object B ✓
Object C ✗
```

แสดงว่า Auto Label ยังไม่สมบูรณ์

### ③ มี False Positive หรือไม่

เช่น Model วาดกรอบในบริเวณที่ไม่มีวัตถุ

```text
┌───────┐
│       │  ← ไม่มี Object
└───────┘
```

ควรตรวจสอบและแก้ไขก่อนนำไป Train ต่อ

### ④ Empty Image

ถ้ารูปไม่มีวัตถุและไม่มี Detection

STEP 6.0 จะสร้าง `.txt` ที่ว่างเปล่าได้

STEP 7.0 จะไม่มี Bounding Box ให้แสดง ซึ่งถือเป็นผลที่ถูกต้องสำหรับ Negative Image

---

# 26. จุดที่ควรปรับปรุงในโค้ด

โค้ดปัจจุบันทำงานได้ แต่ส่วนนี้

```python
{x.stem for x in DATASET.rglob("*")
 if x.suffix.lower() in [".jpg", ".jpeg", ".png"]}
```

จะค้นหารูปใหม่ทุกครั้งที่ตรวจ Label

และด้านล่างก็มีการ

```python
DATASET.rglob("*")
```

ซ้ำเพื่อหา Image อีกครั้ง

ดังนั้นถ้า Dataset มีจำนวนรูปมาก จะค่อนข้างเสียเวลา

สามารถสร้าง Dictionary ของ Image ไว้ครั้งเดียวได้ เช่น

```python
images = {
    p.stem: p
    for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
}
```

แล้วใช้

```python
if p.stem in images
```

และ

```python
img_path = images[label.stem]
```

จะเร็วและอ่านง่ายกว่า

---

# 27. สรุป STEP 7.0

| ส่วน                | หน้าที่                   |
| ------------------- | ------------------------- |
| `rglob("*.txt")`    | หา Label                  |
| `p.stem`            | ใช้จับคู่ Image กับ Label |
| `read_text()`       | อ่าน YOLO Label           |
| `split()`           | แยกข้อมูล                 |
| `map(float, d)`     | แปลงข้อมูลเป็นตัวเลข      |
| `x,y,bw,bh`         | YOLO Bounding Box         |
| `x1,y1,x2,y2`       | Pixel Bounding Box        |
| `cv2.rectangle()`   | วาดกรอบ                   |
| `cv2.cvtColor()`    | BGR → RGB                 |
| `plt.subplots(2,3)` | แสดง 6 รูป                |
| `plt.show()`        | แสดงผล                    |

## ภาพรวม Pipeline

```text
STEP 1
ตรวจ Dataset
    ↓
STEP 2.0
ดู Label เดิม
    ↓
STEP 2.5
เลือก Empty Image
    ↓
STEP 3.0
สร้าง Seed Dataset
    ↓
STEP 4.0
สร้าง data.yaml
    ↓
STEP 5.0
Train Seed Model
    ↓
STEP 6.0
Auto Label
    ↓
STEP 7.0
ตรวจสอบ Auto Label
    ↓
แก้ Label ที่ผิด
    ↓
นำ Dataset ที่ผ่านการตรวจสอบ
ไป Train Model ต่อ
```

**สรุปสั้น ๆ:** STEP 7.0 คือขั้นตอน **Visual Verification** หลัง Auto Label โดยนำ `.txt` ที่โมเดลสร้างขึ้นมาแปลง Coordinate กลับเป็น Pixel แล้ววาด Bounding Box เพื่อให้เราตรวจสอบว่าโมเดลติด Label ถูกตำแหน่งและครบถ้วนหรือไม่
