# STEP 6 — AUTO LABEL IMAGES

## 1. จุดประสงค์ของ Step นี้

Step 6 นำ Model ที่ได้จาก Step 5 คือ

```text
best.pt
```

มาใช้สำหรับ **ทำนาย Object ในรูปที่ยังไม่มี Label**

จากนั้นนำผลการทำนายมาแปลงเป็น YOLO Label Format และสร้างไฟล์ `.txt` ให้กับแต่ละรูปโดยอัตโนมัติ

แนวคิดหลักคือ

```text
รูปที่ยังไม่มี Label
        ↓
     best.pt
        ↓
   YOLO Prediction
        ↓
  Bounding Boxes
        ↓
สร้างไฟล์ .txt
```

---

# 2. Code ทั้งหมดของ Step 6

```python
from pathlib import Path
from ultralytics import YOLO

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

# หา Run ของ Seed
RUN = Path("runs/detect") / f"{DATASET.name}_Seed"
MODEL = RUN / "weights/best.pt"

if not MODEL.exists():
    raise FileNotFoundError(f"ไม่พบโมเดล: {MODEL}")

model = YOLO(str(MODEL))

# รูปที่ยังไม่มี Label
labels = {p.stem for p in DATASET.rglob("*.txt")}

images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and p.stem not in labels
]

# Auto Label
for img in images:
    result = model.predict(
        str(img),
        conf=0.5,
        imgsz=960,
        device=0,
        verbose=False
    )[0]

    with open(img.with_suffix(".txt"), "w") as f:
        for box in result.boxes:
            cls = int(box.cls[0])
            x, y, w, h = box.xywhn[0].tolist()
            f.write(f"{cls} {x:.6f} {y:.6f} {w:.6f} {h:.6f}\n")

print(f"Auto Label เสร็จ: {len(images)} รูป")
```

---

# 3. Import Library

```python
from pathlib import Path
from ultralytics import YOLO
```

ใช้ Library 2 ตัว

### `Path`

ใช้จัดการไฟล์และ Folder

### `YOLO`

ใช้โหลด Model และทำ Prediction

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

# 5. หา Run ของ Seed

```python
RUN = Path("runs/detect") / f"{DATASET.name}_Seed"
```

`DATASET.name` จะได้ชื่อ Folder Dataset

ในกรณีนี้

```python
DATASET.name
```

ได้

```text
P711Test
```

ดังนั้น

```python
f"{DATASET.name}_Seed"
```

จะกลายเป็น

```text
P711Test_Seed
```

จากนั้นนำมาต่อกับ

```text
runs/detect
```

จึงได้

```text
runs/detect/P711Test_Seed
```

---

# 6. กำหนดตำแหน่ง Model

```python
MODEL = RUN / "weights/best.pt"
```

นำ Folder ของ Run มาต่อกับ

```text
weights/best.pt
```

จึงได้ตำแหน่งประมาณ

```text
runs/detect/P711Test_Seed/weights/best.pt
```

ไฟล์นี้คือ Model ที่ได้จาก Step 5

---

# 7. ตรวจสอบว่า Model มีอยู่จริงหรือไม่

```python
if not MODEL.exists():
    raise FileNotFoundError(f"ไม่พบโมเดล: {MODEL}")
```

ตรวจสอบว่าไฟล์

```text
best.pt
```

มีอยู่จริงหรือไม่

ถ้าไม่มี

```python
MODEL.exists()
```

จะเป็น

```python
False
```

จากนั้นโปรแกรมจะหยุดด้วย

```python
FileNotFoundError
```

ตัวอย่าง Error

```text
FileNotFoundError: ไม่พบโมเดล: runs/detect/P711Test_Seed/weights/best.pt
```

ข้อดีคือจะรู้ทันทีว่า Model หายหรือ Path ไม่ถูกต้อง แทนที่จะปล่อยให้เกิด Error ในขั้นตอน Prediction

---

# 8. โหลด Model

```python
model = YOLO(str(MODEL))
```

นำ `best.pt` เข้า YOLO

โดย `MODEL` เป็น `Path`

จึงใช้

```python
str(MODEL)
```

เพื่อแปลงเป็น String

แนวคิดคือ

```text
best.pt
   ↓
YOLO()
   ↓
model
```

จากนี้ตัวแปร

```python
model
```

จะเป็น Model ที่พร้อมสำหรับ Prediction

---

# 9. หา Label ที่มีอยู่แล้ว

```python
labels = {p.stem for p in DATASET.rglob("*.txt")}
```

ค้นหาไฟล์ `.txt` ทั้งหมดใน Dataset

ตัวอย่าง

```text
image001.jpg
image001.txt

image002.jpg
image002.txt

image003.jpg
```

เมื่อใช้

```python
p.stem
```

จะได้

```text
image001
image002
```

ดังนั้น

```python
labels
```

จะเก็บชื่อของรูปที่มี Label อยู่แล้ว

ตัวอย่าง

```python
labels = {
    "image001",
    "image002"
}
```

---

