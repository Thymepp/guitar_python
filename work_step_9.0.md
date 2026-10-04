# STEP 9.0 — TRAIN FINAL MODEL

## 1. หน้าที่ของ STEP 9.0

STEP 9.0 คือขั้นตอน **Train Model รอบ Final**

โดยนำ Dataset ที่สร้างจาก

```text
STEP 8.0 → P711_Final
```

และตรวจสอบด้วย

```text
STEP 8.5 → Visual Check
```

มาสร้าง `data.yaml` แล้วใช้ YOLO Train Model จำนวน 150 Epochs

ภาพรวมคือ

```text
STEP 8.0
Build Final Dataset
        ↓
P711_Final
        ↓
STEP 8.5
ตรวจสอบ Image + Label
        ↓
STEP 9.0
Train Final Model
        ↓
best.pt
        ↓
Final Model
```

---

# 2. Import Library

```python id="yxxp9x"
from pathlib import Path
from ultralytics import YOLO
import yaml
```

### `Path`

ใช้จัดการ Path ของ Dataset และ Model

### `YOLO`

ใช้โหลด Model และ Train ด้วย Ultralytics

### `yaml`

ใช้สร้างไฟล์ `data.yaml`

---

# 3. กำหนด Dataset

```python id="o1nxpy"
# เปลี่ยนแค่ตรงนี้
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

Dataset หลักคือ

```text id="x1y8hk"
C:\Users\pawor\Desktop\P711Test
```

---

# 4. กำหนด Final Dataset

```python id="3z1r3x"
FINAL = DATASET / "P711_Final"
```

ได้ Path

```text id="g4bj4r"
C:\Users\pawor\Desktop\P711Test\P711_Final
```

ซึ่งเป็น Dataset ที่ได้จาก STEP 8.0

---

# 5. สร้าง `data.yaml`

ส่วนนี้ใช้สร้าง Configuration สำหรับ YOLO

```python id="6r8q8q"
data = {
    "path": str(FINAL),
    "train": "images",
    "val": "images",
    "names": {0: "Object"}
}
```

ข้อมูลที่ได้จะมีประมาณนี้

```yaml id="gqzt6j"
path: C:\Users\pawor\Desktop\P711Test\P711_Final
train: images
val: images
names:
  0: Object
```

---

# 6. `path`

```python id="5t5bpf"
"path": str(FINAL)
```

กำหนด Root ของ Dataset

คือ

```text id="9p5g1w"
P711_Final
```

ดังนั้น

```yaml id="gk5m2b"
train: images
```

จะหมายถึง

```text id="e6n9su"
P711_Final/images
```

และ

```yaml id="7eq3dd"
val: images
```

จะหมายถึง

```text id="m3uq1v"
P711_Final/images
```

---

# 7. `train`

```python id="7wyz1d"
"train": "images"
```

กำหนด Folder ที่ใช้ Train

ในโค้ดนี้คือ

```text id="ihb2w4"
P711_Final/images
```

---

# 8. `val`

```python id="e3r1lq"
"val": "images"
```

กำหนด Folder ที่ใช้ Validation

แต่ในโค้ดนี้ **Train และ Validation ใช้ Image Folder เดียวกัน**

```text id="z9x0wv"
train → images
val   → images
```

ดังนั้น

```text id="q1d6w5"
P711_Final/images
        ↑
   ┌────┴────┐
 train      val
