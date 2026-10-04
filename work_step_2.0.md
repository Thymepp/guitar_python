# 2.0 Visualize YOLO Labels

## 1. วัตถุประสงค์

โค้ดนี้ใช้สำหรับ **ตรวจสอบความถูกต้องของ YOLO Label ด้วยภาพ**

โดยจะ:

* ค้นหา Image
* ค้นหา Label `.txt`
* จับคู่ Image กับ Label
* อ่าน Bounding Box จาก YOLO Label
* แปลงพิกัดจาก YOLO Format เป็น Pixel
* วาด Bounding Box ลงบน Image
* แสดงภาพครั้งละ **6 รูป**

เหมาะสำหรับตรวจว่า **Label วาดถูกตำแหน่งหรือไม่**

---

# 2. Import Library

```python
from pathlib import Path
import cv2
import matplotlib.pyplot as plt
```

### `Path`

ใช้จัดการ File และ Folder

### `cv2`

ใช้เปิด Image และวาด Bounding Box

### `matplotlib.pyplot`

ใช้แสดง Image เป็น Grid

---

# 3. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset

---

# 4. หา Image

```python
imgs = {
    p.stem: p
    for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
}
```

สร้าง Dictionary สำหรับเก็บ Image

ตัวอย่าง:

```python
{
    "P711_001": Path("P711_001.jpg"),
    "P711_002": Path("P711_002.jpg")
}
```

โครงสร้างคือ:

```text
Key   = p.stem
Value = p
```

---

# 5. หา Label

```python
labels = sorted([
    p for p in DATASET.rglob("*.txt")
    if p.stem in imgs
])
```

ค้นหาไฟล์ `.txt`

แต่เอาเฉพาะ Label ที่มี Image ชื่อเดียวกัน

ตัวอย่าง:

```text
P711_001.jpg
P711_001.txt
```

ถือว่าเป็นคู่กัน

---

# 6. แสดงทีละ 6 Images

```python
for n in range(0, len(labels), 6):
```

กำหนดให้แสดง Label ครั้งละ 6 ไฟล์

ถ้ามี 18 Labels:

```text
รอบที่ 1 → 0-5
รอบที่ 2 → 6-11
รอบที่ 3 → 12-17
```

---

# 7. สร้าง Grid 2 × 3

```python
fig, ax = plt.subplots(
    2, 3,
    figsize=(15, 9)
)
```

สร้างพื้นที่แสดงภาพ:

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │ Image 6 │
└─────────┴─────────┴─────────┘
```

รวมทั้งหมด:

```text
2 × 3 = 6 Images
```

---

# 8. เปลี่ยน Array ให้เป็น 1 มิติ

```python
ax = ax.ravel()
```

ก่อน:

```text
[
 [ax1, ax2, ax3],
 [ax4, ax5, ax6]
]
```

หลัง:

```text
[ax1, ax2, ax3, ax4, ax5, ax6]
```

ทำให้สามารถใช้:

```python
ax[i]
```

ได้ง่ายขึ้น

---

# 9. วน Label 6 ไฟล์

```python
for i, label in enumerate(labels[n:n+6]):
```

`labels[n:n+6]` คือการตัด List มาไม่เกิน 6 ตัว

ตัวอย่าง:

```text
labels[0:6]
```

ได้ Label ตัวที่:

```text
0 1 2 3 4 5
```

---

# 10. เปิด Image

```python
img = cv2.imread(
    str(imgs[label.stem])
)
```

ใช้ชื่อ Label:

```python
label.stem
```

เพื่อหา Image ที่ชื่อเดียวกันจาก Dictionary `imgs`

เช่น:

```text
Label:
P711_001.txt

label.stem:
P711_001
```

จึงไปหา:

```text
imgs["P711_001"]
```

---

# 11. อ่านขนาด Image

```python
h, w = img.shape[:2]
```

ได้:

```text
h = Height
w = Width
```

ตัวอย่าง:

```text
Image = 1920 × 1080

w = 1920
h = 1080
```

---

# 12. อ่าน YOLO Label

```python
for line in label.read_text(
    encoding="utf-8-sig"
).splitlines():
```

อ่าน Label ทีละบรรทัด

ตัวอย่าง YOLO Label:

```text
0 0.512 0.431 0.120 0.220
```

รูปแบบคือ:

```text
class_id
x_center
y_center
width
height
```

---

# 13. แยกข้อมูลแต่ละบรรทัด

```python
d = line.split()
```

ตัวอย่าง:

```text
0 0.512 0.431 0.120 0.220
```

จะกลายเป็น:

```python
[
    "0",
    "0.512",
    "0.431",
    "0.120",
    "0.220"
]
```

---

# 14. ตรวจว่ามี 5 ค่า

```python
if len(d) == 5:
```

YOLO Bounding Box ต้องมี 5 ค่า:

```text
class_id
x
y
width
height
```

---

# 15. อ่าน Bounding Box

```python
x, y, bw, bh = map(float, d[1:])
```

ตัด Class ID ออก:

```python
d[1:]
```

เหลือ:

```text
x
y
bw
bh
```

โดย:

```text
x  = Center X
y  = Center Y
bw = Box Width
bh = Box Height
```

ค่าทั้งหมดของ YOLO เป็น **Normalized Coordinates**

อยู่ประมาณ:

```text
0.0 → 1.0
```

---

# 16. แปลง YOLO เป็น Pixel

YOLO ใช้:

```text
Center X
Center Y
Width
Height
```

แต่ OpenCV ต้องการ:

```text
x1, y1
x2, y2
```

ดังนั้นคำนวณ:

```python
x1 = int((x - bw / 2) * w)
y1 = int((y - bh / 2) * h)

