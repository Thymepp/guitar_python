# 2.5 Select Empty Images

## 1. วัตถุประสงค์

โค้ดนี้ใช้สร้าง **GUI สำหรับเลือกภาพที่เป็น Empty / บล็อกเปล่า**

โดยมีหลักการ:

```text
Dataset
   ↓
ตัดภาพที่มี Label ออก
   ↓
เหลือเฉพาะภาพที่ยังไม่มี Label
   ↓
แสดงภาพผ่าน GUI
   ↓
ผู้ใช้เลือกภาพที่เป็น "บล็อกเปล่า"
   ↓
กด ยืนยัน
   ↓
บันทึกรายการลง P711_Empty_Selection.txt
```

---

# 2. Import Library

```python
from pathlib import Path
import tkinter as tk
from tkinter import messagebox
from PIL import Image, ImageTk
```

### `Path`

ใช้จัดการไฟล์และ Folder

### `tkinter`

ใช้สร้าง GUI

### `messagebox`

ใช้แสดงข้อความแจ้งเตือน เช่น:

```text
ยังไม่ได้เลือกรูป
```

หรือ:

```text
เลือกบล็อกเปล่าเสร็จแล้ว
```

### `PIL`

ใช้เปิดและย่อ Image ก่อนนำไปแสดงใน Tkinter

---

# 3. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset

---

# 4. หา Image ที่มี Label แล้ว

```python
labeled = {
    p.stem for p in DATASET.rglob("*.txt")
    if p.name != "P711_Empty_Selection.txt"
}
```

ส่วนนี้สำคัญมาก

ใช้ค้นหา `.txt` ทั้งหมด แล้วเอา `.stem` มาเก็บไว้

ตัวอย่าง:

```text
P711_001.txt
```

จะได้:

```text
P711_001
```

ดังนั้น:

```python
labeled
```

อาจมี:

```python
{
    "P711_001",
    "P711_002",
    "P711_003"
}
```

หมายความว่า Image เหล่านี้มี Label อยู่แล้ว

---

# 5. ไม่เอาไฟล์ Selection มานับเป็น Label

```python
if p.name != "P711_Empty_Selection.txt"
```

เนื่องจากไฟล์:

```text
P711_Empty_Selection.txt
```

เป็นไฟล์ที่ใช้เก็บรายชื่อภาพ Empty

ไม่ใช่ YOLO Label

จึงต้องตัดออก

---

# 6. หา Image ที่ยังไม่มี Label

```python
images = sorted([
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and p.stem not in labeled
])
```

ค้นหา Image:

```text
.jpg
.jpeg
.png
```

แต่มีเงื่อนไข:

```python
p.stem not in labeled
```

หมายถึง:

> เอาเฉพาะ Image ที่ยังไม่มี Label

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

จึงเอามาแสดงให้เลือก

---

# 7. ตัวแปรสำหรับเก็บภาพที่เลือก

```python
selected = set()
page = 0
```

### `selected`

เก็บชื่อ Image ที่ผู้ใช้เลือก

ตัวอย่าง:

```python
{
    "P711_010",
    "P711_015",
    "P711_023"
}
```

ใช้ `set()` เพราะไม่ต้องการชื่อซ้ำ

### `page`

เก็บหมายเลขหน้าปัจจุบัน

```text
0 = หน้าแรก
1 = หน้าที่สอง
2 = หน้าที่สาม
```

---

# 8. สร้างหน้าต่าง GUI

```python
root = tk.Tk()
root.title("STEP 2.5 - Select Empty Images")
root.geometry("1200x850")
```

สร้างหน้าต่างหลัก

ชื่อ:

```text
STEP 2.5 - Select Empty Images
```

ขนาด:

```text
1200 × 850
```

---

# 9. แสดงหัวข้อ

```python
tk.Label(
    root,
    text="เลือกภาพบล็อกเปล่า",
    font=("Arial", 22, "bold")
).pack(pady=10)
```

