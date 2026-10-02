# STEP 9 — TRAIN FINAL MODEL

## 1. จุดประสงค์ของ Step นี้

Step 9 คือขั้นตอน **Train โมเดล Final** จาก Dataset ที่สร้างไว้ใน `P711_Final`

โดย Step นี้จะทำงานหลัก ๆ 2 ส่วน:

1. สร้างไฟล์ `data.yaml` สำหรับ `P711_Final`
2. นำ `yolo11n.pt` มา Train ด้วย Dataset Final

ผลลัพธ์ที่ต้องการคือไฟล์โมเดล:

```text
best.pt
```

ซึ่งจะเป็นโมเดลที่ Train จาก Dataset Final แล้ว

---

# 2. โค้ดทั้งหมด

```python
from pathlib import Path
from ultralytics import YOLO
import yaml

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

FINAL = DATASET / "P711_Final"

# =========================
# สร้าง data.yaml
# =========================
data = {
    "path": str(FINAL),
    "train": "images",
    "val": "images",
    "names": {0: "Object"}
}

with open(FINAL / "data.yaml", "w", encoding="utf-8") as f:
    yaml.dump(data, f, allow_unicode=True, sort_keys=False)

# =========================
# Train Final Model
# =========================
model = YOLO(
    r"C:\Users\pawor\Desktop\yolo11n.pt"
)

results = model.train(
    data=str(FINAL / "data.yaml"),
    epochs=150,
    imgsz=960,
    batch=8,
    device=0,
    name=f"{DATASET.name}_Final",
    verbose=False
)

print("\n=================================")
print("STEP 9 — TRAIN FINAL MODEL")
print("=================================")
print("Model :", Path(results.save_dir) / "weights" / "best.pt")
print("=================================")
```

---

# 3. Import Library

```python
from pathlib import Path
from ultralytics import YOLO
import yaml
```

มี 3 Library หลัก

### `Path`

```python
from pathlib import Path
```

ใช้จัดการ Path ของไฟล์และ Folder

เช่น

```python
DATASET / "P711_Final"
```

จะกลายเป็น

```text
C:\Users\pawor\Desktop\P711Test\P711_Final
```

---

### `YOLO`

```python
from ultralytics import YOLO
```

ใช้โหลดโมเดลและ Train

เช่น

```python
model = YOLO("yolo11n.pt")
```

---

### `yaml`

```python
import yaml
```

ใช้สร้างไฟล์

```text
data.yaml
```

สำหรับบอก YOLO ว่า Dataset อยู่ที่ไหน และมี Class อะไรบ้าง

---

# 4. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

ตัวแปร `DATASET` คือ Folder หลักของ Dataset

โครงสร้างประมาณนี้:

```text
P711Test/
│
├── P711_Seed/
│
├── P711_Final/
│
└── ...
```

จุดสำคัญคือใช้

```python
r"..."
```

เพื่อให้ Windows Path ที่มี `\` ไม่เกิดปัญหากับ Escape Character

---

# 5. กำหนด Final Dataset

```python
FINAL = DATASET / "P711_Final"
```

คือการสร้าง Path ไปยัง Final Dataset

ถ้า

```python
DATASET
```

คือ

```text
C:\Users\pawor\Desktop\P711Test
```

ดังนั้น

```python
FINAL
```

จะเป็น

```text
C:\Users\pawor\Desktop\P711Test\P711_Final
```

---

# 6. สร้างข้อมูลสำหรับ `data.yaml`

```python
data = {
    "path": str(FINAL),
    "train": "images",
    "val": "images",
    "names": {0: "Object"}
}
```

ตรงนี้เป็นการสร้าง Dictionary สำหรับเขียนลง `data.yaml`

โครงสร้างที่ได้จะประมาณ:

```yaml
path: C:\Users\pawor\Desktop\P711Test\P711_Final
train: images
val: images
names:
  0: Object
