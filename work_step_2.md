# WORK STEP 2 — ตรวจสอบและแสดง YOLO Bounding Box

## 1. Import Library

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt
```

### `Path`

```python
from pathlib import Path
```

ใช้สำหรับจัดการ **ไฟล์และโฟลเดอร์ (Path)**

---

### `cv2`

```python
import cv2
```

คือ **OpenCV**

ใช้สำหรับจัดการรูปภาพ เช่น

* อ่านรูปภาพ
* วาด Bounding Box
* แปลงสีของรูปภาพ

ฟังก์ชันที่ใช้ใน Step นี้ ได้แก่

```python
cv2.imread()
cv2.rectangle()
cv2.cvtColor()
```

---

### `matplotlib`

```python
import matplotlib.pyplot as plt
```

ใช้สำหรับแสดงรูปภาพหลายรูปพร้อมกัน

ฟังก์ชันที่ใช้ เช่น

```python
plt.subplots()
plt.tight_layout()
plt.show()
```

---

# 2. กำหนดตำแหน่ง Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset ที่ต้องการตรวจสอบ

ตัวแปร `DATASET` จะเก็บ Path ของโฟลเดอร์

```text
C:\Users\pawor\Desktop\P711Test
```

ตัว `r` ด้านหน้า String เรียกว่า **Raw String**

```python
r"C:\Users\pawor\Desktop\P711Test"
```

ช่วยให้ `\` ใน Windows Path ไม่ถูกตีความเป็น Escape Character

---

# 3. ค้นหา Image

```python
imgs = {p.stem:p for p in DATASET.rglob("*")
        if p.suffix.lower() in [".jpg",".jpeg",".png"]}
```

ส่วนนี้ใช้ค้นหารูปภาพทั้งหมดใน Dataset และ Subfolder

## `rglob("*")`

```python
DATASET.rglob("*")
```

หมายถึงค้นหาไฟล์และโฟลเดอร์ทั้งหมดแบบ **Recursive**

คือค้นหาทั้งใน Dataset และโฟลเดอร์ย่อยทั้งหมด

ตัวอย่าง:

```text
P711Test/
├── train/
│   ├── 001.jpg
│   └── 002.jpg
├── valid/
│   └── 003.png
└── test/
    └── 004.jpeg
```

`rglob("*")` สามารถค้นหาไฟล์เหล่านี้ได้ทั้งหมด

---

## `p.suffix`

```python
p.suffix
```

ใช้ดู Extension ของไฟล์

ตัวอย่าง:

```text
001.jpg  → .jpg
002.png  → .png
003.txt  → .txt
```

---

## `.lower()`

```python
p.suffix.lower()
```

แปลง Extension ให้เป็นตัวพิมพ์เล็ก

เช่น

```text
.JPG → .jpg
.PNG → .png
```

ทำให้สามารถตรวจสอบ Extension ได้โดยไม่ต้องสนใจตัวพิมพ์เล็กหรือใหญ่

---

## ตรวจสอบประเภท Image

```python
if p.suffix.lower() in [".jpg",".jpeg",".png"]
```

หมายถึง:

> ถ้าไฟล์มี Extension เป็น `.jpg`, `.jpeg` หรือ `.png` ให้เก็บไฟล์นั้น

---

# 4. `p.stem`

```python
p.stem
```

ใช้เอาชื่อไฟล์โดย **ไม่รวม Extension**

ตัวอย่าง:

```text
image001.jpg
```

จะได้:

```text
image001
```

ดังนั้นถ้ามี:

```text
001.jpg
001.txt
```

ทั้งสองไฟล์จะมี

```python
p.stem
```

เป็น

```text
001
```

ทำให้สามารถจับคู่ Image กับ Label ได้

---

# 5. สร้าง Dictionary ของ Image

```python
imgs = {p.stem:p for p in DATASET.rglob("*")
        if p.suffix.lower() in [".jpg",".jpeg",".png"]}