```

## ⚠️ จุดสำคัญ

นี่ไม่ใช่ Train/Validation Split แบบมาตรฐาน

เพราะ Image เดียวกันอาจถูกใช้ทั้ง

```text id="z8gqrs"
Training
```

และ

```text id="p4w9te"
Validation
```

ดังนั้นค่า Validation Metrics เช่น

```text id="l9n0fm"
mAP
Precision
Recall
```

อาจดูดีเกินจริง เพราะ Model ได้เห็นข้อมูลเหล่านั้นใน Training แล้ว

### สำหรับการทดลอง Seed

สามารถใช้ได้

แต่ถ้าจะประเมิน **Final Model อย่างจริงจัง** ควรแบ่งเป็น

```text id="9n8m1u"
P711_Final/
├── images/
│
├── labels/
│
├── train/
└── val/
```

หรือจัดโครงสร้าง Dataset ให้ Train และ Validation เป็นคนละชุดกัน

---

# 9. `names`

```python id="3mxv7u"
"names": {0: "Object"}
```

กำหนด Class

ใน Dataset นี้มี 1 Class

```text id="v7q9r3"
Class ID = 0
Class Name = Object
```

ดังนั้น Label

```text id="t7h1lq"
0 0.5 0.5 0.2 0.3
```

หมายถึง

```text id="5dj4gt"
Class = Object
```

---

# 10. เขียน `data.yaml`

```python id="6o9q9w"
with open(
    FINAL / "data.yaml",
    "w",
    encoding="utf-8"
) as f:
```

เปิดหรือสร้างไฟล์

```text id="4f4pxn"
P711_Final/data.yaml
```

โหมด

```python id="0o8f0x"
"w"
```

หมายถึง Write

ถ้ามี `data.yaml` เดิมอยู่แล้ว จะถูกเขียนทับ

---

# 11. `yaml.dump()`

```python id="q1i2q7"
yaml.dump(
    data,
    f,
    allow_unicode=True,
    sort_keys=False
)
```

แปลง Dictionary Python

```python id="f3mmv2"
data = {
    ...
}
```

ให้กลายเป็น YAML

### `allow_unicode=True`

อนุญาตให้เขียน Unicode ได้

### `sort_keys=False`

รักษาลำดับ Key ตามที่เขียนไว้

ทำให้ YAML อ่านง่าย เช่น

```yaml id="d6x7xx"
path: ...
train: images
val: images
names: ...
```

แทนที่จะถูกเรียงใหม่ตามตัวอักษร

---

# 12. โหลด Base Model

```python id="gj3sv2"
model = YOLO(
    r"C:\Users\pawor\Desktop\yolo11n.pt"
)
```

โหลด Pretrained Model

```text id="cxh0k7"
yolo11n.pt
```

จาก

```text id="l4v1d5"
C:\Users\pawor\Desktop\
```

Model นี้เป็นจุดเริ่มต้นของ Final Training

---

# 13. ทำไมไม่ Train จาก Model ว่าง?

เพราะ

```text id="yolo11n.pt"
```

เป็น Pretrained Model

มีความสามารถพื้นฐานในการเรียนรู้ Feature ของภาพอยู่แล้ว

ดังนั้นเราจะใช้

```text id="yolo11n.pt"
       ↓
P711 Seed Training
       ↓
Seed best.pt
       ↓
Auto Label
       ↓
P711 Final Dataset
       ↓
STEP 9
```

อย่างไรก็ตาม **โค้ด STEP 9.0 ปัจจุบันโหลด `yolo11n.pt` ใหม่** ไม่ได้โหลด `P711Test_Seed/weights/best.pt`

นี่เป็นจุดสำคัญของ Pipeline

---

# 14. Train Final Model

```python id="l44w09"
results = model.train(
    data=str(FINAL / "data.yaml"),
    epochs=150,
    imgsz=960,
    batch=8,
    device=0,
    name=f"{DATASET.name}_Final",
    verbose=False
)
```

คำสั่งนี้เริ่ม Training

---

# 15. `data`

```python id="1x8as1"
data=str(FINAL / "data.yaml")
```

บอก YOLO ว่า Dataset Configuration อยู่ที่ไหน

ตัวอย่าง

```text id="w6a0fp"
C:\Users\pawor\Desktop\P711Test\P711_Final\data.yaml
```

---

# 16. `epochs=150`

```python id="k0qvfj"
epochs=150
```

Train จำนวน

```text id="y1k9w0"
150 Epochs
```

1 Epoch หมายถึง Model ได้ประมวลผล Training Dataset ครบหนึ่งรอบ

ดังนั้น

```text id="4y7o1v"
150 Epochs
=
Dataset ถูกนำมา Train สูงสุด 150 รอบ
```

Ultralytics อาจหยุดก่อนครบตามเงื่อนไขการหยุดที่กำหนด หากมีการใช้ Early Stopping ตามค่า patience ของการ Train

---

# 17. `imgsz=960`

```python id="x3k9y5"
imgsz=960
```

กำหนด Image Size สำหรับ Training

```text id="8a3s4q"
960 × 960
```

โดยทั่วไป Image จะถูกปรับขนาดตามกระบวนการของ YOLO ก่อนเข้า Model

ข้อดีของ Image Size ที่สูงขึ้นคืออาจช่วยเรื่องวัตถุขนาดเล็กได้

แต่แลกกับ

* ใช้ VRAM มากขึ้น
* Training ช้าลง
* Batch อาจต้องลดลง

---

# 18. `batch=8`

```python id="v0k5uo"
batch=8
```

แต่ละ Training Iteration ใช้ Image จำนวน 8 รูป

โดยประมาณ

```text id="l4vlq6"
8 Images
   ↓
Forward
   ↓
Loss
   ↓
Backward
   ↓