```

---

# 7. `path`

```python
"path": str(FINAL)
```

กำหนด Root ของ Dataset

ตัวอย่าง:

```text
C:\Users\pawor\Desktop\P711Test\P711_Final
```

ใช้

```python
str(FINAL)
```

เพราะ `FINAL` เป็น `Path Object`

จึงแปลงเป็น String ก่อนเขียน YAML

---

# 8. `train`

```python
"train": "images"
```

หมายความว่า Training Dataset อยู่ใน Folder:

```text
P711_Final/images
```

เมื่อรวมกับ `path`:

```text
C:\Users\pawor\Desktop\P711Test\P711_Final
+
images
```

จะได้:

```text
C:\Users\pawor\Desktop\P711Test\P711_Final\images
```

---

# 9. `val`

```python
"val": "images"
```

กำหนด Validation Dataset

ในโค้ดนี้ใช้ Folder เดียวกับ Training:

```text
P711_Final/images
```

ดังนั้น

```text
train → images
val   → images
```

คือใช้รูปชุดเดียวกันทั้ง Train และ Validation

> หมายเหตุ: สำหรับการประเมินความสามารถของโมเดลแบบจริงจัง โดยทั่วไปควรแยก Train และ Validation ออกจากกัน เพื่อให้ Validation ใช้ภาพที่โมเดลไม่เคยเห็นตอน Train

---

# 10. `names`

```python
"names": {0: "Object"}
```

กำหนด Class ของ Dataset

ในที่นี้มีเพียง 1 Class:

```text
Class ID = 0
Class Name = Object
```

ดังนั้น Label YOLO ที่เป็น:

```text
0 0.512000 0.480000 0.250000 0.300000
```

เลขตัวแรก:

```text
0
```

หมายถึง Class:

```text
Object
```

---

# 11. เปิดไฟล์ `data.yaml`

```python
with open(FINAL / "data.yaml", "w", encoding="utf-8") as f:
```

สร้างไฟล์:

```text
P711_Final/data.yaml
```

### `"w"`

หมายถึง Write

ถ้ามีไฟล์เดิมอยู่แล้ว จะเขียนทับไฟล์เดิม

### `encoding="utf-8"`

ใช้ UTF-8 สำหรับเขียนข้อความ

---

# 12. เขียน YAML

```python
yaml.dump(
    data,
    f,
    allow_unicode=True,
    sort_keys=False
)
```

นำ Dictionary:

```python
data
```

ไปเขียนลงไฟล์ YAML

### `allow_unicode=True`

อนุญาตให้เขียน Unicode ได้

เช่นภาษาไทย:

```yaml
names:
  0: วัตถุ
