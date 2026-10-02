# WORK STEP 3 — Select Empty Images

## 1. Import Library

```python
from pathlib import Path
import tkinter as tk
from tkinter import messagebox
from PIL import Image, ImageTk
```

Step นี้ใช้ Library หลัก 3 ตัว:

### `Path`

```python
from pathlib import Path
```

ใช้จัดการ Path ของไฟล์และ Folder

---

### `tkinter`

```python
import tkinter as tk
from tkinter import messagebox
```

ใช้สร้าง **GUI (Graphical User Interface)**

เช่น:

* Window
* Button
* Label
* Frame
* Message Box

---

### `PIL`

```python
from PIL import Image, ImageTk
```

หรือเรียกว่า **Pillow**

ใช้สำหรับเปิดและเตรียมรูปภาพให้สามารถแสดงใน Tkinter ได้

---

# 2. กำหนด Dataset

```python
DATASET = Path(r"C:\Users\pawor\Desktop\P711Test")
```

กำหนดตำแหน่ง Dataset

```text
C:\Users\pawor\Desktop\P711Test
```

เก็บไว้ในตัวแปร:

```python
DATASET
```

---

# 3. หา Image ที่มี Label อยู่แล้ว

```python
labeled = {
    p.stem for p in DATASET.rglob("*.txt")
    if p.name != "P711_Empty_Selection.txt"
}
```

ส่วนนี้มีหน้าที่:

> หา Image ที่มี Label อยู่แล้ว เพื่อไม่ให้นำมาเลือกเป็น Empty Image

---

## `DATASET.rglob("*.txt")`

```python
DATASET.rglob("*.txt")
```

ค้นหาไฟล์ `.txt` ใน Dataset และ Subfolder ทั้งหมด

เช่น:

```text
P711Test/
├── 001.jpg
├── 001.txt
├── 002.jpg
├── 003.jpg
└── 003.txt
```

จะพบ:

```text
001.txt
003.txt
```

---

## `p.stem`

```python
p.stem
```

เอาชื่อไฟล์โดยไม่รวม Extension

ตัวอย่าง:

```text
001.txt → 001
003.txt → 003
```

ดังนั้น `labeled` จะมีลักษณะ:

```python
{"001", "003"}
```

---

# 4. ไม่เอา `P711_Empty_Selection.txt`

```python
if p.name != "P711_Empty_Selection.txt"
```

ตรวจชื่อไฟล์ก่อนเพิ่มเข้า `labeled`

ถ้าไฟล์ชื่อ:

```text
P711_Empty_Selection.txt
```

จะถูกยกเว้น

### ทำไมต้องยกเว้น?

เพราะไฟล์นี้เป็นไฟล์ที่ Step นี้สร้างขึ้นเพื่อเก็บรายการ Empty Images

ถ้าไม่ยกเว้น เมื่อเปิดโปรแกรมอีกครั้ง โปรแกรมอาจนำรายการ Empty ที่เคยเลือกไปตีความว่าเป็น Label ปกติ

---

# 5. หา Image ที่ยังไม่มี Label

```python
images = sorted([
    p for p in DATASET.rglob("*")
    if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    and p.stem not in labeled
])
```

ส่วนนี้ค้นหา Image ที่:

1. เป็น `.jpg`
2. หรือ `.jpeg`
3. หรือ `.png`
4. และยังไม่มี Label เดิม

---

## ตรวจ Extension

```python
p.suffix.lower() in [".jpg", ".jpeg", ".png"]
```

รับเฉพาะไฟล์รูป:

```text
.jpg
.jpeg
.png
```

---

## ตรวจว่าไม่มี Label

```python
p.stem not in labeled
```

ตัวอย่าง:

```text
001.jpg
001.txt
```

เนื่องจาก:

```python
p.stem
```

คือ:

```text
001
```

และ `001` อยู่ใน:

```python
labeled
```

ดังนั้น `001.jpg` จะไม่ถูกนำมาแสดง

---

## ผลลัพธ์

สมมติ Dataset มี:

```text
001.jpg
001.txt

002.jpg

003.jpg
003.txt

004.jpg
```

`images` จะเหลือ:

```text
002.jpg
004.jpg
```

เพราะเป็น Image ที่ยังไม่มี Label

---

# 6. สร้างตัวแปรสำหรับ Selection

```python
selected = set()
```

ใช้เก็บชื่อของ Image ที่ผู้ใช้เลือก

เริ่มต้นยังไม่ได้เลือก:

```python
set()
```

ถ้าเลือก:

```text
002.jpg
004.jpg
```

จะเก็บประมาณ:

```python
{"002", "004"}
```

ใช้ `set` เพราะต้องการตรวจสอบสมาชิกอย่างรวดเร็ว เช่น:

```python
path.stem in selected
```

---

# 7. กำหนดหน้าเริ่มต้น

```python
page = 0
```

กำหนดให้เริ่มที่หน้าแรก

Python ใช้ Index แบบเริ่มจาก `0`

ดังนั้น:

```text
page = 0 → หน้า 1
page = 1 → หน้า 2
page = 2 → หน้า 3
```

---

# 8. สร้างหน้าต่าง Tkinter

```python
root = tk.Tk()
```

สร้าง Main Window ของโปรแกรม

---

# 9. ตั้งชื่อ Window

```python
root.title("STEP 2.5 - Select Empty Images")
```

กำหนดชื่อบน Title Bar:

```text
STEP 2.5 - Select Empty Images
```

---

# 10. กำหนดขนาด Window

```python
root.geometry("1200x850")
```

กำหนดขนาดหน้าต่าง:

```text
กว้าง = 1200
สูง = 850
```

---

# 11. สร้างหัวข้อ

```python
tk.Label(
    root,
    text="เลือกภาพบล็อกเปล่า",
    font=("Arial", 22, "bold")
).pack(pady=10)
```

สร้าง Label สำหรับแสดงข้อความ:

```text
เลือกภาพบล็อกเปล่า
```

ใช้ Font:

```text
Arial
22
Bold
```

---

# 12. สร้างตัวนับจำนวนรูป

```python
count = tk.Label(
    root,
    text="เลือกแล้ว: 0 รูป",
    font=("Arial", 14, "bold")
)
```

แสดงจำนวนรูปที่เลือก

เริ่มต้น:

```text
เลือกแล้ว: 0 รูป
```

---

## `.pack()`

```python
count.pack()
```

นำ Label ไปวางใน Window

---

# 13. สร้าง Frame สำหรับรูปภาพ

```python
frame = tk.Frame(root)
frame.pack(expand=True)
```

`Frame` เป็นพื้นที่สำหรับใส่ Button รูปภาพ

โดย Button แต่ละตัวจะถูกวางใน `frame`

---

# 14. สร้าง List สำหรับเก็บ Button

```python
buttons = []
```

ใช้เก็บ Button ที่กำลังแสดงอยู่

เหตุผลที่ต้องเก็บไว้ เพราะเมื่อเปลี่ยนหน้า เราต้องลบ Button เก่าออกก่อน

---

# 15. Function `toggle()`

```python
def toggle(path):
    if path.stem in selected:
        selected.remove(path.stem)
    else:
        selected.add(path.stem)
    show()
```

Function นี้ทำหน้าที่:

> เลือก / ยกเลิกการเลือก Image

---

## ถ้าเลือกอยู่แล้ว

```python
if path.stem in selected:
```

ตรวจว่า Image นี้อยู่ใน `selected` หรือไม่

ถ้าอยู่:

```python
selected.remove(path.stem)
```

หมายถึง:

> ยกเลิกการเลือก

---

## ถ้ายังไม่ได้เลือก

```python
else:
    selected.add(path.stem)
```

เพิ่ม Image เข้า `selected`

เช่น:

```text
ก่อน:
selected = {}

กด 001.jpg

หลัง:
selected = {"001"}
```

---

## เรียก `show()`

```python
show()
```