```

Dictionary จะมีรูปแบบ:

```text
key : value
```

ตัวอย่าง:

```python
{
    "001": Path("001.jpg"),
    "002": Path("002.jpg"),
    "003": Path("003.png")
}
```

ดังนั้นสามารถเรียกรูปจากชื่อได้ เช่น

```python
imgs["001"]
```

จะได้ Path ของ

```text
001.jpg
```

จุดประสงค์หลักคือใช้สำหรับจับคู่:

```text
001.jpg ←→ 001.txt
002.jpg ←→ 002.txt
003.jpg ←→ 003.txt
```

---

# 6. ค้นหา Label

```python
labels = sorted([p for p in DATASET.rglob("*.txt") if p.stem in imgs])
```

ส่วนนี้ค้นหาไฟล์ Label ที่เป็น `.txt`

---

## `rglob("*.txt")`

```python
DATASET.rglob("*.txt")
```

หมายถึง:

> ค้นหาไฟล์ที่ลงท้ายด้วย `.txt` ใน Dataset และ Subfolder ทั้งหมด

---

## ตรวจสอบ Label ว่ามี Image คู่กัน

```python
if p.stem in imgs
```

ตัวอย่าง:

```text
001.txt
```

จะมี:

```python
p.stem
```

เป็น:

```text
001
```

จากนั้นตรวจว่า:

```python
"001" in imgs
```

หรือไม่

ถ้ามี `001.jpg` อยู่ใน `imgs` จะได้:

```text
True
```

ดังนั้น Label จะถูกเลือกเฉพาะกรณีที่มี Image คู่กัน

---

# 7. `sorted()`

```python
sorted(...)
```

ใช้เรียงลำดับ Label

ทำให้รายการ Label มีลำดับที่แน่นอนก่อนนำไปประมวลผล

---

# 8. แบ่ง Image ทีละ 6 รูป

```python
for n in range(0, len(labels), 6):
```

นี่คือ Loop หลักสำหรับแบ่งรูปออกเป็นกลุ่มละ 6 รูป

รูปแบบของ `range()` คือ:

```python
range(start, stop, step)
```

ในกรณีนี้:

```python
range(0, len(labels), 6)
```

หมายถึง:

```text
เริ่มที่ 0
เพิ่มทีละ 6
หยุดก่อนถึง len(labels)
```

ถ้ามี 20 Label:

```text
0
6
12
18
```

จะแบ่งเป็น:

```text
กลุ่มที่ 1 → 0 ถึง 5
กลุ่มที่ 2 → 6 ถึง 11
กลุ่มที่ 3 → 12 ถึง 17
กลุ่มที่ 4 → 18 ถึง 19
```

---

# 9. สร้าง Grid สำหรับแสดงรูป

```python
fig, ax = plt.subplots(2, 3, figsize=(15, 9))
```

สร้างพื้นที่สำหรับแสดงรูปจำนวน:

```text
2 rows × 3 columns
```

จึงสามารถแสดงได้:

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │ Image 6 │
└─────────┴─────────┴─────────┘
```

`fig` คือ Figure ทั้งหมด

`ax` คือพื้นที่ย่อยแต่ละช่อง

---

# 10. `ravel()`

```python
ax = ax.ravel()
```

ตอนแรก `ax` มีโครงสร้างแบบ 2 มิติ:

```text
[
    [ax1, ax2, ax3],
    [ax4, ax5, ax6]
]
```

หลังจาก:

```python
ax.ravel()
```

จะกลายเป็น Array 1 มิติ:

```text
[ax1, ax2, ax3, ax4, ax5, ax6]
```

จึงสามารถใช้:

```python
ax[i]
```

เพื่อเลือกช่องที่ต้องการได้ง่าย

---

# 11. วนทีละ Label

```python
for i, label in enumerate(labels[n:n+6]):
```

## `labels[n:n+6]`

ใช้ Slice รายการ Label ทีละ 6 ตัว

ตัวอย่างเมื่อ:

```python
n = 0
```

จะได้:

```python
labels[0:6]
```

คือ Label ลำดับที่ 0 ถึง 5

เมื่อ:

```python
n = 6
```

จะได้:

```python
labels[6:12]
```

---

## `enumerate()`

ทำให้ได้ทั้ง:

```text
i
label
```

ตัวอย่าง:

```text
i = 0 → 001.txt
i = 1 → 002.txt
i = 2 → 003.txt
```

`i` ใช้สำหรับเลือกช่องใน Grid:

```python
ax[i]
```

---

# 12. อ่าน Image

```python
img = cv2.imread(str(imgs[label.stem]))
```

เริ่มจาก:

```python
label.stem
```

เช่น:

```text
001
```

จากนั้น:

```python
imgs["001"]
```

จะหา Image ที่ชื่อ `001`

เช่น:

```text
001.jpg
```

จากนั้น:

```python
str(...)
```

แปลง Path เป็น String

