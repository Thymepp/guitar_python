# STEP 10.0 — MODEL TEST

## 1. หน้าที่ของ STEP 10.0

STEP 10.0 ใช้สำหรับ **ทดสอบ Final Model กับรูปภาพจริง** หลังจาก Train เสร็จใน STEP 9.0

โดยจะ

1. โหลด `best.pt`
2. ค้นหารูปภาพสำหรับทดสอบ
3. สุ่ม 5 รูป
4. ให้ Model Predict
5. วาด Bounding Box
6. แสดงจำนวน Object ที่ตรวจพบ
7. แสดงผลจำนวน Object ใน Terminal

ภาพรวม

```text
STEP 9.0
Train Final Model
       ↓
P711Test_Final
       ↓
weights/best.pt
       ↓
STEP 10.0
Model Test
       ↓
สุ่มรูป 5 รูป
       ↓
YOLO Predict
       ↓
Bounding Box
       ↓
Count Objects
```

---

# 2. Import Library

```python
from pathlib import Path
from ultralytics import YOLO
import random
import cv2
import matplotlib.pyplot as plt
```

### `Path`

ใช้จัดการ Path ของ Dataset และ Model

### `YOLO`

ใช้โหลด Final Model และทำ Prediction

### `random`

ใช้สุ่มรูปภาพ

```python
random.sample()
```

### `cv2`

ใช้

* อ่าน Image
* วาด Bounding Box
* แปลง BGR → RGB

### `matplotlib`

ใช้แสดงผล Image หลายรูปพร้อมกัน

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

# 4. กำหนดตำแหน่ง Final Model

```python
MODEL = (
    DATASET.parent
    / "runs" / "detect"
    / f"{DATASET.name}_Final"
    / "weights" / "best.pt"
)
```

ส่วนนี้สร้าง Path ไปยัง

```text
runs/detect/P711Test_Final/weights/best.pt
```

เพราะ

```python
DATASET.parent
```

คือ

```text
C:\Users\pawor\Desktop
```

และ

```python
DATASET.name
```

คือ

```text
P711Test
```

ดังนั้น

```python
f"{DATASET.name}_Final"
```

จะได้

```text
P711Test_Final
```

สุดท้ายคือ

```text
C:\Users\pawor\Desktop\runs\detect\P711Test_Final\weights\best.pt
```

---

# 5. โหลด Final Model

```python
model = YOLO(str(MODEL))
```

นำ

```text
best.pt
```

เข้า YOLO

จากนั้นตัวแปร

```python
model
```

จะเป็น Final Model ที่พร้อม Prediction

---

# 6. ค้นหารูปสำหรับทดสอบ

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and DATASET / "P711_Final" not in p.parents
]
```

ค้นหารูปทั้งหมดภายใน Dataset

รองรับ

```text
.jpg
.jpeg
.png
```

---

# 7. ทำไมต้องข้าม `P711_Final`?

```python
and DATASET / "P711_Final" not in p.parents
```

เพราะเราไม่ต้องการเอารูปใน

```text
P711_Final/images
```

มาทดสอบ

เนื่องจาก Final Dataset เป็นข้อมูลที่ใช้ในการ Train Model

ถ้านำรูปจาก Training Dataset มาทดสอบ อาจทำให้ผลดูดีเกินจริง

ดังนั้นโค้ดพยายามเลือก Image ที่อยู่นอก

```text
P711_Final/
```

---

# 8. สุ่ม 5 รูป

```python
samples = random.sample(
    images,
    min(5, len(images))
)
```

เลือก Image แบบสุ่มจำนวนสูงสุด 5 รูป

ถ้ามี 100 รูป

```text
100 Images
     ↓
สุ่ม
     ↓
5 Images
```

ถ้ามีเพียง 3 รูป

```text
3 Images
     ↓
min(5, 3)
     ↓
3 Images
```

จึงไม่เกิด Error จากการขอสุ่ม 5 รูปทั้งที่มีรูปไม่ถึง 5 รูป

---

# 9. สร้างพื้นที่แสดงผล

```python
fig, ax = plt.subplots(
    2,
    3,
    figsize=(16, 10)
)
```

สร้าง Grid ขนาด

```text
2 × 3
```

รวม 6 ช่อง

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │         │
└─────────┴─────────┴─────────┘
```

เนื่องจากสุ่มแค่ 5 รูป จึงเหลือ 1 ช่องว่าง

---

# 10. แปลง Axes เป็น 1 มิติ

```python
ax = ax.ravel()
```

ทำให้เรียกช่องง่ายเป็น

```text
ax[0]
ax[1]
ax[2]
ax[3]
ax[4]
ax[5]
```