x2 = int((x + bw / 2) * w)
y2 = int((y + bh / 2) * h)
```

---

# 17. ทำไมต้อง `bw / 2`

YOLO เก็บ Bounding Box จากจุดกึ่งกลาง:

```text
          x
          ↓
     ┌─────────┐
     │    ●    │
     │         │
     └─────────┘
```

ดังนั้น:

```text
Left   = Center X - Width / 2
Right  = Center X + Width / 2

Top    = Center Y - Height / 2
Bottom = Center Y + Height / 2
```

---

# 18. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (x1, y1),
    (x2, y2),
    (0, 255, 0),
    2
)
```

ความหมาย:

```text
img       = ภาพ
(x1, y1)  = มุมซ้ายบน
(x2, y2)  = มุมขวาล่าง
(0,255,0) = สีเขียวใน BGR
2         = ความหนาเส้น
```

---

# 19. แปลง BGR → RGB

OpenCV อ่านภาพเป็น:

```text
BGR
```

แต่ Matplotlib ใช้:

```text
RGB
```

จึงต้องใช้:

```python
cv2.cvtColor(
    img,
    cv2.COLOR_BGR2RGB
)
```

แล้วจึง:

```python
ax[i].imshow(...)
```

---

# 20. ปิด Axis

```python
ax[i].axis("off")
```

ซ่อน:

```text
X-axis
Y-axis
```

เพื่อให้แสดงเฉพาะภาพ

---

# 21. ปิดช่องที่ไม่มี Image

กรณีรอบสุดท้ายมีไม่ครบ 6 รูป

เช่นมีเหลือเพียง 2 รูป:

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │         │
├─────────┼─────────┼─────────┤
│         │         │         │
└─────────┴─────────┴─────────┘
```

โค้ด:

```python
for a in ax[len(labels[n:n+6]):]:
    a.axis("off")
```

จะซ่อนช่องที่ไม่มีข้อมูล

---

# 22. จัด Layout

```python
plt.tight_layout()
```

ช่วยจัดระยะห่างระหว่างภาพให้เหมาะสม

---

# 23. แสดงผล

```python
plt.show()
```

แสดง Grid ของภาพพร้อม Bounding Box

---

# 24. Flow การทำงาน

```text
Dataset
   │
   ├── ค้นหา Images
   │      └── .jpg / .jpeg / .png
   │
   ├── สร้าง Dictionary
   │      └── stem → image path
   │
   ├── ค้นหา Labels
   │      └── .txt ที่มี Image คู่กัน
   │
   ├── แบ่งทีละ 6
   │
   ├── อ่าน Image
   │
   ├── อ่าน YOLO Label
   │
   ├── YOLO Coordinate
   │      │
   │      ▼
   │   Pixel Coordinate
   │
   ├── วาด Bounding Box
   │
   └── แสดงผล 2 × 3
```

---

# 25. สูตรสำคัญ

YOLO:

```text
x_center
y_center
box_width
box_height
```

แปลงเป็น Pixel:

```text
x1 = (x_center - box_width / 2) × image_width

y1 = (y_center - box_height / 2) × image_height

x2 = (x_center + box_width / 2) × image_width

y2 = (y_center + box_height / 2) × image_height
```

---

# 26. สรุป

โค้ดนี้คือ **Label Visualization / Bounding Box Viewer**

ใช้สำหรับตรวจสอบว่า Dataset YOLO ถูกต้องหรือไม่

```text
YOLO Label
    ↓
อ่าน x, y, width, height
    ↓
แปลง Normalized → Pixel
    ↓
cv2.rectangle()
    ↓
Bounding Box
    ↓
Matplotlib
    ↓
แสดง 6 รูปต่อหน้า
```

### จุดประสงค์หลัก

> **ดูด้วยตาเปล่าว่า Bounding Box ใน Label ตรงกับ Object ใน Image หรือไม่**

ถ้า Box ครอบ Object ถูกต้อง → Label มีแนวโน้มถูกต้อง

ถ้า Box เบี้ยว / เลื่อน / ขนาดผิด → ต้องตรวจแก้ Label