แล้ว:

```python
cv2.imread(...)
```

ใช้ OpenCV อ่านรูปเข้า Python

รูปที่อ่านได้ถูกเก็บไว้ใน:

```python
img
```

---

# 13. หาขนาดของ Image

```python
h, w = img.shape[:2]
```

Image ของ OpenCV มี Shape ประมาณ:

```text
(height, width, channel)
```

ตัวอย่าง:

```text
(1080, 1920, 3)
```

ดังนั้น:

```python
img.shape[:2]
```

จะเอาเฉพาะ:

```text
(1080, 1920)
```

จึงได้:

```text
h = 1080
w = 1920
```

---

# 14. อ่านข้อมูลใน Label

```python
for line in label.read_text(encoding="utf-8-sig").splitlines():
```

อ่านไฟล์ `.txt` แล้ววนทีละบรรทัด

ตัวอย่าง Label แบบ YOLO:

```text
0 0.5 0.5 0.2 0.3
1 0.2 0.4 0.1 0.2
```

---

## `read_text()`

```python
label.read_text(encoding="utf-8-sig")
```

อ่านข้อความจากไฟล์ Label

ใช้:

```python
encoding="utf-8-sig"
```

เพื่อรองรับไฟล์ที่มี UTF-8 BOM

---

## `splitlines()`

```python
.splitlines()
```

แบ่งข้อความออกเป็นแต่ละบรรทัด

ตัวอย่าง:

```text
0 0.5 0.5 0.2 0.3
1 0.2 0.4 0.1 0.2
```

จะกลายเป็น:

```python
[
    "0 0.5 0.5 0.2 0.3",
    "1 0.2 0.4 0.1 0.2"
]
```

---

# 15. แยกข้อมูลในแต่ละบรรทัด

```python
d = line.split()
```

ตัวอย่าง:

```text
0 0.5 0.5 0.2 0.3
```

เมื่อใช้:

```python
line.split()
```

จะได้:

```python
["0", "0.5", "0.5", "0.2", "0.3"]
```

---

# 16. ตรวจสอบว่ามี 5 ค่า

```python
if len(d) == 5:
```

YOLO Detection Label มีรูปแบบ:

```text
class x_center y_center width height
```

ดังนั้นมีทั้งหมด 5 ค่า

ตัวอย่าง:

```text
0 0.5 0.5 0.2 0.3
│ │   │   │   │
│ │   │   │   └── height
│ │   │   └────── width
│ │   └────────── y_center
│ └────────────── x_center
└──────────────── class
```

---

# 17. ดึงค่า Coordinate ของ Bounding Box

```python
x,y,bw,bh = map(float, d[1:])
```

`d[1:]` หมายถึงเอาข้อมูลตั้งแต่ตำแหน่งที่ 1 เป็นต้นไป

จาก:

```python
["0", "0.5", "0.5", "0.2", "0.3"]
```

จะได้:

```python
["0.5", "0.5", "0.2", "0.3"]
```

จากนั้น:

```python
map(float, ...)
```

แปลง String เป็น Float

จึงได้:

```text
x  = 0.5
y  = 0.5
bw = 0.2
bh = 0.3
```

โดย:

```text
x  = x_center
y  = y_center
bw = box width
bh = box height
```

---

# 18. แปลง YOLO Coordinate เป็น Pixel — มุมซ้ายบน

```python
x1,y1 = int((x-bw/2)*w), int((y-bh/2)*h)
```

YOLO ใช้ Coordinate แบบ **Normalized**

ค่าจะอยู่ประมาณ:

```text
0 ถึง 1
```

แต่ OpenCV ต้องใช้ Coordinate แบบ Pixel

เช่น:

```text
x = 960
y = 540
```

สูตรสำหรับหามุมซ้ายบนคือ:

```text
x1 = (x_center - width/2) × image_width
y1 = (y_center - height/2) × image_height
```

ในโค้ด:

```python
x1 = int((x - bw/2) * w)
y1 = int((y - bh/2) * h)
```

---

# 19. แปลง YOLO Coordinate เป็น Pixel — มุมขวาล่าง

```python
x2,y2 = int((x+bw/2)*w), int((y+bh/2)*h)
```

สูตรคือ:

```text
x2 = (x_center + width/2) × image_width
y2 = (y_center + height/2) × image_height
```

จึงได้ Bounding Box:

```text
(x1,y1)
   ┌─────────────────┐
   │                 │
   │     Object      │
   │                 │
   └─────────────────┘
                  (x2,y2)
```

---

# 20. วาด Bounding Box

```python
cv2.rectangle(img,(x1,y1),(x2,y2),(0,255,0),2)
```

ใช้ OpenCV วาดสี่เหลี่ยมบน Image

รูปแบบ:

```python
cv2.rectangle(
    image,
    top_left,
    bottom_right,
    color,
    thickness
)
```

ดังนั้น:

```python
img
```

คือรูปที่จะวาด

```python
(x1,y1)
```

คือมุมซ้ายบน

```python
(x2,y2)
```

คือมุมขวาล่าง

```python
(0,255,0)
```

คือสีเขียวในระบบ BGR ของ OpenCV

```python
2
```

คือความหนาของเส้น 2 pixels

---

# 21. แปลงสี BGR → RGB

```python
cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
```

OpenCV อ่านรูปเป็น:

```text
BGR
```

แต่ Matplotlib ใช้:

```text
RGB
```

ดังนั้นต้องแปลงก่อนแสดงผล

```python
cv2.COLOR_BGR2RGB
```

---

# 22. แสดง Image ในช่องที่กำหนด

```python
ax[i].imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
```

`ax[i]` เลือกช่องที่ต้องการ

ตัวอย่าง:

```text
i = 0 → ช่องที่ 1
i = 1 → ช่องที่ 2
i = 2 → ช่องที่ 3
i = 3 → ช่องที่ 4
i = 4 → ช่องที่ 5
i = 5 → ช่องที่ 6
```

`imshow()` ใช้แสดงรูปภาพ

---

# 23. ซ่อน Axis

```python
ax[i].axis("off")
```

ซ่อนแกน X และ Y

ทำให้แสดงเฉพาะรูปภาพและ Bounding Box

---

# 24. ซ่อนช่องว่าง

```python
for a in ax[len(labels[n:n+6]):]:
    a.axis("off")
```

ใช้กรณีที่หน้าสุดท้ายมี Image ไม่ครบ 6 รูป

ตัวอย่างมีทั้งหมด 8 รูป:

```text
หน้าที่ 1

┌─────┬─────┬─────┐
│  1  │  2  │  3  │
├─────┼─────┼─────┤
│  4  │  5  │  6  │
└─────┴─────┴─────┘


หน้าที่ 2

┌─────┬─────┬─────┐
│  7  │  8  │     │
├─────┼─────┼─────┤
│     │     │     │
└─────┴─────┴─────┘
```

ช่องที่ไม่มี Image จะถูกซ่อนด้วย:

```python
a.axis("off")
```

---

# 25. จัด Layout

```python
plt.tight_layout()
```

จัดระยะห่างของ Image แต่ละช่องโดยอัตโนมัติ

เพื่อให้รูปไม่ชนกันและใช้พื้นที่ได้เหมาะสม

---

# 26. แสดงผล

```python
plt.show()
```

สั่งให้ Matplotlib แสดง Figure

---

# สรุปการทำงานของ WORK STEP 2

```text
Dataset
   │
   ▼
ค้นหา Image
(.jpg / .jpeg / .png)
   │
   ▼
สร้าง Dictionary
stem → Image Path
   │
   ▼
ค้นหา Label
(.txt)
   │
   ▼
ตรวจว่า Label มี Image คู่กัน
   │
   ▼
แบ่งทีละ 6 Image
   │
   ▼
สร้าง Grid 2 × 3
   │
   ▼
อ่าน Image
   │
   ▼
อ่าน YOLO Label
   │
   ▼
ตรวจข้อมูล 5 ค่า
   │
   ▼
ดึง x, y, width, height
   │
   ▼
YOLO Coordinate
(Normalized 0–1)
   │
   ▼
แปลงเป็น Pixel Coordinate
   │
   ▼
วาด Bounding Box
   │
   ▼
BGR → RGB
   │
   ▼
แสดงผล 6 รูปต่อหน้า
```

## เป้าหมายของ Step 2

**ตรวจสอบว่า YOLO Label สามารถจับคู่กับ Image และ Bounding Box ถูกวาดในตำแหน่งที่ถูกต้องหรือไม่**

ถ้ากรอบ Bounding Box ตรงกับ Object ในภาพ แสดงว่า Coordinate ของ Label สามารถนำมาใช้งานกับ Image ได้ถูกต้อง
