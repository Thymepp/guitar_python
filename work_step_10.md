# STEP 10 — MODEL TEST

## 1. จุดประสงค์ของ Step นี้

Step 10 คือการนำ **Final Model** ที่ได้จาก Step 9 มาทดสอบกับรูปภาพจริง

สิ่งที่ Step นี้ทำ:

1. หา `best.pt` ของ Final Model
2. โหลดโมเดล
3. ค้นหารูปภาพสำหรับทดสอบ
4. ไม่ใช้รูปจาก `P711_Final`
5. สุ่มรูปมาทดสอบสูงสุด 5 รูป
6. ให้ YOLO Predict
7. นับจำนวน Bounding Box ที่ตรวจพบ
8. วาดกรอบสีเขียวลงบนภาพ
9. แสดงผลเป็น Grid 2×3
10. พิมพ์จำนวน Object ของแต่ละรูปใน Terminal

Flow:

```text
Final Model
    │
    ▼
best.pt
    │
    ▼
โหลด YOLO
    │
    ▼
ค้นหารูป Test
    │
    ├── ไม่เอา P711_Final
    │
    ▼
สุ่ม 5 รูป
    │
    ▼
Predict
    │
    ├── Bounding Box
    └── Object Count
    │
    ▼
แสดงภาพ + กรอบสีเขียว
```

---

# 2. โค้ดทั้งหมด

```python
from pathlib import Path
from ultralytics import YOLO
import random
import cv2
import matplotlib.pyplot as plt

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

MODEL = (
    DATASET.parent
    / "runs" / "detect"
    / f"{DATASET.name}_Final"
    / "weights" / "best.pt"
)

model = YOLO(str(MODEL))

# รูปสำหรับทดสอบ
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and DATASET / "P711_Final" not in p.parents
]

# สุ่ม 5 รูป
samples = random.sample(images, min(5, len(images)))

fig, ax = plt.subplots(2, 3, figsize=(16, 10))
ax = ax.ravel()

for i, img_path in enumerate(samples):

    result = model.predict(
        str(img_path),
        conf=0.5,
        imgsz=960,
        device=0,
        verbose=False
    )[0]

    count = len(result.boxes)

    # อ่านภาพต้นฉบับ
    img = cv2.imread(str(img_path))

    h, w = img.shape[:2]

    # วาดเฉพาะกรอบสีเขียว
    for box in result.boxes:

        x1, y1, x2, y2 = box.xyxy[0].tolist()

        cv2.rectangle(
            img,
            (int(x1), int(y1)),
            (int(x2), int(y2)),
            (0, 255, 0),
            2
        )

    ax[i].imshow(
        cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    )

    ax[i].set_title(
        f"Count: {count}",
        fontsize=14,
        fontweight="bold"
    )

    ax[i].axis("off")

# ช่องที่เหลือ
for a in ax[len(samples):]:
    a.axis("off")

plt.tight_layout()
plt.show()

print("\n=================================")
print("STEP 10 — MODEL TEST")
print("=================================")

for img_path in samples:

    result = model.predict(
        str(img_path),
        conf=0.5,
        imgsz=960,
        device=0,
        verbose=False
    )[0]

    print(f"{img_path.name} : {len(result.boxes)} Objects")

print("=================================")
```

---

# 3. Import Library

```python
from pathlib import Path
from ultralytics import YOLO
import random
import cv2
import matplotlib.pyplot as plt
```

ใช้ทั้งหมด 5 ส่วนหลัก

| Library      | หน้าที่                    |
| ------------ | -------------------------- |
| `Path`       | จัดการ Path ของไฟล์/Folder |
| `YOLO`       | โหลดและ Predict ด้วยโมเดล  |
| `random`     | สุ่มภาพ                    |
| `cv2`        | อ่านภาพและวาด Bounding Box |
| `matplotlib` | แสดงภาพ                    |

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนด Folder หลัก:

```text
C:\Users\pawor\Desktop\P711Test
```

ตัวแปรนี้ถูกใช้เป็นจุดเริ่มต้นในการค้นหา Dataset และ Model

---

# 5. หา Final Model