แสดงข้อความ:

```text
เลือกภาพบล็อกเปล่า
```

---

# 10. แสดงจำนวนภาพที่เลือก

```python
count = tk.Label(
    root,
    text="เลือกแล้ว: 0 รูป",
    font=("Arial", 14, "bold")
)
count.pack()
```

ตอนเริ่ม:

```text
เลือกแล้ว: 0 รูป
```

ถ้าเลือก 5 รูป:

```text
เลือกแล้ว: 5 รูป
```

---

# 11. สร้างพื้นที่แสดง Image

```python
frame = tk.Frame(root)
frame.pack(expand=True)
```

`frame` ทำหน้าที่เป็น Container สำหรับเก็บปุ่มรูปภาพ

---

# 12. เก็บปุ่ม Image

```python
buttons = []
```

เก็บปุ่มทั้งหมดที่กำลังแสดงอยู่

เพื่อให้สามารถลบออกเมื่อเปลี่ยนหน้าได้

---

# 13. Function `toggle()`

```python
def toggle(path):
    if path.stem in selected:
        selected.remove(path.stem)
    else:
        selected.add(path.stem)

    show()
```

ใช้สำหรับ:

> กด Image เพื่อเลือก / ยกเลิกการเลือก

### ถ้ายังไม่ได้เลือก

```python
selected.add(path.stem)
```

เพิ่มเข้าไป

### ถ้าเลือกอยู่แล้ว

```python
selected.remove(path.stem)
```

เอาออก

จากนั้น:

```python
show()
```

เพื่อ Refresh GUI

---

# 14. Function `show()`

```python
def show():
```

หน้าที่คือ:

> แสดง Image ของหน้าปัจจุบัน

---

## ลบปุ่มเก่า

```python
for b in buttons:
    b.destroy()

buttons.clear()
```

เมื่อเปลี่ยนหน้า จะลบปุ่ม Image เก่าออกก่อน

---

# 15. เลือก Image ครั้งละ 6 รูป

```python
current = images[page*6:page*6+6]
```

ถ้า:

```text
page = 0
```

จะได้:

```text
images[0:6]
```

ถ้า:

```text
page = 1
```

จะได้:

```text
images[6:12]
```

ดังนั้นแสดงครั้งละ:

```text
6 Images
```

---

# 16. วนสร้างปุ่ม Image

```python
for i, path in enumerate(current):
```

วน Image ทีละรูป

---

# 17. เปิด Image

```python
img = Image.open(path)
```

ใช้ PIL เปิด Image

---

# 18. ย่อ Image

```python
img.thumbnail((350, 260))
```

กำหนดขนาดสูงสุด:

```text
Width  = 350
Height = 260
```

โดยรักษา Aspect Ratio ของภาพไว้

---

# 19. แปลง Image สำหรับ Tkinter

```python
photo = ImageTk.PhotoImage(img)
```

Tkinter ไม่สามารถใช้ PIL Image โดยตรงได้

จึงต้องแปลงเป็น:

```text
ImageTk.PhotoImage
```

---

# 20. สร้างปุ่ม Image

```python
b = tk.Button(
    frame,
    image=photo,
    text=path.name,
    compound="top",
    command=lambda p=path: toggle(p),
    bg="lightgreen"
        if path.stem in selected
        else "SystemButtonFace"
)
```

ปุ่มประกอบด้วย:

* รูปภาพ
* ชื่อไฟล์
* Function เมื่อกด
* สีแสดงสถานะการเลือก

---

# 21. สีเขียวหมายถึงเลือกแล้ว

```python
bg="lightgreen"
    if path.stem in selected
    else "SystemButtonFace"
```

ถ้า Image ถูกเลือก:

```text
🟩 สีเขียว
```

ถ้ายังไม่ได้เลือก:

```text
สีปกติ
```

ทำให้รู้ได้ทันทีว่าภาพไหนถูกเลือกแล้ว

---