Update Model
```

ถ้า GPU VRAM ไม่พอ อาจเกิด CUDA Out Of Memory

ในกรณีนั้นอาจต้องลดเป็น

```python id="b5q9or"
batch=4
```

หรือ

```python id="5y3klf"
batch=2
```

---

# 19. `device=0`

```python id="svb6t4"
device=0
```

หมายถึงใช้ GPU หมายเลข 0

โดยทั่วไป

```text id="hkj6fd"
device=0
```

หมายถึง GPU ตัวแรก

ถ้าไม่มี GPU ที่เหมาะสม อาจต้องเปลี่ยนเป็น CPU เช่น

```python id="j04t6b"
device="cpu"
```

แต่ Training จะช้ากว่ามาก

---

# 20. `name`

```python id="v6e1at"
name=f"{DATASET.name}_Final"
```

`DATASET.name` คือ

```text id="z22b3r"
P711Test
```

ดังนั้นชื่อ Run จะเป็น

```text id="v3fzv7"
P711Test_Final
```

โดยทั่วไปผลการ Train จะถูกเก็บในประมาณ

```text id="9dknr0"
runs/
└── detect/
    └── P711Test_Final/
        ├── weights/
        │   ├── best.pt
        │   └── last.pt
        ├── results.csv
        └── ...
```

ตำแหน่งจริงควรยึดจาก

```python id="vkv4cz"
results.save_dir
```

---

# 21. `verbose=False`

```python id="02ab6x"
verbose=False
```

ลดข้อความที่แสดงระหว่าง Training

ทำให้ Terminal ไม่แสดงรายละเอียดมากเกินไป

---

# 22. `results`

```python id="i4c3yw"
results = model.train(...)
```

เก็บผลลัพธ์จากการ Train

จากนั้นสามารถเข้าถึงตำแหน่ง Run ได้ผ่าน

```python id="q1h1km"
results.save_dir
```

---

# 23. แสดงตำแหน่ง Best Model

```python id="6tmm8f"
print(
    "Model :",
    Path(results.save_dir) / "weights" / "best.pt"
)
```

จะได้ประมาณ

```text id="x4d2vq"
Model : runs\detect\P711Test_Final\weights\best.pt
```

ไฟล์สำคัญคือ

```text id="26r5f5"
best.pt
```

---

# 24. `best.pt` คืออะไร?

ระหว่าง Training Model จะมี Checkpoint หลายช่วง

โดยทั่วไปจะมี

```text id="74pyw8"
best.pt
last.pt
```

### `best.pt`

คือ Checkpoint ที่ถือว่าดีที่สุดตาม Metric ที่ใช้เลือก Best Model ระหว่างการ Train

เหมาะสำหรับนำไปใช้งานหรือทดสอบต่อ

### `last.pt`

คือ Checkpoint จาก Epoch สุดท้ายที่ Training ดำเนินไปถึง

ดังนั้นโดยทั่วไปหลัง Train เสร็จ

```text id="qg8qjd"
best.pt
```

มักเป็นไฟล์ที่เราต้องการนำไปใช้

---

# 25. จุดสำคัญมากของ STEP 9.0

ปัจจุบันโค้ดเขียนว่า

```python id="q6xj4k"
model = YOLO(
    r"C:\Users\pawor\Desktop\yolo11n.pt"
)
```

หมายความว่า Final Training เริ่มจาก

```text id="0d4f4k"
yolo11n.pt
```

ไม่ใช่

```text id="qkrx1j"
P711Test_Seed/weights/best.pt
```

ดังนั้น Pipeline จริงคือ

```text id="3q1n5m"
yolo11n.pt
     ↓
STEP 5.0
     ↓
Seed best.pt
     ↓
Auto Label
     ↓
P711_Final
     ↓
STEP 9.0
     ↓
yolo11n.pt
     ↓
Final best.pt
```

ถ้าจุดประสงค์ของ Pipeline คือ **ใช้ความรู้จาก Seed Model มา Train ต่อ** ควรเปลี่ยน Model ที่ STEP 9.0 ไปใช้ Seed `best.pt`

เช่น

```python id="wgrxj4"
model = YOLO(
    r"C:\Users\pawor\Desktop\runs\detect\P711Test_Seed\weights\best.pt"
)
```

หรือกำหนด Path ให้ชัดเจนตามตำแหน่งจริงของ Run

ผลจะกลายเป็น

```text id="p49s5h"
yolo11n.pt
     ↓
Seed Training
     ↓
Seed best.pt
     ↓
Final Training
     ↓
Final best.pt
```

วิธีนี้คือการ **Fine-tune ต่อจาก Seed Model**

---

# 26. อีกจุดสำคัญ — `train` และ `val` ใช้ชุดเดียวกัน

ปัจจุบัน

```python id="g5ecm1"
"train": "images",
"val": "images",
```

หมายความว่า

```text id="y7u4r8"
Training Images
       =