หลังจากเลือกหรือยกเลิก จะสร้างหน้าจอใหม่เพื่ออัปเดตสีของ Button

---

# 16. Function `show()`

```python
def show():
```

Function นี้เป็นส่วนหลักของ GUI

หน้าที่คือ:

> แสดง Image ของหน้าปัจจุบัน พร้อมสถานะว่า Image ไหนถูกเลือก

---

# 17. ประกาศ `buttons` เป็น Global

```python
global buttons
```

บอกว่า Function นี้จะใช้ตัวแปร `buttons` ที่อยู่นอก Function

เพราะต้องแก้ไข List นี้

---

# 18. ลบ Button เก่า

```python
for b in buttons:
    b.destroy()
buttons.clear()
```

ก่อนสร้างหน้าปัจจุบัน ต้องลบ Button จากหน้าก่อนออกก่อน

```python
b.destroy()
```

ลบ Button ออกจาก GUI

จากนั้น:

```python
buttons.clear()
```

ล้างรายการ Button ใน Python

---

# 19. เลือก Image ของหน้าปัจจุบัน

```python
current = images[page*6:page*6+6]
```

แบ่ง Image ทีละ 6 รูป

สูตร:

```text
เริ่ม = page × 6
จบ = page × 6 + 6
```

---

### หน้า 1

```text
page = 0
```

จะเป็น:

```python
images[0:6]
```

---

### หน้า 2

```text
page = 1
```

จะเป็น:

```python
images[6:12]
```

---

### หน้า 3

```text
page = 2
```

จะเป็น:

```python
images[12:18]
```

ดังนั้น 1 หน้าแสดงสูงสุด 6 รูป

---

# 20. วนสร้าง Button ให้แต่ละ Image

```python
for i, path in enumerate(current):
```

`enumerate()` ทำให้ได้:

```text
i
path
```

ตัวอย่าง:

```text
i = 0 → 001.jpg
i = 1 → 002.jpg
i = 2 → 003.jpg
```

---

# 21. เปิด Image

```python
img = Image.open(path)
```

ใช้ Pillow เปิด Image

---

# 22. ย่อ Image

```python
img.thumbnail((350, 260))
```

กำหนดขนาดสูงสุด:

```text
กว้าง = 350
สูง = 260
```

`thumbnail()` จะรักษา Aspect Ratio ของรูป

ไม่ได้บังคับให้รูปทุกภาพเป็น 350×260 เสมอไป

แต่จะไม่เกินขนาดดังกล่าว

---

# 23. แปลง Image สำหรับ Tkinter

```python
photo = ImageTk.PhotoImage(img)
```

Tkinter ไม่สามารถใช้ PIL Image โดยตรงได้

จึงต้องแปลงเป็น:

```python
ImageTk.PhotoImage
```

ก่อนนำไปแสดงบน Button

---

# 24. สร้าง Button

```python
b = tk.Button(
    frame,
    image=photo,
    text=path.name,
    compound="top",
    command=lambda p=path: toggle(p),
    bg="lightgreen" if path.stem in selected else "SystemButtonFace"
)
```

สร้าง Button สำหรับแต่ละ Image

---

## `frame`

```python
frame
```

บอกว่า Button นี้จะอยู่ใน Frame ไหน

---

## `image=photo`

```python
image=photo
```

แสดงรูปบน Button

---

## `text=path.name`

```python
text=path.name
```

แสดงชื่อไฟล์

เช่น:

```text
001.jpg
```

---

## `compound="top"`

```python
compound="top"
```

กำหนดให้ Text อยู่ด้านบนของ Image

โครงสร้างประมาณ:

```text
001.jpg
   ↓
┌─────────────┐
│             │
│    IMAGE    │
│             │
└─────────────┘
```

---

# 25. กำหนดสิ่งที่จะเกิดขึ้นเมื่อกด Button

```python
command=lambda p=path: toggle(p)
```

เมื่อกด Image Button:

```text
Button
   ↓
toggle(path)
   ↓
เลือก / ยกเลิกเลือก
   ↓
show()
   ↓
อัปเดตหน้าจอ
```

