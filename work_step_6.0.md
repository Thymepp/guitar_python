# 6.0 Auto Label

## 1. วัตถุประสงค์

โค้ดนี้ใช้ **Seed Model (`best.pt`) ที่ได้จาก STEP 5.0** เพื่อทำนายภาพที่ยังไม่มี Label แล้วสร้าง YOLO `.txt` ให้อัตโนมัติ

Flow:

```text id="h1v2x3"
STEP 5.0
best.pt
   │
   ▼
Load Model
   │
   ▼
หา Images ที่ยังไม่มี Label
   │
   ▼
Model Predict
   │
   ▼
Bounding Boxes
   │
   ▼
สร้าง YOLO .txt
```

---

# 2. Import Library

```python
from pathlib import Path
from ultralytics import YOLO
```

### `Path`

ใช้จัดการ Path ของ Dataset, Model และ Image

### `YOLO`

ใช้โหลด Model และทำ Prediction

---

# 3. กำหนด Dataset

```python
DATASET = Path(
    r"C:\Users\pawor\Desktop\P711Test"
)
```

กำหนด Dataset หลัก

---

# 4. หา Run ของ Seed Model

```python
RUN = Path("runs/detect") / f"{DATASET.name}_Seed"
```

`DATASET.name` คือ:

```text
P711Test
```

ดังนั้น:

```python
f"{DATASET.name}_Seed"
```

จะได้:

```text
P711Test_Seed
```

และ `RUN` จะชี้ไปที่:

```text
runs/detect/P711Test_Seed/
```

---

# 5. กำหนดตำแหน่ง `best.pt`

```python
MODEL = RUN / "weights/best.pt"
```

ตำแหน่ง Model:

```text
runs/
└── detect/
    └── P711Test_Seed/
        └── weights/
            └── best.pt
```

---

# 6. ตรวจว่า Model มีอยู่หรือไม่

```python
if not MODEL.exists():
    raise FileNotFoundError(
        f"ไม่พบโมเดล: {MODEL}"
    )
```

ถ้าไม่มี:

```text
best.pt
```

โปรแกรมจะหยุดทันที

ตัวอย่าง Error:

```text
FileNotFoundError:
ไม่พบโมเดล: runs\detect\P711Test_Seed\weights\best.pt
```

ช่วยป้องกันการ Run Prediction โดยไม่มี Model

---

# 7. โหลด Model

```python
model = YOLO(str(MODEL))
```

โหลด:

```text
best.pt
```

เข้าสู่ YOLO Model

จากนั้น:

```text
model
```

พร้อมสำหรับ Prediction

---

# 8. หา Image ที่ยังไม่มี Label

ก่อนอื่นสร้าง Set ของ Label:

```python
labels = {
    p.stem
    for p in DATASET.rglob("*.txt")
}
```

ตัวอย่าง:

```text
P711_001.txt
P711_002.txt
P711_003.txt
```

จะกลายเป็น:

```python
{
    "P711_001",
    "P711_002",
    "P711_003"
}
```

---

# 9. หา Images ที่ไม่มี Label

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [
        ".jpg",
        ".jpeg",
        ".png"
    ]
    and p.stem not in labels
]
```

เงื่อนไขสำคัญ:

```python
p.stem not in labels
```

หมายถึง:

> เลือกเฉพาะ Image ที่ยังไม่มี `.txt`

ตัวอย่าง:

```text
P711_001.jpg
P711_001.txt
```

ไม่เอา

แต่:

```text
P711_010.jpg
```

ไม่มี:

```text
P711_010.txt
```

จึงนำไป Auto Label

---

# 10. Auto Label ทีละ Image

```python
for img in images:
```

วน Image ที่ยังไม่มี Label ทีละรูป

---

# 11. Model Prediction

```python
result = model.predict(
    str(img),
    conf=0.5,
    imgsz=960,
    device=0,
    verbose=False
)[0]
```

ให้ Model ทำนาย Object ใน Image

---

# 12. `conf=0.5`

```python
conf=0.5
```

กำหนด Confidence Threshold:

```text
50%
```

โดยทั่วไป Detection ที่มี Confidence ต่ำกว่า Threshold จะไม่ถูกนำมาใช้

ตัวอย่าง:

```text
Object A → 0.92 → ผ่าน
Object B → 0.76 → ผ่าน
Object C → 0.42 → ไม่ผ่าน
```

ดังนั้น:

```text
conf = 0.5
```

หมายถึงเลือก Detection ที่ Confidence ตั้งแต่ประมาณ 0.5 ขึ้นไป

---

# 13. `imgsz=960`

```python
imgsz=960
```

ใช้ Input Size:

```text
960 × 960
```

ให้สอดคล้องกับการ Train ใน STEP 5.0:

```text
STEP 5.0
imgsz = 960
```

---

# 14. `device=0`

```python
device=0
```

ใช้ GPU หมายเลข:

```text
GPU 0
```

สำหรับ Prediction

---

# 15. `verbose=False`

```python
verbose=False
```

ไม่แสดงรายละเอียด Prediction ทุก Image ออกทาง Console

ทำให้ Console สะอาดขึ้น

---

# 16. `[0]`

```python
)[0]
```

`model.predict()` คืนผลลัพธ์เป็น Collection ของ Results

ในกรณีที่ส่ง Image ทีละหนึ่งรูป:

```python
model.predict(...)[0]
```

คือผลลัพธ์ของ Image นั้น

---

# 17. เปิดไฟล์ YOLO Label

```python
with open(
    img.with_suffix(".txt"),
    "w"
) as f:
```

เปลี่ยน:

```text
P711_010.jpg
```

เป็น:

```text
P711_010.txt
```

และเปิดด้วยโหมด:

```text
w = write
```

---

# 18. วน Bounding Box

```python
for box in result.boxes:
```

Model อาจตรวจพบหลาย Object ในภาพเดียว

ตัวอย่าง:

```text
Image
 ├── Object 1
 ├── Object 2
 └── Object 3