```python
MODEL = (
    DATASET.parent
    / "runs" / "detect"
    / f"{DATASET.name}_Final"
    / "weights" / "best.pt"
)
```

ส่วนนี้ใช้สร้าง Path ไปยัง:

```text
best.pt
```

จากโครงสร้าง:

```text
Desktop/
│
├── P711Test/
│
└── runs/
    └── detect/
        └── P711Test_Final/
            └── weights/
                └── best.pt
```

---

# 6. ทำความเข้าใจ `DATASET.parent`

ถ้า:

```python
DATASET
```

คือ:

```text
C:\Users\pawor\Desktop\P711Test
```

คำสั่ง:

```python
DATASET.parent
```

จะได้:

```text
C:\Users\pawor\Desktop
```

จึงสามารถต่อไปยัง:

```text
runs/detect/P711Test_Final/weights/best.pt
```

ได้

---

# 7. ทำความเข้าใจ `DATASET.name`

```python
DATASET.name
```

จะได้:

```text
P711Test
```

ดังนั้น:

```python
f"{DATASET.name}_Final"
```

จะกลายเป็น:

```text
P711Test_Final
```

ตรงกับชื่อ Run ที่สร้างจาก Step 9

---

# 8. โหลดโมเดล

```python
model = YOLO(str(MODEL))
```

นำ Path ของ:

```text
best.pt
```

ไปโหลดด้วย YOLO

ตัวอย่าง:

```text
best.pt
   ↓
YOLO()
   ↓
model
```

จากนั้นตัวแปร `model` สามารถใช้:

```python
model.predict(...)
```

เพื่อทดสอบภาพได้

---

# 9. ค้นหารูปภาพสำหรับ Test

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and DATASET / "P711_Final" not in p.parents
]
```

ส่วนนี้ใช้ค้นหารูปทั้งหมดภายใต้:

```text
P711Test
```

ด้วย:

```python
DATASET.rglob("*")
```

---

# 10. `rglob("*")`

```python
DATASET.rglob("*")
```

หมายถึงค้นหาไฟล์และ Folder แบบ Recursive

ตัวอย่าง:

```text
P711Test/
├── image1.jpg
├── folderA/
│   ├── image2.jpg
│   └── image3.jpg
└── P711_Final/
    └── images/
        └── image4.jpg
```

`rglob("*")` สามารถเดินเข้าไปค้นหาทุกระดับ

```text
ระดับ 1
   ↓
ระดับ 2
   ↓
ระดับ 3
   ↓
...
```

---

# 11. ตรวจสอบนามสกุลรูป

```python
p.suffix.lower() in [".jpg", ".jpeg", ".png"]
```

รับเฉพาะ:

```text
.jpg
.jpeg
.png
```

การใช้:

```python
.lower()
```

ทำให้รองรับทั้ง:

```text
.JPG
.jpg
.JPEG
.jpeg
.PNG
.png
```

---

# 12. ไม่เอารูปจาก `P711_Final`

```python
and DATASET / "P711_Final" not in p.parents
```

จุดนี้สำคัญมาก

เพราะ Step 10 ต้องการทดสอบกับรูปที่ไม่ได้อยู่ใน:

```text
P711_Final
```

จึงตัด Folder นี้ออก

ตัวอย่าง:

```text
P711Test/
│
├── test01.jpg          ← เอา
├── test02.jpg          ← เอา
│
└── P711_Final/
    └── images/
        ├── final01.jpg ← ไม่เอา
        └── final02.jpg ← ไม่เอา