# 22. สำคัญ: เก็บ Reference ของ Image

```python
b.image = photo
```

บรรทัดนี้สำคัญกับ Tkinter

เพื่อป้องกัน Image ถูก Garbage Collection แล้วหายจาก Button

---

# 23. จัดตำแหน่ง Image

```python
b.grid(
    row=i//3,
    column=i%3,
    padx=10,
    pady=10
)
```

จัดเป็น Grid:

```text
┌─────────┬─────────┬─────────┐
│ Image 1 │ Image 2 │ Image 3 │
├─────────┼─────────┼─────────┤
│ Image 4 │ Image 5 │ Image 6 │
└─────────┴─────────┴─────────┘
```

สูตร:

```python
row = i // 3
column = i % 3
```

---

# 24. อัปเดตจำนวนที่เลือก

```python
count.config(
    text=f"เลือกแล้ว: {len(selected)} รูป"
)
```

ตัวอย่าง:

```text
เลือกแล้ว: 12 รูป
```

---

# 25. แสดงเลขหน้า

```python
page_label.config(
    text=f"หน้า {page+1}/{max(1,(len(images)+5)//6)}"
)
```

ถ้ามี 20 รูป:

```text
20 ÷ 6
```

ต้องมีทั้งหมด:

```text
4 หน้า
```

แสดง:

```text
หน้า 1/4
```

---

# 26. Function `next_page()`

```python
def next_page():
    global page

    if (page+1)*6 < len(images):
        page += 1
        show()
```

ใช้ไปหน้าถัดไป

เช่น:

```text
หน้า 1 → หน้า 2
```

แต่จะไม่ไปต่อถ้าเป็นหน้าสุดท้าย

---

# 27. Function `prev_page()`

```python
def prev_page():
    global page

    if page > 0:
        page -= 1
        show()
```

ใช้กลับไปหน้าก่อนหน้า

และป้องกันไม่ให้เลขหน้าเป็น:

```text
-1
```

---

# 28. Function `save()`

```python
def save():
```

ใช้สำหรับบันทึกภาพที่เลือก

---

# 29. ตรวจว่ายังไม่ได้เลือก

```python
if not selected:
    messagebox.showwarning(
        "แจ้งเตือน",
        "ยังไม่ได้เลือกรูป"
    )
    return
```

ถ้ายังไม่ได้เลือกภาพ:

```text
⚠ ยังไม่ได้เลือกรูป
```

และหยุดการบันทึก

---

# 30. กำหนดไฟล์ Output

```python
out = DATASET / "P711_Empty_Selection.txt"
```

ไฟล์ผลลัพธ์:

```text
P711_Empty_Selection.txt
```

อยู่ใน Dataset

---

# 31. บันทึก Path ของภาพ

```python
with open(out, "w", encoding="utf-8") as f:

    for p in images:

        if p.stem in selected:
            f.write(str(p) + "\n")
```

จะเขียน Path เต็มของ Image ที่เลือก

ตัวอย่าง:

```text
C:\Users\pawor\Desktop\P711Test\P711_010.jpg
C:\Users\pawor\Desktop\P711Test\P711_015.jpg
C:\Users\pawor\Desktop\P711Test\P711_023.jpg
```

หนึ่ง Image ต่อหนึ่งบรรทัด

---

# 32. แสดงข้อความเมื่อเสร็จ

```python
messagebox.showinfo(
    "เสร็จแล้ว",
    f"เลือกบล็อกเปล่า {len(selected)} รูป"
)
```

ตัวอย่าง:

```text
เสร็จแล้ว

เลือกบล็อกเปล่า 25 รูป
```

จากนั้น:

```python
root.destroy()
```

ปิด GUI

---

# 33. ปุ่มควบคุม

สร้างพื้นที่:

```python
control = tk.Frame(root)
control.pack(pady=10)
```

ประกอบด้วย 3 ส่วน:

```text
┌──────────────┐  ┌────────┐  ┌──────────────┐
│ ◀ ก่อนหน้า  │  │ หน้า 1 │  │ ถัดไป ▶      │
└──────────────┘  └────────┘  └──────────────┘
```

---

# 34. ปุ่มยืนยัน

```python
tk.Button(
    root,
    text="✓ ยืนยัน",
    command=save,
    font=("Arial", 14, "bold"),
    width=18
).pack(pady=10)
```

เมื่อกด:

```text
✓ ยืนยัน
```

จะเรียก:

```python
save()
```

เพื่อบันทึกข้อมูล

---

# 35. เริ่มแสดงภาพ

```python
show()
```

เรียก Function `show()` ครั้งแรก

เพื่อแสดงหน้าแรก

---

# 36. เริ่ม GUI

```python
root.mainloop()
```

ทำให้ Tkinter เริ่ม Event Loop

และรอการกระทำจากผู้ใช้ เช่น:

```text
คลิกภาพ
เปลี่ยนหน้า
กดยืนยัน
```

---

# 37. Flow การทำงานทั้งหมด

```text
P711Test
    │
    ├── ค้นหา .txt
    │      │
    │      └── มี Label แล้ว
    │
    ├── ค้นหา Images
    │      │
    │      └── ตัด Images ที่มี Label ออก
    │
    ▼
Images ที่ยังไม่มี Label
    │
    ▼
แสดง GUI
    │
    ├── หน้า 1
    │    ├── Image 1
    │    ├── Image 2
    │    ├── ...
    │    └── Image 6
    │
    ├── หน้า 2
    │
    └── ...
    │
    ▼
ผู้ใช้เลือก "บล็อกเปล่า"
    │
    ▼
selected = set()
    │
    ▼
กด ✓ ยืนยัน
    │
    ▼
P711_Empty_Selection.txt
```

---

# 38. ตัวอย่างผลลัพธ์

ถ้าผู้ใช้เลือก:

```text
P711_010.jpg
P711_015.jpg
P711_023.jpg
```

ไฟล์:

```text
P711_Empty_Selection.txt
```

จะมี:

```text
C:\Users\pawor\Desktop\P711Test\P711_010.jpg
C:\Users\pawor\Desktop\P711Test\P711_015.jpg
C:\Users\pawor\Desktop\P711Test\P711_023.jpg
```

---

# 39. จุดสำคัญของ STEP 2.5

โค้ดนี้ **ไม่ได้สร้าง YOLO Label**

แต่ทำหน้าที่:

> **คัดเลือกภาพที่ไม่มี Label แล้วระบุว่าภาพไหนเป็น Empty Image**

ดังนั้นไฟล์:

```text
P711_Empty_Selection.txt
```

เป็น **รายการภาพที่ผู้ใช้เลือก**

ไม่ใช่ YOLO Annotation Label

---

# 40. ความสัมพันธ์กับ STEP 2.0

### STEP 2.0

ตรวจสอบ Label:

```text
Image
   +
YOLO Label
   ↓
ตรวจ Bounding Box
```

### STEP 2.5

คัดเลือกภาพ Empty:

```text
Image ที่ไม่มี Label
   ↓
ผู้ใช้ตรวจด้วยตา
   ↓
เลือก Empty
   ↓
P711_Empty_Selection.txt
```

---

# 41. สรุป

```text
STEP 2.5
──────────────

หา Image
    ↓
ตัด Image ที่มี Label แล้ว
    ↓
แสดง 6 รูป / หน้า
    ↓
คลิกเลือก Empty Image
    ↓
เปลี่ยนสีเป็นเขียว
    ↓
กด ✓ ยืนยัน
    ↓
บันทึก P711_Empty_Selection.txt
```

**หน้าที่หลักของ STEP 2.5 คือ**

> **คัดเลือกภาพที่ไม่มี Object / เป็นบล็อกเปล่า เพื่อใช้เป็น Empty Images ใน Dataset**
