# STEP 5 — TRAIN YOLO ด้วย SEED DATASET

## 1. จุดประสงค์ของ Step นี้

Step 5 คือขั้นตอน **Train YOLO Model ด้วย Dataset ที่เตรียมไว้ใน `P711_Seed`**

โดย Step ก่อนหน้าได้สร้าง

```text
P711_Seed/
└── data.yaml
```

จากนั้น Step 5 จะนำ `data.yaml` นี้ไปให้ YOLO ใช้สำหรับ Training

ผลลัพธ์ที่สำคัญที่สุดคือ

```text
best.pt
```

ซึ่งเป็น Weight ของโมเดลที่ได้จากการ Train และสามารถนำไปใช้ในขั้นตอนถัดไปได้

---

# 2. Code ทั้งหมดของ Step 5

```python
from pathlib import Path
from ultralytics import YOLO

# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")

# หา data.yaml ของ Seed
yaml = next(f for f in DATASET.rglob("data.yaml")
            if "seed" in f.parent.name.lower())

model = YOLO(r"C:\Users\pawor\Desktop\yolo11n.pt")

results = model.train(
    data=str(yaml),
    epochs=100,     # รอบ Train
    imgsz=960,      # ขนาดภาพ
    batch=8,        # จำนวนภาพต่อรอบ
    device=0,       # GPU
    name=f"{DATASET.name}_Seed",
    verbose=False
)

print("\n===== TRAIN SUMMARY =====")
print("Model  :", Path(results.save_dir) / "weights" / "best.pt")
print("=========================")
```

---

# 3. Import Library

```python
from pathlib import Path
from ultralytics import YOLO
```

มีการนำเข้า 2 อย่าง

### `Path`

มาจาก

```python
from pathlib import Path
```

ใช้จัดการ Path ของไฟล์และ Folder

เช่น

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

ทำให้สามารถเขียน

```python
DATASET.rglob(...)
```

หรือ

```python
DATASET.name
```

ได้

---

### `YOLO`

```python
from ultralytics import YOLO
```

ใช้สำหรับโหลด YOLO Model และสั่ง Train

ตัวอย่างเช่น

```python
model = YOLO("yolo11n.pt")
```

และ

```python
model.train(...)
```

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

โดยใช้

```python
Path(...)
```

เพื่อให้ Python สามารถจัดการ Path ได้ง่ายขึ้น

---

# 5. หา data.yaml ของ Seed

```python
yaml = next(f for f in DATASET.rglob("data.yaml")
            if "seed" in f.parent.name.lower())
```

ส่วนนี้มีหน้าที่หาไฟล์

```text
data.yaml
```

ที่อยู่ใน Folder ซึ่งมีชื่อเกี่ยวกับ

```text
Seed
```

---

## 5.1 `rglob("data.yaml")`

```python
DATASET.rglob("data.yaml")
```

หมายถึง

> ค้นหาไฟล์ชื่อ `data.yaml` ในทุก Folder และ Subfolder ภายใน Dataset

ตัวอย่างโครงสร้าง

```text
P711Test/
├── data.yaml
├── P711_Seed/
│   └── data.yaml
├── folder1/
│   └── data.yaml
└── folder2/
    └── data.yaml
```

`rglob()` จะค้นหา `data.yaml` ทั้งหมด

---

## 5.2 `f.parent.name`

```python
f.parent.name
```

หมายถึงชื่อ Folder ที่ไฟล์ `data.yaml` อยู่

เช่น

```text
P711_Seed/data.yaml
```

จะได้

```python
f.parent.name
```

เป็น

```text
P711_Seed
```

---

## 5.3 `.lower()`

```python
f.parent.name.lower()
```

แปลงชื่อ Folder ให้เป็นตัวพิมพ์เล็กทั้งหมด

เช่น

```text
P711_Seed
```

จะกลายเป็น

```text
p711_seed
```

ทำให้สามารถตรวจสอบคำว่า `seed` ได้โดยไม่สนใจตัวพิมพ์ใหญ่/เล็ก

---

## 5.4 ตรวจสอบ `"seed" in ...`

```python
if "seed" in f.parent.name.lower()
```

หมายถึง

> เลือกเฉพาะ `data.yaml` ที่อยู่ใน Folder ซึ่งมีคำว่า `seed`