Validation Images
```

จึงไม่ควรใช้ Metrics จาก Validation ชุดนี้เป็นตัววัด Generalization ของ Model แบบจริงจัง

โครงสร้างที่แนะนำสำหรับ Final Dataset คือ

```text id="0wmv3f"
P711_Final/
│
├── train/
│   ├── images/
│   └── labels/
│
├── val/
│   ├── images/
│   └── labels/
│
└── data.yaml
```

แล้ว

```yaml id="5l6v7m"
train: train/images
val: val/images
```

จะเป็นการประเมิน Model ได้เหมาะสมกว่า

---

# 27. ทำไม STEP 8.5 สำคัญก่อน STEP 9.0?

เพราะ STEP 8.0 รวม

```text id="h9hjce"
Seed
+
Auto Label
```

ถ้า Auto Label ผิดจำนวนมาก แล้วนำไป Train ทันที

```text id="yn0x7g"
Wrong Labels
     ↓
Training
     ↓
Model เรียนรู้ข้อมูลผิด
```

ดังนั้น

```text id="h6j4cs"
STEP 7.0
ตรวจ Auto Label
       ↓
STEP 8.0
Build Final
       ↓
STEP 8.5
ตรวจ Final
       ↓
STEP 9.0
Train
```

เป็นลำดับที่สมเหตุสมผล

---

# 28. ผลลัพธ์หลัง STEP 9.0

โดยทั่วไปจะได้ Folder Run ประมาณ

```text id="7t5qbm"
runs/
└── detect/
    └── P711Test_Final/
        │
        ├── weights/
        │   ├── best.pt
        │   └── last.pt
        │
        ├── results.csv
        ├── args.yaml
        ├── results.png
        └── ...
```

ไฟล์สำคัญที่สุดคือ

```text id="0hcvds"
weights/best.pt
```

นี่คือ **Final Model**

---

# 29. Pipeline ทั้งหมด

```text id="m72wkg"
                    DATASET
                       │
                       ▼
                 STEP 1
              Check Dataset
                       │
                       ▼
                STEP 2.0
            Visualize Labels
                       │
                       ▼
                STEP 2.5
             Select Empty
                       │
                       ▼
                STEP 3.0
             Build Seed
                       │
                       ▼
                STEP 4.0
             Build YAML
                       │
                       ▼
                STEP 5.0
             Train Seed
                       │
                       ▼
                 Seed best.pt
                       │
                       ▼
                STEP 6.0
              Auto Label
                       │
                       ▼
                STEP 7.0
           Check Auto Label
                       │
                       ▼
                STEP 8.0
            Build Final
                       │
                       ▼
                STEP 8.5
           Check Final Dataset
                       │
                       ▼
                STEP 9.0
           Train Final Model
                       │
                       ▼
                Final best.pt
```

---

# 30. สรุป STEP 9.0

| Code               | หน้าที่             |
| ------------------ | ------------------- |
| `DATASET`          | Dataset หลัก        |
| `FINAL`            | Final Dataset       |
| `data = {...}`     | สร้าง Configuration |
| `yaml.dump()`      | สร้าง `data.yaml`   |
| `YOLO()`           | โหลด Base Model     |
| `epochs=150`       | Train 150 Epochs    |
| `imgsz=960`        | Image Size 960      |
| `batch=8`          | Batch Size 8        |
| `device=0`         | ใช้ GPU 0           |
| `name=...Final`    | ตั้งชื่อ Run        |
| `results.save_dir` | ตำแหน่งผลการ Train  |
| `best.pt`          | Best Model          |

## สรุปสั้นที่สุด

```text id="5s1f4n"
P711_Final
    ↓
สร้าง data.yaml
    ↓
โหลด yolo11n.pt
    ↓
Train 150 Epochs
    ↓
Image Size 960
    ↓
Batch 8
    ↓
GPU 0
    ↓
P711Test_Final
    ↓
weights/best.pt
```

**STEP 9.0 = Final Training**

และมี **2 จุดที่ควรพิจารณาก่อนใช้จริง**:

1. `val: images` ทำให้ Train และ Validation เป็นชุดเดียวกัน → ควรแยก Train/Val หากต้องการประเมิน Model อย่างถูกต้อง
2. ตอนนี้ Final Train เริ่มจาก `yolo11n.pt` ใหม่ → ถ้าต้องการ **Train ต่อจาก Seed Model** ควรใช้ `P711Test_Seed/weights/best.pt` แทน