```

---

# 13. `p.parents` คืออะไร

ถ้า:

```text
p =
C:\Users\pawor\Desktop\P711Test\P711_Final\images\abc.jpg
```

`p.parents` จะประกอบด้วย Parent Folder หลายระดับ เช่น:

```text
P711_Final/images
P711_Final
P711Test
Desktop
...
```

ดังนั้น:

```python
DATASET / "P711_Final" not in p.parents
```

สามารถตรวจได้ว่ารูปนั้นอยู่ภายใต้ `P711_Final` หรือไม่

---

# 14. สุ่มรูป 5 รูป

```python
samples = random.sample(images, min(5, len(images)))
```

ใช้:

```python
random.sample()
```

เพื่อสุ่มรูปจาก `images`

ต้องการสูงสุด:

```text
5 รูป
```

---

# 15. ทำไมใช้ `min(5, len(images))`

ถ้ามีรูปมากกว่า 5 รูป:

```text
len(images) = 100
```

จะสุ่ม:

```text
5 รูป
```

แต่ถ้ามีเพียง 3 รูป:

```text
len(images) = 3
```

จะสุ่ม:

```text
3 รูป
```

ดังนั้นไม่เกิดปัญหาจากการพยายามสุ่ม 5 รูป ทั้งที่มีรูปไม่ถึง 5 รูป

ตัวอย่าง:

```text
มี 100 รูป → สุ่ม 5
มี 10 รูป  → สุ่ม 5
มี 5 รูป   → สุ่ม 5
มี 3 รูป   → สุ่ม 3
มี 1 รูป   → สุ่ม 1
มี 0 รูป   → สุ่ม 0
```

---

# 16. สร้างพื้นที่แสดงผล

```python
fig, ax = plt.subplots(2, 3, figsize=(16, 10))
```

สร้าง Grid:

```text
┌──────────┬──────────┬──────────┐
│ Image 1  │ Image 2  │ Image 3  │
├──────────┼──────────┼──────────┤
│ Image 4  │ Image 5  │          │
└──────────┴──────────┴──────────┘
```

มีทั้งหมด:

```text
2 × 3 = 6 ช่อง
```

แต่ Step นี้สุ่มเพียง 5 รูป

ดังนั้นจะเหลือ:

```text
1 ช่องว่าง
```

---

# 17. `ax.ravel()`

```python
ax = ax.ravel()
```

ก่อนหน้านี้ `ax` มีรูปแบบ 2×3:

```text
[
 [ax0, ax1, ax2],
 [ax3, ax4, ax5]
]
```

หลังใช้:

```python
ax.ravel()
```

จะกลายเป็น Array 1 มิติ:

```text
[ax0, ax1, ax2, ax3, ax4, ax5]
```

จึงสามารถใช้:

```python
ax[i]
```

ได้ง่าย

---

# 18. วนทดสอบภาพ

```python
for i, img_path in enumerate(samples):
```

วนผ่านภาพที่สุ่มมา

ตัวอย่าง:

```text
samples
│
├── image01.jpg
├── image02.jpg
├── image03.jpg
├── image04.jpg
└── image05.jpg
```

จะได้:

```text
i = 0 → image01.jpg
i = 1 → image02.jpg
i = 2 → image03.jpg
i = 3 → image04.jpg
i = 4 → image05.jpg
```

---

# 19. Predict ด้วย Final Model

```python
result = model.predict(
    str(img_path),
    conf=0.5,
    imgsz=960,
    device=0,
    verbose=False
)[0]
```

ส่งภาพเข้า Final Model

---

# 20. `conf=0.5`

```python
conf=0.5
```

กำหนด Confidence Threshold เป็น:

```text
50%
```

แนวคิด:

```text
Prediction
     │
     ├── Confidence < 0.50
     │       ↓
     │      ไม่ใช้
     │
     └── Confidence ≥ 0.50
             ↓
           ใช้
