# Python For Loop

## For Loop

`for` loop ใช้สำหรับสั่งให้โปรแกรมทำงานซ้ำกับข้อมูลหลาย ๆ ค่า

ตัวอย่าง:

```python
for i in range(5):
    print(i)
```

Output:

```text
0
1
2
3
4
```

---

## range()

`range()` ใช้สร้างลำดับตัวเลขสำหรับนำไปวนใน `for`

```python
for i in range(5):
    print(i)
```

`range(5)` จะได้:

```text
0
1
2
3
4
```

โดยค่าเริ่มต้นจะเริ่มจาก `0` และหยุดก่อนถึง `5`

---

## range(start, stop)

สามารถกำหนดจุดเริ่มต้นได้

```python
for i in range(1, 6):
    print(i)
```

Output:

```text
1
2
3
4
5
```

รูปแบบ:

```python
range(start, stop)
```

ค่า `stop` จะไม่ถูกรวมอยู่ในผลลัพธ์

---

## range(start, stop, step)

สามารถกำหนดจำนวนที่เพิ่มหรือลดในแต่ละรอบได้

```python
for i in range(0, 10, 2):
    print(i)
```

Output:

```text
0
2
4
6
8
```

ตัวอย่างนับถอยหลัง:

```python
for i in range(5, 0, -1):
    print(i)
```

Output:

```text
5
4
3
2
1
```

---

## Loop Through a List

`for` สามารถใช้วนดูสมาชิกใน `list` ได้

```python
fruits = ["apple", "banana", "orange"]

for fruit in fruits:
    print(fruit)
```

Output:

```text
apple
banana
orange
```

ตัวแปร `fruit` จะได้รับสมาชิกของ List ทีละตัวในแต่ละรอบ

---

## Loop Through a String

สามารถใช้ `for` วนดูตัวอักษรแต่ละตัวใน String ได้

```python
name = "Jessa"

for char in name:
    print(char)
```

Output:

```text
J
e
s
s
a
```

---

## For Loop with if

สามารถใช้ `if` ภายใน `for` ได้

```python
for i in range(1, 6):
    if i % 2 == 0:
        print(i)
```

Output:

```text
2
4
```

ตัวอย่างนี้ใช้ `%` เพื่อตรวจสอบว่าเลขเป็นเลขคู่หรือไม่

---

## Sum Numbers with For Loop

สามารถใช้ `for` เพื่อหาผลรวมของตัวเลขได้

```python
total = 0

for i in range(1, 6):
    total = total + i

print(total)
```

Output:

```text
15
```

เพราะ:

```text
1 + 2 + 3 + 4 + 5 = 15
```

---

## Multiplication Table

สามารถใช้ `for` สร้างตารางสูตรคูณได้

```python
number = 5

for i in range(1, 11):
    print(number, "x", i, "=", number * i)
```

Output:

```text
5 x 1 = 5
5 x 2 = 10
5 x 3 = 15
5 x 4 = 20
5 x 5 = 25
5 x 6 = 30
5 x 7 = 35
5 x 8 = 40
5 x 9 = 45
5 x 10 = 50
```

---

## Input with For Loop

สามารถรับจำนวนรอบจากผู้ใช้แล้วใช้ `for` ทำงานตามจำนวนที่กำหนดได้

```python
n = int(input("Enter a number: "))

for i in range(n):
    print("Hello")
```

ถ้ากรอก:

```text
3
```

Output:

```text
Hello
Hello
Hello
```

---

## Nested For Loop

สามารถใช้ `for` ซ้อนอยู่ภายใน `for` ได้

```python
for i in range(1, 4):
    for j in range(1, 4):
        print(i, j)
```

Output:

```text
1 1
1 2
1 3
2 1
2 2
2 3
3 1
3 2
3 3
```

Nested `for` เหมาะกับงานที่ต้องวนข้อมูลหลายระดับ

---

## break

`break` ใช้หยุด Loop ทันที

```python
for i in range(1, 10):
    if i == 5:
        break
    print(i)
```

Output:

```text
1
2
3
4
```

เมื่อ `i` มีค่าเป็น `5` โปรแกรมจะออกจาก Loop

---

## continue

`continue` ใช้ข้ามการทำงานในรอบปัจจุบัน แล้วไปยังรอบถัดไป

```python
for i in range(1, 6):
    if i == 3:
        continue
    print(i)
```

Output:

```text
1
2
4
5
```

รอบที่ `i == 3` จะไม่ทำคำสั่ง `print(i)`

---

## else with For Loop

Python สามารถใช้ `else` ร่วมกับ `for` ได้

```python
for i in range(3):
    print(i)
else:
    print("Loop finished")
```

Output:

```text
0
1
2
Loop finished
```