```

จึงวนทีละ Bounding Box

---

# 19. อ่าน Class ID

```python
cls = int(box.cls[0])
```

ดึง Class ID จาก Detection

ตัวอย่าง:

```text
Class ID = 0
```

หรือ:

```text
Class ID = 1
```

ต้องตรงกับ `names` ใน `data.yaml`

---

# 20. อ่าน Bounding Box แบบ YOLO

```python
x, y, w, h = box.xywhn[0].tolist()
```

`xywhn` หมายถึง:

```text
x = Center X
y = Center Y
w = Width
h = Height
```

ตัวอักษร `n` หมายถึง **Normalized**

ดังนั้นค่าจะอยู่ในช่วงประมาณ:

```text
0.0 → 1.0
```

ตัวอย่าง:

```text
0.512
0.431
0.120
0.220
```

---

# 21. สร้าง YOLO Label

```python
f.write(
    f"{cls} "
    f"{x:.6f} "
    f"{y:.6f} "
    f"{w:.6f} "
    f"{h:.6f}\n"
)
```

เขียนข้อมูลในรูปแบบ:

```text
class x_center y_center width height
```

ตัวอย่าง:

```text
0 0.512000 0.431000 0.120000 0.220000
```

นี่คือมาตรฐาน YOLO Bounding Box Format

---

# 22. ตัวอย่างการทำงาน

สมมติ:

```text
P711_010.jpg
```

ยังไม่มี Label

Model ตรวจพบ:

```text
Class 0
Confidence 0.87
```

และ Bounding Box:

```text
x = 0.52
y = 0.48
w = 0.20
h = 0.30
```

โปรแกรมจะสร้าง:

```text
P711_010.txt
```

ภายใน:

```text
0 0.520000 0.480000 0.200000 0.300000
```

---

# 23. ถ้าพบหลาย Object

สมมติ Image มี 3 Object:

```text
Class 0
Class 1
Class 0
```

ไฟล์ Label จะเป็น:

```text
0 0.52 0.48 0.20 0.30
1 0.20 0.40 0.15 0.25
0 0.75 0.60 0.18 0.22
```

**หนึ่งบรรทัด = หนึ่ง Object**

---

# 24. กรณี Model ไม่พบ Object

ถ้า:

```python
result.boxes
```

ไม่มี Detection

Loop:

```python
for box in result.boxes:
```

จะไม่ทำงาน

ดังนั้นไฟล์:

```text
P711_010.txt
```

จะถูกสร้างเป็นไฟล์ว่าง

ซึ่งสามารถหมายถึง:

```text
Image ไม่มี Object ที่ Model ตรวจพบ
```

---

# 25. ⚠️ จุดสำคัญของโค้ดนี้

โค้ดกำหนดว่า:

```python
labels = {
    p.stem
    for p in DATASET.rglob("*.txt")
}
```

ดังนั้น **ทุก `.txt` ถูกถือว่าเป็น Label**

รวมถึงไฟล์:

```text
P711_Empty_Selection.txt
```

ด้วย

แต่เนื่องจากชื่อ:

```text
P711_Empty_Selection
```

ไม่น่าจะตรงกับชื่อ Image จึงโดยทั่วไปไม่กระทบการเลือก Image

อย่างไรก็ตาม หากต้องการให้ Logic สะอาดขึ้น ควรตัดไฟล์ Selection ออกโดยตรง

เช่น:

```python
labels = {
    p.stem
    for p in DATASET.rglob("*.txt")
    if p.name != "P711_Empty_Selection.txt"
}
```

---

# 26. ⚠️ จุดสำคัญอีกอย่าง: Empty Image

STEP 2.5 เลือก Empty Images ไว้ใน:

```text
P711_Empty_Selection.txt
```

แต่ STEP 6.0 ใช้:

```python
p.stem not in labels
```

เป็นตัวตัดสินว่า Image ไหนไม่มี Label

ดังนั้นภาพ Empty ที่ถูกเลือกไว้ แต่ยังไม่มี `.txt` จริง จะถูกนำมา Auto Label ด้วย

นี่อาจ **ไม่ใช่สิ่งที่ต้องการ** ถ้า Empty Images ถูกกำหนดไว้แล้วว่าเป็น Negative Samples

ถ้าต้องการรักษา Empty Images ไม่ให้ Auto Label ควรเพิ่ม:

```python
and p.stem not in empty_names
```

---

# 27. Logic ที่แนะนำ

สำหรับ Pipeline นี้ ควรแยกเป็น:

```text
Labeled
   ↓