```

ดังนั้น Bounding Box ที่มี Confidence ต่ำกว่า Threshold จะไม่ถูกนำมาใช้ในผลลัพธ์นี้

---

# 21. `imgsz=960`

```python
imgsz=960
```

กำหนดขนาด Input สำหรับ Predict เป็น:

```text
960 × 960
```

ให้สอดคล้องกับขนาดที่ใช้ในการ Train Final Model ใน Step 9

---

# 22. `device=0`

```python
device=0
```

กำหนดให้ Predict ด้วย GPU หมายเลข 0

```text
GPU 0
```

---

# 23. `[0]`

```python
)[0]
```

`model.predict()` สามารถคืนผลลัพธ์หลายภาพได้

แต่ในแต่ละรอบนี้ส่งภาพเข้าไปทีละ 1 ภาพ

จึงเลือกผลลัพธ์ของภาพแรกด้วย:

```python
[0]
```

---

# 24. นับจำนวน Object

```python
count = len(result.boxes)
```

`result.boxes` คือ Bounding Boxes ที่ Model ตรวจพบ

เช่น:

```text
result.boxes
│
├── Box 1
├── Box 2
├── Box 3
└── Box 4
```

ดังนั้น:

```python
len(result.boxes)
```

จะได้:

```text
4
```

จึงหมายความว่า Model ตรวจพบ:

```text
4 Objects
```

---

# 25. อ่านภาพต้นฉบับ

```python
img = cv2.imread(str(img_path))
```

OpenCV อ่านภาพเข้ามาเป็น Array

```text
Image File
    ↓
cv2.imread()
    ↓
NumPy Array
```

---

# 26. หา Width และ Height

```python
h, w = img.shape[:2]
```

ถ้าภาพมีขนาด:

```text
1920 × 1080
```

โดยทั่วไป:

```text
h = 1080
w = 1920
```

แต่ในโค้ดนี้ `h, w` ไม่ได้ถูกนำไปใช้ต่อโดยตรงในการวาดกรอบ เพราะ `box.xyxy` ให้พิกัดในหน่วย Pixel อยู่แล้ว

ดังนั้นบรรทัดนี้เป็นการอ่านขนาดภาพไว้ แต่ **ไม่จำเป็นต่อการคำนวณ Bounding Box ในโค้ดชุดนี้**

---

# 27. วนผ่าน Bounding Box

```python
for box in result.boxes:
```

ถ้า Model ตรวจพบ 3 Object:

```text
result.boxes
│
├── box 1
├── box 2
└── box 3
```

Loop จะทำงาน 3 รอบ

---

# 28. อ่านพิกัด Bounding Box

```python
x1, y1, x2, y2 = box.xyxy[0].tolist()
```

`xyxy` คือรูปแบบพิกัด:

```text
x1 = ซ้าย
y1 = บน
x2 = ขวา
y2 = ล่าง
```

ตัวอย่าง:

```text
x1 = 100
y1 = 200
x2 = 500
y2 = 600
```

หมายถึง:

```text
        x1                 x2
        ↓                  ↓
        ┌──────────────────┐
 y1 →   │                  │
        │      Object      │
        │                  │
        └──────────────────┘
 y2 → 
```

---

# 29. `tolist()`

```python
box.xyxy[0].tolist()
```

แปลง Tensor เป็น Python List

เช่น:

```text
Tensor
 ↓
[100.5, 200.2, 500.7, 600.8]
```

จากนั้นจึงนำไปเก็บ:

```python
x1, y1, x2, y2
```

---

# 30. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (int(x1), int(y1)),
    (int(x2), int(y2)),
    (0, 255, 0),
    2
)
```

ใช้ OpenCV วาดสี่เหลี่ยมรอบ Object

รูปแบบ:

```text
cv2.rectangle(
    image,
    จุดซ้ายบน,
    จุดขวาล่าง,
    สี,
    ความหนา
)
```

---

# 31. แปลงพิกัดเป็น Integer

```python
(int(x1), int(y1))
```

และ:

```python
(int(x2), int(y2))
```

เพราะ OpenCV ต้องการพิกัด Pixel เป็นจำนวนเต็ม

ตัวอย่าง:

```text
100.72 → 100
200.91 → 200
```

---

# 32. สี `(0, 255, 0)`

```python
(0, 255, 0)
```

OpenCV ใช้รูปแบบ:

```text
BGR
```

ดังนั้น:

```text
B = 0
G = 255
R = 0
```

จึงเป็น:

```text
สีเขียว
```

---

# 33. ความหนา `2`

```python
2
```

คือความหนาของเส้น Bounding Box:

```text
Thickness = 2 pixels
```

---

# 34. แสดงภาพด้วย Matplotlib

```python
ax[i].imshow(
    cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
)
```

OpenCV อ่านภาพเป็น:

```text
BGR
```

แต่ Matplotlib ต้องการ:

```text
RGB
```

จึงต้องแปลงด้วย:

```python
cv2.cvtColor(
    img,
    cv2.COLOR_BGR2RGB
)
```

---

# 35. แสดงจำนวน Object บนภาพ

```python
ax[i].set_title(
    f"Count: {count}",
    fontsize=14,
    fontweight="bold"
)
```

ถ้า Model ตรวจพบ 6 Object:

```text
Count: 6
```

จะถูกแสดงด้านบนของภาพ

---

# 36. `fontsize=14`

```python
fontsize=14
```

กำหนดขนาดตัวอักษรของ Title

---

# 37. `fontweight="bold"`

```python
fontweight="bold"
```

ทำให้ข้อความ:

```text
Count: 6
```

เป็นตัวหนา

---

# 38. ซ่อนแกน

```python
ax[i].axis("off")
```

ไม่แสดง:

```text
X Axis
Y Axis
ตัวเลข
Tick
```

ทำให้เห็นเฉพาะภาพ

---

# 39. จัดการช่องที่เหลือ

```python
for a in ax[len(samples):]:
    a.axis("off")
```

เนื่องจาก Grid มี 6 ช่อง แต่สุ่มเพียง 5 รูป

จึงมี 1 ช่องเหลือ

```text
┌───────┬───────┬───────┐
│ Img 1 │ Img 2 │ Img 3 │
├───────┼───────┼───────┤
│ Img 4 │ Img 5 │ EMPTY │
└───────┴───────┴───────┘
```

โค้ดนี้จะซ่อนช่อง `EMPTY`

---

# 40. จัด Layout

```python
plt.tight_layout()
```

ปรับระยะห่างของแต่ละภาพให้เหมาะสม

---

# 41. แสดงผล

```python
plt.show()
```

เปิดหน้าต่างแสดงผล:

```text
┌────────────┬────────────┬────────────┐
│   Image 1  │   Image 2  │   Image 3  │
│            │            │            │
│  ┌──────┐  │ ┌────────┐ │   ┌─────┐  │
│  │Object│  │ │ Object │ │   │ Obj │  │
│  └──────┘  │ └────────┘ │   └─────┘  │
│ Count: 2   │ Count: 4   │ Count: 1   │
├────────────┼────────────┼────────────┤
│   Image 4  │   Image 5  │            │
│  Count: 3  │  Count: 5  │            │
└────────────┴────────────┴────────────┘
```

---

# 42. แสดง Header ใน Terminal

```python
print("\n=================================")
print("STEP 10 — MODEL TEST")
print("=================================")
```

แสดงว่าเริ่มส่วน Summary ของ Step 10

ตัวอย่าง:

```text
=================================
STEP 10 — MODEL TEST
=================================
```

---

# 43. Predict ซ้ำเพื่อสรุปผล

```python
for img_path in samples:

    result = model.predict(
        str(img_path),
        conf=0.5,
        imgsz=960,
        device=0,
        verbose=False
    )[0]

    print(f"{img_path.name} : {len(result.boxes)} Objects")
```

ส่วนนี้ Predict ภาพเดิมอีกครั้ง

จุดประสงค์คือเอาผลลัพธ์มา Print ใน Terminal

ตัวอย่าง:

```text
image001.jpg : 4 Objects
image002.jpg : 7 Objects
image003.jpg : 3 Objects
image004.jpg : 5 Objects
image005.jpg : 6 Objects
```

---

# 44. ทำไม Predict สองครั้ง?

ในโค้ดนี้มี:

### ครั้งที่ 1

ใช้สำหรับ:

```text
Predict
↓
วาด Bounding Box
↓
แสดงภาพ
```

### ครั้งที่ 2

ใช้สำหรับ:

```text
Predict
↓
นับ Object
↓
Print Terminal
```

ดังนั้นผลลัพธ์เชิงตรรกะเหมือนกัน แต่ Model ถูกเรียก Predict สองรอบ

ถ้าต้องการลดเวลาในการทำงาน สามารถเก็บ `result` จากรอบแรกไว้แล้วนำมาใช้ Print ได้ โดยไม่ต้อง Predict ซ้ำ