---

# 11. Predict แต่ละ Image

```python
for i, img_path in enumerate(samples):
```

วนทีละ Image ที่สุ่มได้

จากนั้น

```python
result = model.predict(
    str(img_path),
    conf=0.5,
    imgsz=960,
    device=0,
    verbose=False
)[0]
```

ให้ Model ตรวจจับ Object

---

# 12. `conf=0.5`

```python
conf=0.5
```

กำหนด Confidence Threshold

หมายความว่า Model ต้องมีความมั่นใจอย่างน้อย

```text
50%
```

จึงนำ Detection นั้นมาใช้

ตัวอย่าง

```text
Object A → 0.92 ✓
Object B → 0.76 ✓
Object C → 0.43 ✗
```

เมื่อ `conf=0.5`

จะเหลือ

```text
Object A
Object B
```

---

# 13. `imgsz=960`

```python
imgsz=960
```

กำหนดขนาด Image ที่ใช้ใน Prediction

```text
960 × 960
```

ให้สอดคล้องกับ Training ที่ใช้ใน STEP 9.0

---

# 14. `device=0`

```python
device=0
```

ใช้ GPU ตัวที่ 0

ช่วยให้ Prediction เร็วกว่าการใช้ CPU ในกรณีที่มี GPU ที่รองรับ

---

# 15. `verbose=False`

```python
verbose=False
```

ลดข้อความที่ YOLO แสดงใน Terminal

ทำให้ Output สะอาดขึ้น

---

# 16. นับจำนวน Object

```python
count = len(result.boxes)
```

`result.boxes` คือ Bounding Box ที่ Model ตรวจพบ

ดังนั้น

```text
3 Boxes
↓
3 Objects
```

ในกรณี Dataset มีเพียง 1 Class ก็สามารถใช้จำนวน Box เป็นจำนวน Object ที่ตรวจพบได้

---

# 17. อ่านภาพต้นฉบับ

```python
img = cv2.imread(str(img_path))
```

อ่าน Image ด้วย OpenCV

จากนั้น

```python
h, w = img.shape[:2]
```

อ่าน

```text
h = Height
w = Width
```

---

# 18. วน Bounding Box

```python
for box in result.boxes:
```

วนทุก Detection ที่ Model ตรวจพบ

ตัวอย่าง

```text
result.boxes

Box 1
Box 2
Box 3
```

ก็จะวนทั้งหมด 3 รอบ

---

# 19. อ่าน Bounding Box แบบ Pixel

```python
x1, y1, x2, y2 = box.xyxy[0].tolist()
```

`xyxy` หมายถึง

```text
x1 = ซ้าย
y1 = บน
x2 = ขวา
y2 = ล่าง
```

ต่างจาก YOLO Label ที่เราเคยอ่านใน STEP 7.0 / 8.5 ซึ่งเป็น

```text
x_center
y_center
width
height
```

ตรงนี้ Model ให้ Coordinate แบบ

```text
(x1, y1, x2, y2)
```

พร้อมใช้งานสำหรับ OpenCV

---

# 20. วาด Bounding Box

```python
cv2.rectangle(
    img,
    (int(x1), int(y1)),
    (int(x2), int(y2)),
    (0, 255, 0),
    2
)
```

วาดกรอบสีเขียว

```text
(x1,y1)
    ↓
┌──────────────┐
│    Object    │
└──────────────┘
             ↑
          (x2,y2)
```

ใช้ความหนา

```text
2 Pixel
```

---

# 21. แสดง Image

```python
ax[i].imshow(
    cv2.cvtColor(
        img,
        cv2.COLOR_BGR2RGB
    )
)
```

OpenCV อ่านเป็น BGR

แต่ Matplotlib ต้องการ RGB

จึงแปลง

```text
BGR
 ↓
RGB
```

ก่อนแสดง

---

# 22. แสดงจำนวน Object บนภาพ

```python
ax[i].set_title(
    f"Count: {count}",
    fontsize=14,
    fontweight="bold"
)
```

ถ้า Model ตรวจพบ 4 Object จะขึ้นด้านบนของรูปว่า

```text
Count: 4
```

ทำให้ดูผลได้ง่าย

---

# 23. ปิด Axis

```python
ax[i].axis("off")
```

ซ่อนแกน X/Y

ทำให้เห็นเฉพาะ

```text
Image
+
Bounding Box
+
Count
```

---

# 24. ซ่อนช่องที่เหลือ

```python
for a in ax[len(samples):]:
    a.axis("off")
```

สมมติสุ่มได้ 5 รูป