เช่น

```text
P711_Seed
```

ผ่านเงื่อนไข เพราะมีคำว่า

```text
Seed
```

---

# 6. `next(...)`

```python
yaml = next(
    f for f in DATASET.rglob("data.yaml")
    if "seed" in f.parent.name.lower()
)
```

`next()` จะเอา **ผลลัพธ์ตัวแรก** ที่ตรงกับเงื่อนไข

ดังนั้นถ้ามี

```text
P711_Seed/data.yaml
```

ก็จะได้ Path ของไฟล์นี้มาเก็บไว้ในตัวแปร

```python
yaml
```

เช่น

```python
yaml
```

อาจมีค่าเป็น

```text
C:\Users\pawor\Desktop\P711Test\P711_Seed\data.yaml
```

---

# 7. โหลด YOLO Model

```python
model = YOLO(r"C:\Users\pawor\Desktop\yolo11n.pt")
```

โหลด Weight เริ่มต้นของ YOLO

ในที่นี้คือ

```text
yolo11n.pt
```

โดย

```text
yolo11n
```

คือ YOLO รุ่น Nano

และ

```text
.pt
```

คือไฟล์ Weight ของโมเดล

แนวคิดคือ

```text
yolo11n.pt
      │
      ▼
โหลดเข้า YOLO
      │
      ▼
Train ด้วย Dataset ของเรา
```

---

# 8. เริ่ม Training

```python
results = model.train(
```

เรียกคำสั่ง

```python
model.train()
```

เพื่อเริ่ม Training

และเก็บผลลัพธ์ไว้ใน

```python
results
```

---

# 9. `data=str(yaml)`

```python
data=str(yaml)
```

บอก YOLO ว่า Dataset Configuration อยู่ที่ไหน

ตัวแปร

```python
yaml
```

เป็น `Path`

จึงแปลงเป็น String ด้วย

```python
str(yaml)
```

ตัวอย่าง

```text
C:\Users\pawor\Desktop\P711Test\P711_Seed\data.yaml
```

YOLO จะอ่านข้อมูลจากไฟล์นี้ เช่น

```yaml
path: ...
train: images
val: images
names:
  ...
```

---

# 10. `epochs=100`

```python
epochs=100
```

กำหนดจำนวนรอบในการ Train

เท่ากับ

```text
100 Epochs
```

แนวคิดง่าย ๆ คือ

```text
Dataset
   ↓
Train รอบที่ 1
   ↓
Train รอบที่ 2
   ↓
...
   ↓
Train รอบที่ 100
```

ยิ่ง Epoch มาก ไม่ได้หมายความว่าโมเดลจะดีขึ้นเสมอไป เพราะอาจเกิด Overfitting ได้

---

# 11. `imgsz=960`

```python
imgsz=960
```

กำหนดขนาดภาพที่ YOLO ใช้ในการ Training

ในที่นี้คือ

```text
960 × 960
```

โดยทั่วไป YOLO จะปรับภาพให้เหมาะกับขนาดที่กำหนดก่อนนำไป Train

---

# 12. `batch=8`

```python
batch=8
```

กำหนดจำนวนภาพที่นำเข้า Model ต่อหนึ่ง Batch

ในที่นี้คือ

```text
8 images / batch
```

ตัวอย่างแนวคิด

```text
Image 1 ─┐
Image 2  │
Image 3  │
Image 4  │
Image 5  ├── Batch 1
Image 6  │
Image 7  │
Image 8 ─┘

        ↓

      YOLO
        ↓

    Update Model
```

ค่า `batch` มีผลต่อการใช้ VRAM ของ GPU

---

# 13. `device=0`

```python
device=0
```

กำหนดให้นำ GPU ตัวที่ 0 มาใช้ในการ Train

โดยทั่วไป

```text
device=0
```

หมายถึง GPU ตัวแรกที่ระบบมองเห็น

ถ้าไม่มี GPU หรือไม่สามารถใช้ GPU ได้ การกำหนดนี้อาจทำให้เกิด Error

---

# 14. `name=f"{DATASET.name}_Seed"`

```python
name=f"{DATASET.name}_Seed"
```

กำหนดชื่อของ Training Run

จาก

```python
DATASET.name
```