# 10. หาเฉพาะรูปที่ยังไม่มี Label

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and p.stem not in labels
]
```

ส่วนนี้สำคัญมาก เพราะ Step 6 **ไม่ต้องการ Auto Label รูปที่มี Label อยู่แล้ว**

ต้องการเฉพาะ

> รูปที่มี Image แต่ยังไม่มี `.txt`

---

## 10.1 ค้นหาไฟล์ทั้งหมด

```python
DATASET.rglob("*")
```

ค้นหาไฟล์และ Folder ทุกอย่างภายใน Dataset รวมถึง Subfolder

---

## 10.2 ตรวจสอบนามสกุลรูป

```python
p.suffix.lower() in [".jpg", ".jpeg", ".png"]
```

เลือกเฉพาะ

```text
.jpg
.jpeg
.png
```

---

## 10.3 ตรวจสอบว่าไม่มี Label

```python
p.stem not in labels
```

เช่น

```text
image001.jpg
image001.txt
```

จะไม่ถูกเลือก เพราะ

```text
image001
```

อยู่ใน `labels`

แต่ถ้าเป็น

```text
image003.jpg
```

และไม่มี

```text
image003.txt
```

จะถูกเลือก

---

# 11. ตัวอย่างการคัดรูป

สมมติ Dataset มี

```text
P711Test/
├── image001.jpg
├── image001.txt
├── image002.jpg
├── image002.txt
├── image003.jpg
├── image004.jpg
└── image005.jpg
```

หลังจากค้นหา Label

```python
labels
```

จะประมาณ

```text
{"image001", "image002"}
```

ดังนั้น `images` จะเหลือ

```text
image003.jpg
image004.jpg
image005.jpg
```

เพื่อนำไป Auto Label

---

# 12. เริ่ม Auto Label

```python
for img in images:
```

วนทีละรูปที่ยังไม่มี Label

ตัวอย่าง

```text
image003.jpg
      ↓
Prediction

image004.jpg
      ↓
Prediction

image005.jpg
      ↓
Prediction
```

---

# 13. Model Prediction

```python
result = model.predict(
    str(img),
    conf=0.5,
    imgsz=960,
    device=0,
    verbose=False
)[0]
```

ให้ Model ทำนาย Object ในรูป

---

## `str(img)`

```python
str(img)
```

คือ Path ของรูปที่กำลังทำนาย

เช่น

```text
C:\Users\pawor\Desktop\P711Test\image003.jpg
```

---

## `conf=0.5`

```python
conf=0.5
```

กำหนด Confidence Threshold เป็น `0.5`

หมายถึงใช้ Detection ที่มี Confidence ตั้งแต่ประมาณ

```text
50%
```

ขึ้นไป

แนวคิด

```text
Confidence < 0.5
        ↓
      ไม่เอา

Confidence >= 0.5
        ↓
       เอา
```

---

## `imgsz=960`

```python
imgsz=960
```

ใช้ขนาดภาพ

```text
960 × 960
```

ในการ Prediction

ให้สอดคล้องกับขนาดที่ใช้ใน Step 5

---

## `device=0`

```python
device=0
```

ใช้ GPU ตัวที่ 0 ในการ Prediction

---

## `verbose=False`

```python
verbose=False
```

ลดข้อความรายละเอียดที่แสดงใน Console

---

# 14. `[0]`

```python
)[0]
```

ผลจาก

```python
model.predict()
```

สามารถคืนผลลัพธ์เป็นรายการของ Results

ในกรณีที่ส่งรูปทีละรูป จึงเลือกผลลัพธ์ตัวแรกด้วย

```python
[0]
```

แล้วเก็บไว้ใน

```python
result
```

---

# 15. สร้างไฟล์ Label

```python
with open(img.with_suffix(".txt"), "w") as f:
```

สร้างไฟล์ `.txt` ที่มีชื่อเดียวกับรูป

ตัวอย่าง

```text
image003.jpg
```

จะกลายเป็น

```text
image003.txt
```

คำสั่ง

```python
img.with_suffix(".txt")
```

ทำหน้าที่เปลี่ยนนามสกุลของไฟล์

---

# 16. `"w"` คืออะไร?

```python
open(..., "w")
```

`w` หมายถึง Write

คือเปิดไฟล์เพื่อเขียนข้อมูล

ถ้าไฟล์ไม่มีอยู่

> สร้างใหม่

ถ้ามีอยู่

> เขียนทับ

อย่างไรก็ตาม Step 6 เลือกเฉพาะรูปที่ยังไม่มี Label อยู่แล้ว จึงโดยปกติจะเป็นการสร้างไฟล์ใหม่

---

# 17. วนดู Bounding Box

```python
for box in result.boxes:
```

`result.boxes` คือ Bounding Boxes ที่ Model ตรวจพบ

ตัวอย่างถ้า Model ตรวจพบ 2 Object

```text
result.boxes
    │
    ├── box 1
    └── box 2