Grid มี 6 ช่อง

ช่องสุดท้ายจะถูกซ่อน

ผลคือ

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │         │
└─────────┴─────────┴─────────┘
```

---

# 25. แสดงผล

```python
plt.tight_layout()
plt.show()
```

`tight_layout()` จัดระยะห่างของรูป

จากนั้น `show()` แสดงภาพ

---

# 26. แสดงหัวข้อ STEP 10

```python
print("\n=================================")
print("STEP 10 — MODEL TEST")
print("=================================")
```

แสดงหัวข้อใน Terminal

---

# 27. Predict ซ้ำเพื่อแสดงผลใน Terminal

```python
for img_path in samples:

    result = model.predict(
        str(img_path),
        conf=0.5,
        imgsz=960,
        device=0,
        verbose=False
    )[0]

    print(
        f"{img_path.name} : "
        f"{len(result.boxes)} Objects"
    )
```

ส่วนนี้ทำ Prediction อีกครั้งสำหรับ Image เดิม

แล้วแสดงจำนวน Object

ตัวอย่าง

```text
image001.jpg : 3 Objects
image002.jpg : 5 Objects
image003.jpg : 1 Objects
image004.jpg : 0 Objects
image005.jpg : 2 Objects
```

---

# 28. ผลลัพธ์ที่ได้

STEP 10.0 จะมี Output สองแบบ

## แบบที่ 1 — Visual

แสดงรูปพร้อม Bounding Box

```text
┌─────────────────┐
│   Count: 3      │
│                 │
│   ┌────┐        │
│   │    │        │
│   └────┘ ┌───┐  │
│          │   │  │
│          └───┘  │
└─────────────────┘
```

## แบบที่ 2 — Terminal

```text
=================================
STEP 10 — MODEL TEST
=================================
image001.jpg : 3 Objects
image002.jpg : 5 Objects
image003.jpg : 1 Objects
image004.jpg : 0 Objects
image005.jpg : 2 Objects
=================================
```

---

# 29. STEP 10.0 ตรวจอะไร?

STEP 10.0 ไม่ได้ตรวจแค่จำนวน Object

ควรดูด้วยว่า

### ① Model ตรวจเจอ Object หรือไม่

```text
Object → ✓
```

### ② Bounding Box อยู่ถูกตำแหน่งหรือไม่

```text
┌────────────┐
│   Object   │
└────────────┘
```

### ③ มี False Positive หรือไม่

```text
┌────────────┐
│            │
│            │ ← ไม่มี Object
└────────────┘
```

แต่ Model กลับตรวจพบ

### ④ มี False Negative หรือไม่

มี Object จริง แต่ Model ไม่ตรวจพบ

```text
Object A ✓
Object B ✗
Object C ✓
```

---

# 30. `Count` ไม่เท่ากับความถูกต้อง

สมมติภาพจริงมี

```text
5 Objects
```

Model ตรวจพบ

```text
Count: 5
```

ไม่ได้หมายความว่า Model ถูกต้อง 100%

อาจเป็น

```text
Object จริง
A ✓
B ✓
C ✓
D ✓
E ✓
```

แต่ Model อาจตรวจ

```text
A ✓
B ✓
C ✓
D ✗
F ✗
```

จำนวนยังเป็น 5 แต่ Detection ผิด 2 จุด

ดังนั้นต้องดู **Bounding Box ด้วยสายตา** ไม่ใช่ดูเฉพาะ Count

---

# 31. จุดสำคัญ — สุ่มรูปจาก Dataset

โค้ดใช้

```python
random.sample()
```

ดังนั้นทุกครั้งที่รันอาจได้รูปไม่เหมือนเดิม

ตัวอย่าง

```text
Run 1
→ A B C D E

Run 2
→ F G H I J
```

ข้อดีคือสามารถสุ่มตรวจหลายชุดได้

---

# 32. ถ้าต้องการให้สุ่มได้ชุดเดิม

สามารถเพิ่ม

```python
random.seed(42)
```

ก่อน

```python
random.sample()
```

เช่น

```python
random.seed(42)

samples = random.sample(
    images,
    min(5, len(images))
)
```

จะทำให้การสุ่มมีผลลัพธ์เดิมเมื่อข้อมูลชุดเดิม

เหมาะกับการเปรียบเทียบ Model หลาย Version

---

# 33. จุดที่ควรปรับปรุง — Predict ซ้ำสองครั้ง

ในโค้ดปัจจุบัน

ครั้งแรก Predict เพื่อวาดรูป

```python
result = model.predict(...)
```

ครั้งที่สอง Predict เพื่อพิมพ์ Count

```python
result = model.predict(...)
```

ดังนั้นแต่ละ Image ถูก Predict **2 รอบ**

```text
Image
 ↓