ถ้า Dataset คือ

```text
P711Test
```

จะได้

```text
P711Test_Seed
```

ดังนั้น Folder สำหรับเก็บผล Training จะใช้ชื่อประมาณ

```text
P711Test_Seed
```

---

# 15. `verbose=False`

```python
verbose=False
```

ปิดการแสดงรายละเอียดบางส่วนของ Training ใน Console

ทำให้หน้าจอไม่แสดง Log จำนวนมาก

---

# 16. ผลลัพธ์จาก `model.train()`

```python
results = model.train(...)
```

หลังจาก Train เสร็จ YOLO จะคืนผลลัพธ์กลับมาเก็บใน

```python
results
```

ตัวแปรนี้มีข้อมูลเกี่ยวกับ Training Run รวมถึงตำแหน่งที่ใช้เก็บผลลัพธ์

---

# 17. แสดงหัวข้อ Summary

```python
print("\n===== TRAIN SUMMARY =====")
```

แสดงข้อความ

```text
===== TRAIN SUMMARY =====
```

เพื่อบอกว่าเข้าสู่ส่วนสรุปผล Training

---

# 18. หา `best.pt`

```python
Path(results.save_dir) / "weights" / "best.pt"
```

ส่วนนี้สร้าง Path ไปยัง Weight ที่สำคัญที่สุดของ Training

โครงสร้างคือ

```text
results.save_dir
       │
       └── weights
             │
             └── best.pt
```

ดังนั้น

```python
Path(results.save_dir) / "weights" / "best.pt"
```

จะได้ตำแหน่งไฟล์ประมาณ

```text
...\runs\detect\P711Test_Seed\weights\best.pt
```

---

# 19. `best.pt` คืออะไร?

หลังจาก Training YOLO จะสร้าง Weight หลายไฟล์ เช่น

```text
weights/
├── best.pt
└── last.pt
```

โดย

### `best.pt`

คือ Weight จาก Epoch ที่ได้ผลประเมินที่ดีที่สุดตาม metric ที่ใช้ในการฝึก

### `last.pt`

คือ Weight จาก Epoch สุดท้ายที่ Train

ดังนั้น Step นี้จึงแสดงตำแหน่งของ

```text
best.pt
```

เพื่อให้นำไปใช้ต่อได้ง่าย

---

# 20. แสดงตำแหน่ง Model

```python
print("Model  :", Path(results.save_dir) / "weights" / "best.pt")
```

ตัวอย่าง Output

```text
===== TRAIN SUMMARY =====
Model  : runs\detect\P711Test_Seed\weights\best.pt
=========================
```

จุดสำคัญคือไฟล์

```text
best.pt
```

นี่คือ Model ที่จะสามารถนำไปใช้ในขั้นตอนถัดไป เช่น

```text
Prediction
Validation
Testing
Inference
```

---

# 21. Flow ของ Step 5

การทำงานทั้งหมดสามารถมองเป็น Flow ได้ดังนี้

```text
P711Test
   │
   ├── P711_Seed
   │      └── data.yaml
   │
   ▼
ค้นหา data.yaml
   │
   ▼
โหลด yolo11n.pt
   │
   ▼
model.train()
   │
   ├── epochs = 100
   ├── imgsz  = 960
   ├── batch  = 8
   └── device = GPU 0
   │
   ▼
Training
   │
   ▼
Save Results
   │
   └── weights
          ├── best.pt
          └── last.pt
```

---

# 22. สรุป Step 5

Step 5 ทำหน้าที่

1. กำหนด Dataset หลัก
2. ค้นหา `data.yaml` ของ `P711_Seed`
3. โหลด YOLO Model จาก `yolo11n.pt`
4. เริ่ม Training
5. Train จำนวน `100 epochs`
6. ใช้ภาพขนาด `960`
7. ใช้ Batch ขนาด `8`
8. ใช้ GPU ตัวที่ `0`
9. ตั้งชื่อ Training Run เป็น `<ชื่อ Dataset>_Seed`
10. หาและแสดงตำแหน่ง `best.pt`

ผลลัพธ์สำคัญของ Step นี้คือ

```text
best.pt
```

ซึ่งเป็น Weight ที่จะนำไปใช้เป็น Model สำหรับขั้นตอนต่อไป