### ทำไมใช้ `lambda p=path`?

เพื่อเก็บ `path` ของ Image แต่ละ Button เอาไว้

ทำให้เมื่อกด Button ตัวใด จะรู้ว่ากำลังกด Image ไหน

---

# 26. เปลี่ยนสี Button เมื่อถูกเลือก

```python
bg="lightgreen" if path.stem in selected else "SystemButtonFace"
```

เป็น Conditional Expression

ความหมายคือ:

```text
ถ้า Image ถูกเลือก
    ↓
lightgreen

ถ้ายังไม่ถูกเลือก
    ↓
SystemButtonFace
```

ดังนั้น Image ที่เลือกจะมีพื้นหลังสีเขียว

ช่วยให้ผู้ใช้รู้ว่าเลือกภาพไหนไปแล้ว

---

# 27. เก็บ Reference ของ Image

```python
b.image = photo
```

บรรทัดนี้สำคัญ

เก็บ `photo` ไว้กับ Button

เพื่อไม่ให้ Python ลบ Image ที่กำลังใช้งานออกจาก Memory

---

# 28. วาง Button เป็น Grid

```python
b.grid(
    row=i//3,
    column=i%3,
    padx=10,
    pady=10
)
```

จัด Image เป็น:

```text
3 columns
```

โดยใช้:

```python
i//3
```

หา Row

และ:

```python
i%3
```

หา Column

---

### ตัวอย่าง

```text
i = 0
row = 0
column = 0

i = 1
row = 0
column = 1

i = 2
row = 0
column = 2

i = 3
row = 1
column = 0

i = 4
row = 1
column = 1

i = 5
row = 1
column = 2
```

จึงได้:

```text
┌─────┬─────┬─────┐
│  0  │  1  │  2  │
├─────┼─────┼─────┤
│  3  │  4  │  5  │
└─────┴─────┴─────┘
```

---

# 29. เก็บ Button ลง List

```python
buttons.append(b)
```

เก็บ Button ที่สร้างไว้ใน:

```python
buttons
```

เพื่อใช้ลบในครั้งต่อไปเมื่อเปลี่ยนหน้า

---

# 30. อัปเดตจำนวนรูปที่เลือก

```python
count.config(text=f"เลือกแล้ว: {len(selected)} รูป")
```

`len(selected)` นับจำนวน Image ที่เลือก

เช่น:

```python
selected = {"001", "003", "007"}
```

จะได้:

```text
len(selected) = 3
```

หน้าจอจะแสดง:

```text
เลือกแล้ว: 3 รูป
```

---

# 31. แสดงหมายเลขหน้า

```python
page_label.config(
    text=f"หน้า {page+1}/{max(1,(len(images)+5)//6)}"
)
```

ใช้แสดง:

```text
หน้า 1/5
หน้า 2/5
หน้า 3/5
...
```

---

## ทำไมต้อง `page+1`?

เพราะ Python เริ่มนับจาก `0`

แต่ผู้ใช้ต้องการเห็น:

```text
หน้า 1
```

ดังนั้น:

```python
page = 0
```

จะแสดง:

```python
page + 1
```

เป็น:

```text
1
```

---

## คำนวณจำนวนหน้าทั้งหมด

```python
(len(images)+5)//6
```

ใช้คำนวณว่า Image ทั้งหมดต้องแบ่งกี่หน้า

เช่น:

```text
1–6 รูป   → 1 หน้า
7–12 รูป  → 2 หน้า
13–18 รูป → 3 หน้า
```

การบวก `5` ก่อนหารด้วย `6` ทำให้การหารจำนวนเต็มสามารถปัดขึ้นได้

ตัวอย่าง:

```text
10 รูป

(10 + 5) // 6
15 // 6
= 2
```

ดังนั้น 10 รูป = 2 หน้า

---

## `max(1, ...)`

```python
max(1, ...)
```

ป้องกันไม่ให้จำนวนหน้าเป็น `0`

ถ้าไม่มี Image:

```text
จำนวนหน้าอย่างน้อย = 1
```

---

# 32. Function `next_page()`

```python
def next_page():
    global page
    if (page+1)*6 < len(images):
        page += 1
        show()
```

ใช้สำหรับปุ่ม:

```text
ถัดไป ▶
```

---

## ตรวจว่ามีหน้าถัดไปหรือไม่

```python
if (page+1)*6 < len(images):
```

ถ้ายังมี Image เหลืออยู่ จึงอนุญาตให้ไปหน้าถัดไป

---

## เพิ่มเลขหน้า

```python
page += 1
```

เท่ากับ:

```python
page = page + 1
```

---

## แสดงหน้าใหม่

```python
show()
```

โหลด Image ของหน้าใหม่

---

# 33. Function `prev_page()`

```python
def prev_page():
    global page
    if page > 0:
        page -= 1
        show()
```

ใช้สำหรับปุ่ม:

```text
◀ ก่อนหน้า
```

---

## ตรวจว่าไม่ใช่หน้าแรก

```python
if page > 0:
```

ถ้า:

```text
page = 0
```

จะไม่สามารถย้อนกลับได้

---

## ลดเลขหน้า

```python
page -= 1
```

เท่ากับ:

```python
page = page - 1
```

จากนั้น:

```python
show()
```

แสดงหน้าใหม่

---

# 34. Function `save()`

```python
def save():
```

ทำงานเมื่อผู้ใช้กด:

```text
✓ ยืนยัน
```

หน้าที่คือ:

> บันทึกรายการ Image ที่เลือกลงไฟล์ `P711_Empty_Selection.txt`

---

# 35. ตรวจว่ายังไม่ได้เลือก Image

```python
if not selected:
```

ถ้า:

```python
selected = set()
```

หมายถึงยังไม่ได้เลือกอะไร

---

# 36. แสดง Warning

```python
messagebox.showwarning(
    "แจ้งเตือน",
    "ยังไม่ได้เลือกรูป"
)
```

แสดงกล่องแจ้งเตือน:

```text
แจ้งเตือน

ยังไม่ได้เลือกรูป
```

แล้ว Function จะจบโดยไม่บันทึก

---

# 37. กำหนด Output File

```python
out = DATASET / "P711_Empty_Selection.txt"
```

กำหนดไฟล์ที่จะเก็บรายการ Empty Image

Path จะเป็น:

```text
C:\Users\pawor\Desktop\P711Test\P711_Empty_Selection.txt
```

เครื่องหมาย:

```python
/
```

ของ `Path` ใช้สำหรับต่อ Path

---

# 38. เปิดไฟล์สำหรับเขียน

```python
with open(out, "w", encoding="utf-8") as f:
```

เปิดไฟล์ในโหมด:

```text
w = write
```

หมายถึงเขียนข้อมูลลงไฟล์

ถ้าไฟล์มีอยู่แล้ว จะเขียนทับเนื้อหาเดิม

---

# 39. วน Image ทั้งหมด

```python
for p in images:
```

ตรวจสอบ Image ทุกตัวที่ผ่านการกรองใน Step ก่อนหน้า

---

# 40. เลือกเฉพาะ Image ที่ผู้ใช้เลือก

```python
if p.stem in selected:
```

ถ้า Image นี้อยู่ใน `selected`

จึงบันทึกลงไฟล์

---

# 41. เขียน Path ลงไฟล์

```python
f.write(str(p) + "\n")
```

เขียน Path เต็มของ Image

เช่น:

```text
C:\Users\pawor\Desktop\P711Test\002.jpg
C:\Users\pawor\Desktop\P711Test\004.jpg
```

`\n` หมายถึงขึ้นบรรทัดใหม่

---

# 42. แสดงข้อความเมื่อบันทึกเสร็จ

```python
messagebox.showinfo(
    "เสร็จแล้ว",
    f"เลือกบล็อกเปล่า {len(selected)} รูป"
)
```

แสดงจำนวน Image ที่เลือก

เช่น:

```text
เสร็จแล้ว

เลือกบล็อกเปล่า 15 รูป
```

---

# 43. ปิดโปรแกรม

```python
root.destroy()
```

ปิด Main Window

จบการทำงานของ GUI

---

# 44. สร้าง Control Frame

```python
control = tk.Frame(root)
control.pack(pady=10)
```

สร้างพื้นที่สำหรับปุ่มควบคุม:

```text
◀ ก่อนหน้า
หน้า 1/5
ถัดไป ▶
```

---

# 45. ปุ่มก่อนหน้า

```python
tk.Button(
    control,
    text="◀ ก่อนหน้า",
    command=prev_page,
    width=12
).grid(row=0, column=0, padx=10)
```

สร้างปุ่ม:

```text
◀ ก่อนหน้า
```

เมื่อกด:

```python
prev_page()
```

---

# 46. Label แสดงหน้า

```python
page_label = tk.Label(
    control,
    text="หน้า 1",
    width=12
)
```

ใช้แสดงเลขหน้า

---

# 47. ปุ่มถัดไป

```python
tk.Button(
    control,
    text="ถัดไป ▶",
    command=next_page,
    width=12
).grid(row=0, column=2, padx=10)
```

เมื่อกด:

```python
next_page()
```

จะเปลี่ยนไปหน้าถัดไป

---

# 48. ปุ่มยืนยัน

```python
tk.Button(
    root,
    text="✓ ยืนยัน",
    command=save,
    font=("Arial", 14, "bold"),
    width=18
).pack(pady=10)
```

สร้างปุ่ม:

```text
✓ ยืนยัน
```

เมื่อกดจะเรียก:

```python
save()
```

เพื่อบันทึกรายการ Image ที่เลือก

---

# 49. แสดงหน้าแรก

```python
show()
```

เรียก Function `show()` ครั้งแรก

เพื่อโหลด Image หน้าแรกขึ้นมา

---

# 50. เริ่ม Tkinter Event Loop

```python
root.mainloop()
```

เริ่มการทำงานของ GUI

โปรแกรมจะรอ User:

```text
คลิก Image
     ↓
toggle()

กด ถัดไป
     ↓
next_page()

กด ก่อนหน้า
     ↓
prev_page()

กด ยืนยัน
     ↓
save()
```

โปรแกรมจะทำงานต่อไปจนกว่า Window จะถูกปิด

---

# สรุปการทำงานของ WORK STEP 3

```text
Dataset
   │
   ▼
ค้นหา .txt
   │
   ▼
สร้าง labeled
(รูปที่มี Label แล้ว)
   │
   ▼
ค้นหา Image
   │
   ▼
ตัด Image ที่มี Label ออก
   │
   ▼
เหลือ Image ที่ยังไม่มี Label
   │
   ▼
เปิด GUI
   │
   ▼
แสดง Image 6 รูป / หน้า
   │
   ▼
User คลิก Image
   │
   ▼
toggle()
   │
   ├── เลือก → เพิ่มเข้า selected
   │
   └── ยกเลิก → remove จาก selected
   │
   ▼
แสดงสีเขียวเพื่อบอกว่าเลือกแล้ว
   │
   ▼
กด "ยืนยัน"
   │
   ▼
สร้าง P711_Empty_Selection.txt
   │
   ▼
เขียน Path ของ Image ที่เลือก
```

# เป้าหมายของ Step 3

**คัดเลือกภาพที่เป็น "บล็อกเปล่า" (Empty Images) จากภาพที่ยังไม่มี Label แล้วบันทึกรายการภาพที่เลือกไว้ใน `P711_Empty_Selection.txt`**

ไฟล์ผลลัพธ์จะมีลักษณะ:

```text
C:\Users\pawor\Desktop\P711Test\image001.jpg
C:\Users\pawor\Desktop\P711Test\image007.jpg
C:\Users\pawor\Desktop\P711Test\image015.jpg
```

โดย **1 บรรทัด = 1 Image ที่เลือกเป็น Empty Image**