```

โปรแกรมจะวนทีละ Box

---

# 18. หา Class ID

```python
cls = int(box.cls[0])
```

ดึง Class ID ของ Object

ตัวอย่าง

```text
Class 0
Class 1
Class 2
```

แล้วแปลงเป็น Integer ด้วย

```python
int(...)
```

เช่น

```python
cls = 0
```

---

# 19. ดึง Bounding Box แบบ Normalized

```python
x, y, w, h = box.xywhn[0].tolist()
```

ส่วนนี้ดึงข้อมูล Bounding Box ในรูปแบบ YOLO

```text
x
y
w
h
```

โดย `xywhn` หมายถึง

```text
x = center X
y = center Y
w = width
h = height
```

และตัว `n` หมายถึง **Normalized**

ค่าจะอยู่ในช่วงประมาณ

```text
0.0 - 1.0
```

แทนการใช้ Pixel โดยตรง

---

# 20. YOLO Label Format

ข้อมูลที่เขียนลงไฟล์จะมีรูปแบบ

```text
class x_center y_center width height
```

ตัวอย่าง

```text
0 0.512500 0.430000 0.250000 0.300000
```

แปลว่า

```text
Class       = 0
Center X    = 0.512500
Center Y    = 0.430000
Width       = 0.250000
Height      = 0.300000
```

---

# 21. เขียน Label ลงไฟล์

```python
f.write(
    f"{cls} {x:.6f} {y:.6f} {w:.6f} {h:.6f}\n"
)
```

เขียนข้อมูลของ Bounding Box ลงไฟล์ `.txt`

---

## `:.6f`

เช่น

```python
x = 0.512345678
```

เมื่อใช้

```python
{x:.6f}
```

จะกลายเป็น

```text
0.512346
```

คือแสดงทศนิยม 6 ตำแหน่ง

---

# 22. ตัวอย่างผลลัพธ์

ถ้า Model ตรวจพบ Object 2 ตัวใน

```text
image003.jpg
```

โปรแกรมอาจสร้าง

```text
image003.txt
```

ภายในเป็น

```text
0 0.512500 0.430000 0.250000 0.300000
1 0.700000 0.600000 0.150000 0.200000
```

แต่ละบรรทัดคือ Bounding Box 1 อัน

---

# 23. แสดงจำนวนรูปที่ Auto Label

```python
print(f"Auto Label เสร็จ: {len(images)} รูป")
```

นับจำนวนรูปที่ถูกนำเข้า Auto Label

เช่นถ้ามี 150 รูป

```text
Auto Label เสร็จ: 150 รูป
```

---

# 24. Flow ของ Step 6

```text
P711Test
   │
   ▼
หา best.pt
   │
   ▼
โหลด Model
   │
   ▼
ค้นหาไฟล์ .txt ที่มีอยู่แล้ว
   │
   ▼
ค้นหารูปที่ยังไม่มี Label
   │
   ▼
วนทีละรูป
   │
   ▼
YOLO Predict
   │
   ▼
Confidence >= 0.5
   │
   ▼
ได้ Bounding Boxes
   │
   ▼
แปลงเป็น YOLO Format
   │
   ▼
สร้าง image.txt
   │
   ▼
ทำจนครบทุกรูป
```

---

# 25. ผลลัพธ์ก่อนและหลัง Step 6

### ก่อน Step 6

```text
P711Test/
├── image001.jpg
├── image001.txt
├── image002.jpg
├── image002.txt
├── image003.jpg
├── image004.jpg
└── image005.jpg
```

รูป `image003`, `image004`, `image005` ยังไม่มี Label

---

### หลัง Step 6

```text
P711Test/
├── image001.jpg
├── image001.txt
├── image002.jpg
├── image002.txt
├── image003.jpg
├── image003.txt   ← Auto Label
├── image004.jpg
├── image004.txt   ← Auto Label
├── image005.jpg
└── image005.txt   ← Auto Label
```

ดังนั้น Step 6 จะช่วยเพิ่ม Label ให้กับรูปที่ยังไม่มี `.txt`

---

# 26. สรุป Step 6

Step 6 ทำหน้าที่

1. หา Run ของ Seed
2. หา `weights/best.pt`
3. ตรวจสอบว่า Model มีอยู่จริง
4. โหลด Model
5. ค้นหา Label ที่มีอยู่แล้ว
6. ค้นหารูปที่ยังไม่มี Label
7. ใช้ Model ทำนายรูปเหล่านั้น
8. ใช้ Confidence Threshold `0.5`
9. ใช้ Image Size `960`
10. ใช้ GPU `device=0`
11. ดึง Class และ Bounding Box
12. แปลงข้อมูลเป็น YOLO Format
13. สร้างไฟล์ `.txt` คู่กับรูป
14. แสดงจำนวนรูปที่ Auto Label

**เป้าหมายสำคัญของ Step นี้คือ**

```text
รูปที่ยังไม่มี Label
        ↓
    best.pt
        ↓
     Predict
        ↓
   Auto Label
        ↓
สร้าง .txt ให้รูป
```

ดังนั้น Step 6 เป็นขั้นตอน **Pseudo Labeling / Auto Labeling** เพื่อเพิ่ม Label ให้กับ Dataset โดยใช้ Model ที่ Train จาก Seed Dataset ใน Step 5