ไม่ Auto Label

Empty ที่มนุษย์เลือก
   ↓
ไม่ Auto Label

Unlabeled ที่เหลือ
   ↓
Auto Label
```

ดังนั้น:

```python
empty_file = DATASET / "P711_Empty_Selection.txt"

empty_names = {
    Path(x).stem
    for x in empty_file.read_text(
        encoding="utf-8"
    ).splitlines()
    if x.strip()
} if empty_file.exists() else set()
```

แล้วเลือก Image:

```python
images = [
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and p.stem not in labels
    and p.stem not in empty_names
]
```

จะทำให้:

```text
มี Label
   → ข้าม

เป็น Empty ที่มนุษย์ยืนยัน
   → ข้าม

ไม่มี Label และไม่ใช่ Empty
   → Auto Label
```

---

# 28. สรุปผล

```python
print(
    f"Auto Label เสร็จ: {len(images)} รูป"
)
```

ตัวอย่าง:

```text
Auto Label เสร็จ: 350 รูป
```

หมายถึงมี Image ที่ถูกนำเข้า Prediction จำนวน 350 รูป

**ไม่ได้หมายความว่า Model ตรวจพบ Object 350 ตัว**

---

# 29. Flow ของ STEP 6.0

```text
P711_Seed/best.pt
        │
        ▼
     Load YOLO
        │
        ▼
หา Image ที่ไม่มี Label
        │
        ▼
    Model Predict
        │
        ├── Confidence ≥ 0.5
        │
        ▼
   Detection Boxes
        │
        ▼
Class + XYWH Normalized
        │
        ▼
สร้าง .txt
```

---

# 30. ความสัมพันธ์กับ STEP 5.0

### STEP 5.0

```text
Seed Dataset
     +
yolo11n.pt
     ↓
Train
     ↓
best.pt
```

### STEP 6.0

```text
best.pt
     +
Unlabeled Images
     ↓
Predict
     ↓
YOLO Labels
```

ดังนั้น:

> **STEP 5.0 สร้าง Model ส่วน STEP 6.0 ใช้ Model นั้นสร้าง Label**

---

# 31. Pipeline รวมถึง STEP 6.0

```text
STEP 1
CHECK DATASET
       ↓
STEP 2.0
VISUALIZE LABEL
       ↓
STEP 2.5
SELECT EMPTY
       ↓
STEP 3.0
BUILD SEED
       ↓
STEP 4.0
BUILD DATA.YAML
       ↓
STEP 5.0
TRAIN SEED MODEL
       ↓
best.pt
       ↓
STEP 6.0
AUTO LABEL
       ↓
สร้าง YOLO .txt
       ↓
Dataset มี Label เพิ่มขึ้น
```

---

# 32. สรุป

**STEP 6.0 = Auto Label**

หน้าที่หลัก:

```text
best.pt
   ↓
หา Image ที่ยังไม่มี Label
   ↓
YOLO Predict
   ↓
กรองด้วย Confidence 0.5
   ↓
อ่าน Class + Bounding Box
   ↓
แปลงเป็น YOLO Format
   ↓
สร้าง .txt
```

ผลลัพธ์:

```text
P711_010.jpg
P711_010.txt   ← Auto Generated

P711_011.jpg
P711_011.txt   ← Auto Generated

P711_012.jpg
P711_012.txt   ← Auto Generated
```

> **STEP 6.0 คือขั้นตอน Pseudo-Labeling: ใช้ Model รุ่นแรกที่ Train จาก Seed Dataset มาช่วยสร้าง Annotation ให้กับภาพที่ยังไม่มี Label**