---

# 45. ตัวอย่างผลลัพธ์

สมมติสุ่มได้:

```text
A001.jpg
A017.jpg
A032.jpg
A054.jpg
A088.jpg
```

และ Model ตรวจพบ:

```text
A001.jpg → 4 Objects
A017.jpg → 2 Objects
A032.jpg → 6 Objects
A054.jpg → 3 Objects
A088.jpg → 5 Objects
```

บนภาพจะเห็น:

```text
Count: 4
Count: 2
Count: 6
Count: 3
Count: 5
```

และ Terminal:

```text
=================================
STEP 10 — MODEL TEST
=================================
A001.jpg : 4 Objects
A017.jpg : 2 Objects
A032.jpg : 6 Objects
A054.jpg : 3 Objects
A088.jpg : 5 Objects
=================================
```

---

# 46. จุดสำคัญของ Step 10

Step นี้เป็นการ **ทดสอบการทำงานของโมเดลแบบ Visual Test**

ไม่ได้ตรวจเพียงว่า Model Predict ได้หรือไม่ แต่ยังดู:

```text
1. Bounding Box อยู่ตรง Object หรือไม่
2. มี Object ที่ตรวจไม่พบหรือไม่
3. มีกรอบที่เกิดขึ้นผิดตำแหน่งหรือไม่
4. จำนวน Object สมเหตุสมผลหรือไม่
5. Confidence Threshold = 0.5 เหมาะสมหรือไม่
```

---

# 47. ความสัมพันธ์กับ Step 9

Step 9:

```text
P711_Final
     ↓
Train
     ↓
best.pt
```

Step 10:

```text
best.pt
     ↓
Load Model
     ↓
สุ่มภาพ Test
     ↓
Predict
     ↓
Bounding Boxes
     ↓
Object Count
```

ดังนั้น:

```text
STEP 9 = สร้าง Final Model
STEP 10 = ทดสอบ Final Model
```

---

# 48. Pipeline ทั้งหมด

```text
STEP 4
สร้าง Seed data.yaml
        ↓
STEP 5
Train Seed Model
        ↓
STEP 6
Auto Label
        ↓
STEP 7
ตรวจสอบ Auto Label
        ↓
STEP 8
Build P711_Final
        ↓
STEP 9
Train Final Model
        ↓
best.pt
        ↓
STEP 10
Model Test
        ↓
สุ่ม 5 รูป
        ↓
Predict
        ↓
┌───────────────────┐
│ Bounding Box      │
│ Object Count      │
│ Visual Inspection │
└───────────────────┘
```

---

# 49. สรุป Step 10

| ส่วน                          | หน้าที่                      |
| ----------------------------- | ---------------------------- |
| `DATASET`                     | กำหนด Dataset หลัก           |
| `MODEL`                       | หา `best.pt` ของ Final Model |
| `YOLO()`                      | โหลด Final Model             |
| `rglob("*")`                  | ค้นหารูปแบบ Recursive        |
| `P711_Final not in p.parents` | ไม่เอารูปจาก Final Dataset   |
| `random.sample()`             | สุ่มภาพ                      |
| `min(5, len(images))`         | จำกัดสูงสุด 5 รูป            |
| `model.predict()`             | Predict ภาพ                  |
| `conf=0.5`                    | Confidence Threshold         |
| `imgsz=960`                   | Input Size                   |
| `device=0`                    | ใช้ GPU 0                    |
| `result.boxes`                | Bounding Boxes               |
| `len(result.boxes)`           | จำนวน Object                 |
| `cv2.rectangle()`             | วาดกรอบ                      |
| `plt.subplots(2,3)`           | สร้าง Grid 2×3               |
| `plt.show()`                  | แสดงภาพ                      |
| `print()`                     | แสดงจำนวน Object ใน Terminal |

**ผลลัพธ์สุดท้ายของ Step 10 คือการเห็นภาพจริงว่า Final Model ตรวจจับ Object ได้อย่างไร พร้อมจำนวน Object ที่ตรวจพบในแต่ละภาพ**