`else` จะทำงานเมื่อ Loop จบตามปกติ

---

## Common Mistakes

### ลืม Indentation

ผิด:

```python
# for i in range(5):
# print(i)
```

ถูก:

```python
for i in range(5):
    print(i)
```

คำสั่งภายใน `for` ต้องเยื้องเข้ามา โดยทั่วไปนิยมใช้ `4 spaces`

---

### เข้าใจ range() ผิด

```python
for i in range(5):
    print(i)
```

ไม่ได้แสดง `5`

Output:

```text
0
1
2
3
4
```

เพราะ `range(5)` หยุดก่อนถึง `5`

---

### ลืมเพิ่มค่าในการสะสม

ตัวอย่าง:

```python
total = 0

for i in range(1, 6):
    total = total + i

print(total)
```

ต้องกำหนด `total = 0` ก่อนเริ่ม Loop

---

# Combining List, For Loop, and if

สามารถนำสิ่งที่เรียนมาก่อนหน้านี้มารวมกันได้

```python
numbers = [10, 15, 20, 25, 30]

for number in numbers:
    if number % 2 == 0:
        print(number, "Even")
    else:
        print(number, "Odd")
```

Output:

```text
10 Even
15 Odd
20 Even
25 Odd
30 Even
```

โปรแกรมนี้ใช้:

- List
- `for`
- `if / else`
- `%`
- Variables
- `print()`

ทั้งหมดร่วมกัน

---

# Summary

| Topic | Example | Description |
| --- | --- | --- |
| `for` | `for i in range(5):` | วนทำงานซ้ำ |
| `range()` | `range(5)` | สร้างลำดับตัวเลข |
| `range(start, stop)` | `range(1, 6)` | กำหนดจุดเริ่มต้นและจุดสิ้นสุด |
| `range(start, stop, step)` | `range(0, 10, 2)` | กำหนดช่วงการเพิ่ม/ลด |
| List Loop | `for item in items:` | วนสมาชิกใน List |
| String Loop | `for char in text:` | วนตัวอักษรใน String |
| `break` | `break` | หยุด Loop |
| `continue` | `continue` | ข้ามรอบปัจจุบัน |
| Nested Loop | `for` ภายใน `for` | วน Loop หลายระดับ |
| `else` | `for ... else` | ทำงานเมื่อ Loop จบตามปกติ |

---

# Key Concepts

สิ่งสำคัญที่ควรจำ:

```text
for → วนทำงานซ้ำ
range() → สร้างลำดับตัวเลข
break → หยุด Loop
continue → ข้ามรอบปัจจุบัน
```

โครงสร้างพื้นฐาน:

```python
for variable in sequence:
    statement
```

ตัวอย่าง:

```python
for i in range(1, 6):
    print(i)
```

---

# Mini Exercises

## Exercise 1: Print 1–10

เขียนโปรแกรมใช้ `for` แสดงตัวเลข:

```text
1
2
3
4
5
6
7
8
9
10
```

---

## Exercise 2: Even Numbers

ใช้ `for` แสดงเลขคู่ตั้งแต่ `1` ถึง `20`

ตัวอย่าง Output:

```text
2
4
6
...
20
```

---

## Exercise 3: Sum Numbers

เขียนโปรแกรมหาผลรวมของตัวเลข `1` ถึง `100`

ผลลัพธ์ควรเป็น:

```text
5050
```

---

## Exercise 4: Multiplication Table

รับตัวเลขจากผู้ใช้ แล้วแสดงสูตรคูณตั้งแต่ `1` ถึง `12`

ตัวอย่าง:

```text
Enter number: 7

7 x 1 = 7
7 x 2 = 14
...
7 x 12 = 84
```

---

## Exercise 5: Find Even Numbers in a List

กำหนด:

```python
numbers = [3, 8, 12, 15, 21, 24, 30]
```

ใช้ `for` และ `if` แสดงเฉพาะเลขคู่

Expected Output:

```text
8
12
24
30
```

---

# Next Lesson

หลังจากเรียน `for Loop` แล้ว ขั้นต่อไปคือ:

1. **`while Loop`**
2. **Functions**
3. **List Methods**
4. **Dictionary Methods**
5. **Set & Tuple Operations**
6. **String Methods**
7. **Exception Handling**
8. **File Handling**
9. **Modules & Packages**
10. **Object-Oriented Programming (OOP)**
11. **Virtual Environment & pip**
12. **Mini Projects**

ลำดับถัดไปที่แนะนำคือ **`while Loop`** เพราะจะช่วยให้เข้าใจการวนซ้ำโดยใช้เงื่อนไข และต่อยอดไปสู่การสร้างโปรแกรมที่ทำงานซ้ำจนกว่าเงื่อนไขจะเปลี่ยน