Predict #1 → วาด Box
 ↓
Predict #2 → นับ Box
```

ไม่จำเป็นต้อง Predict ซ้ำ

สามารถเก็บผลไว้ได้ เช่น

```python
results = []

for i, img_path in enumerate(samples):

    result = model.predict(
        str(img_path),
        conf=0.5,
        imgsz=960,
        device=0,
        verbose=False
    )[0]

    results.append(
        (img_path, result)
    )
```

จากนั้นนำ `results` ไปใช้ทั้งแสดงรูปและพิมพ์ Count

จะลดการทำงานของ Model ลงครึ่งหนึ่งสำหรับส่วนนี้

---

# 34. จุดที่ควรระวังเรื่อง Test Dataset

โค้ดนี้ใช้

```python
DATASET.rglob("*")
```

แล้วข้ามเฉพาะ

```text
P711_Final
```

ดังนั้น Image จาก Folder อื่นภายใน `P711Test` ยังสามารถถูกสุ่มได้

นี่เหมาะสมถ้า Folder เหล่านั้นเป็น **ข้อมูลที่ไม่ได้ใช้ Train**

แต่ถ้ารูปเหล่านั้นเป็นรูปที่ใช้ Train ใน Seed หรือ Final แล้ว ก็ไม่ควรถือว่าเป็น Test แบบอิสระ

หลักที่ดีที่สุดคือ

```text
TRAIN
   ≠
VALIDATION
   ≠
TEST
```

---

# 35. Train / Validation / Test

สำหรับการประเมิน Model อย่างจริงจัง ควรมีข้อมูลแยกกัน

```text
Dataset
│
├── Train
│   └── ใช้สอน Model
│
├── Val
│   └── ใช้ประเมินระหว่าง Training
│
└── Test
    └── ใช้ประเมิน Model หลัง Training
```

ดังนั้น STEP 10.0 จะมีคุณค่ามากที่สุดถ้า `images` ที่นำมาสุ่มเป็น **Test Set ที่ Model ไม่เคยเห็นใน Training**

---

# 36. Pipeline ทั้งหมด

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
Build Seed data.yaml
      ↓
STEP 5.0
Train Seed Model
      ↓
Seed best.pt
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
STEP 9.0
Train Final Model
      ↓
Final best.pt
      ↓
STEP 10.0
Model Test
      ↓
Visual Detection
+
Object Count
```

---

# 37. สรุป STEP 10.0

| Code                        | หน้าที่                 |
| --------------------------- | ----------------------- |
| `MODEL`                     | ตำแหน่ง Final `best.pt` |
| `YOLO()`                    | โหลด Final Model        |
| `rglob("*")`                | ค้นหา Image             |
| `P711_Final not in parents` | ข้าม Final Dataset      |
| `random.sample()`           | สุ่ม Image              |
| `model.predict()`           | ตรวจจับ Object          |
| `conf=0.5`                  | Confidence ขั้นต่ำ 50%  |
| `imgsz=960`                 | Prediction Image Size   |
| `device=0`                  | ใช้ GPU 0               |
| `result.boxes`              | Detection Boxes         |
| `len(result.boxes)`         | จำนวน Detection         |
| `box.xyxy`                  | Pixel Bounding Box      |
| `cv2.rectangle()`           | วาด Bounding Box        |
| `imshow()`                  | แสดงผล                  |
| `set_title()`               | แสดง Count              |
| `print()`                   | รายงานผลใน Terminal     |

## สรุปสั้นที่สุด

```text
P711Test_Final/best.pt
        ↓
     STEP 10.0
        ↓
   สุ่ม 5 รูป
        ↓
     Predict
        ↓
┌─────────────────┐
│ Bounding Box    │
│ + Count Objects │
└─────────────────┘
        ↓
ตรวจด้วยสายตา
        ↓
ประเมินว่า Final Model
ใช้งานได้ดีหรือไม่
```

**STEP 10.0 = Final Model Testing**

และมีข้อสังเกตสำคัญที่สุด 2 เรื่อง:

1. **Count อย่างเดียวบอกไม่ได้ว่า Model ถูกต้อง** ต้องดูตำแหน่ง Bounding Box และ False Positive/False Negative ด้วย
2. ถ้าต้องการวัดประสิทธิภาพจริง ควรใช้ **Test Set ที่ไม่เคยถูกใช้ในการ Train** แยกจาก `P711_Final` ไม่ใช่สุ่มจากข้อมูลที่ Model เคยเห็น