```

### `sort_keys=False`

ไม่เรียง Key ใหม่

จึงรักษาลำดับตามที่กำหนดไว้:

```text
path
train
val
names
```

---

# 13. ผลลัพธ์หลังสร้าง `data.yaml`

Folder จะมีลักษณะ:

```text
P711_Final/
│
├── images/
│   ├── image001.jpg
│   ├── image002.jpg
│   └── ...
│
├── labels/
│   ├── image001.txt
│   ├── image002.txt
│   └── ...
│
└── data.yaml
```

โดย `data.yaml` ทำหน้าที่เป็นตัวบอก YOLO ว่า Dataset อยู่ตรงไหน

---

# 14. โหลด Base Model

```python
model = YOLO(
    r"C:\Users\pawor\Desktop\yolo11n.pt"
)
```

โหลดโมเดล:

```text
yolo11n.pt
```

เข้ามาในตัวแปร:

```python
model
```

จากนั้นจึงสามารถใช้:

```python
model.train()
```

เพื่อ Train ได้

---

# 15. เริ่ม Train

```python
results = model.train(
```

คำสั่งนี้เป็นหัวใจหลักของ Step 9

YOLO จะอ่าน:

```text
P711_Final/data.yaml
```

แล้วนำ Dataset ไป Train

ผลลัพธ์จากการ Train ถูกเก็บไว้ใน:

```python
results
```

---

# 16. `data`

```python
data=str(FINAL / "data.yaml")
```

บอก YOLO ว่าให้ใช้ Dataset Configuration จาก:

```text
P711_Final/data.yaml
```

เช่น:

```text
C:\Users\pawor\Desktop\P711Test\P711_Final\data.yaml
```

---

# 17. `epochs=150`

```python
epochs=150
```

กำหนดจำนวนรอบในการ Train:

```text
150 Epochs
```

ตัวอย่างแนวคิด:

```text
Dataset
   ↓
Epoch 1
   ↓
Epoch 2
   ↓
...
   ↓
Epoch 150
```

หนึ่ง Epoch คือการที่โมเดลประมวลผล Training Dataset ครบหนึ่งรอบ

---

# 18. `imgsz=960`

```python
imgsz=960
```

กำหนดขนาดภาพที่ใช้ในการ Train เป็น:

```text
960 × 960
```

YOLO จะปรับภาพเข้าสู่ขนาดที่กำหนดก่อนนำไป Train

---

# 19. `batch=8`

```python
batch=8
```

กำหนดจำนวนภาพต่อหนึ่ง Batch เป็น:

```text
8 images
```

แนวคิด:

```text
Image 1 ─┐
Image 2  │
Image 3  │
Image 4  │
Image 5  ├── Batch 1
Image 6  │
Image 7  │
Image 8 ─┘
```

จากนั้นจึงคำนวณและปรับ Model

---

# 20. `device=0`

```python
device=0
```

กำหนดให้ใช้ GPU หมายเลข:

```text
GPU 0
```

โดยทั่วไปในเครื่องที่มี GPU NVIDIA และ CUDA พร้อมใช้งาน:

```text
device=0
```

หมายถึง GPU ตัวแรก

---

# 21. `name`

```python
name=f"{DATASET.name}_Final"
```

`DATASET.name` คือชื่อ Folder สุดท้ายของ Path

จาก:

```text
C:\Users\pawor\Desktop\P711Test
```

จะได้:

```text
P711Test
```

ดังนั้น:

```python
f"{DATASET.name}_Final"
```

จะได้:

```text
P711Test_Final
```

Ultralytics จะใช้ชื่อนี้เป็นชื่อ Run

โดยทั่วไปผลลัพธ์จะอยู่ประมาณ:

```text
runs/detect/P711Test_Final/
```

---

# 22. `verbose=False`

```python
verbose=False
```

ลดข้อความรายละเอียดที่แสดงใน Terminal ระหว่าง Train

แทนที่จะให้แสดงรายละเอียดจำนวนมาก จะเหลือ Output ที่จำเป็นมากขึ้น

---

# 23. ผลลัพธ์จากการ Train

หลังจาก Train เสร็จ:

```python
results
```

จะเก็บข้อมูลเกี่ยวกับ Run ที่เพิ่ง Train

จากนั้นโค้ดใช้:

```python
results.save_dir
```

เพื่อหา Folder ที่ใช้เก็บผลลัพธ์

---

# 24. หา `best.pt`

```python
Path(results.save_dir) / "weights" / "best.pt"
```

โดยทั่วไปโครงสร้างจะเป็น:

```text
runs/
└── detect/
    └── P711Test_Final/
        ├── weights/
        │   ├── best.pt
        │   └── last.pt
        │
        ├── results.csv
        ├── results.png
        └── ...
```

ไฟล์สำคัญคือ:

```text
best.pt
```

ซึ่งเป็น Weight ที่ Ultralytics บันทึกไว้ว่าเป็น Best Model ตามตัวชี้วัดของการ Train/Validation ที่ใช้ใน Run นั้น

---

# 25. Print Summary

```python
print("\n=================================")
print("STEP 9 — TRAIN FINAL MODEL")
print("=================================")
print("Model :", Path(results.save_dir) / "weights" / "best.pt")
print("=================================")
```

ใช้แสดงสรุปหลัง Train เสร็จ

ตัวอย่าง:

```text
=================================
STEP 9 — TRAIN FINAL MODEL
=================================
Model : runs\detect\P711Test_Final\weights\best.pt
=================================
```

ทำให้สามารถรู้ทันทีว่าโมเดล Final ถูกสร้างไว้ที่ไหน

---

# 26. Flow ของ Step 9

```text
P711_Final
     │
     ├── images/
     │
     └── labels/
     │
     ↓
สร้าง data.yaml
     │
     ↓
โหลด yolo11n.pt
     │
     ↓
YOLO Train
     │
     ├── epochs = 150
     ├── imgsz  = 960
     ├── batch  = 8
     └── GPU    = 0
     │
     ↓
P711Test_Final
     │
     └── weights/
          ├── best.pt
          └── last.pt
```

---

# 27. ความสัมพันธ์กับ Step ก่อนหน้า

ภาพรวม Pipeline:

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
สร้าง P711_Final
        ↓
STEP 9
Train Final Model
        ↓
      best.pt
```

ดังนั้น Step 9 เป็นขั้นตอนที่นำ Dataset Final ซึ่งรวมข้อมูลจาก Pipeline ก่อนหน้าแล้ว มา Train เป็นโมเดล Final

---

# 28. สิ่งที่ควรได้หลังจบ Step 9

ควรมีอย่างน้อย:

```text
P711_Final/
│
├── images/
├── labels/
└── data.yaml
```

และใน Run ของ Ultralytics:

```text
runs/
└── detect/
    └── P711Test_Final/
        └── weights/
            ├── best.pt
            └── last.pt
```

โมเดลที่ Step นี้รายงานออกมาคือ:

```text
best.pt
```

---

# 29. สรุป

Step 9 ทำหน้าที่:

```text
P711_Final
     ↓
สร้าง data.yaml
     ↓
โหลด yolo11n.pt
     ↓
Train 150 Epochs
     ↓
ใช้ภาพขนาด 960
     ↓
Batch 8
     ↓
GPU 0
     ↓
P711Test_Final
     ↓
best.pt
```

**เป้าหมายสุดท้ายของ Step 9 คือสร้าง Final Model จาก `P711_Final` เพื่อนำไปใช้ในขั้นตอนถัดไป เช่น Predict/Test กับภาพใหม่**
