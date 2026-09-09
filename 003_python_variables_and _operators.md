Absolutely. Since you’ve finished **Python Data Types**, a good next step is **Python Variables & Operators**, then move into conditions and loops.

 Here’s the next `.md` file in the same style:

 Python Variables and Operators

# Python Variables and Operators

 ## Variables

 ตัวแปร (Variable) ใช้สำหรับเก็บข้อมูลไว้ในโปรแกรม

```
name = "Jessa"
age = 20
price = 35.75

print(name)
print(age)
print(price)
```

 Output:

```
Jessa
20
35.75
```

---

 ## Creating Variables

 Python ไม่จำเป็นต้องประกาศ Data Type ก่อนสร้างตัวแปร

```
name = "Jessa"
age = 20
is_student = True

print(name)
print(age)
print(is_student)
```

 Python จะรู้ Data Type จากค่าที่เราเก็บไว้

```
print(type(name))
print(type(age))
print(type(is_student))
```

 Output:

```
<class 'str'>
<class 'int'>
<class 'bool'>
```

---

 ## Variable Naming Rules

 ชื่อ Variable ควรปฏิบัติตามกฎของ Python:

 - ต้องขึ้นต้นด้วยตัวอักษร (`a-z`, `A-Z`) หรือ `_`
- ห้ามขึ้นต้นด้วยตัวเลข
- สามารถใช้ตัวเลขได้หลังจากตัวแรก
- ห้ามมีช่องว่าง
- ห้ามใช้ Python keywords

 ถูกต้อง:

```
name = "Jessa"
age2 = 20
_student = True
student_name = "Jessa"
```

 ไม่ถูกต้อง:

```
# 2age = 20
# student name = "Jessa"
# class = "Python"
```

---

 ## Multiple Variables

 สามารถกำหนดหลายตัวแปรในบรรทัดเดียวได้

```
name, age, score = "Jessa", 20, 95

print(name)
print(age)
print(score)
```

 Output:

```
Jessa
20
95
```

---

 ## Assigning the Same Value

 สามารถกำหนดค่าเดียวกันให้หลายตัวแปรได้

```
x = y = z = 10

print(x)
print(y)
print(z)
```

 Output:

```
10
10
10
```

---

 # Operators

 Operators คือเครื่องหมายที่ใช้ในการคำนวณหรือเปรียบเทียบข้อมูล

 Python มี Operators หลายประเภท:

 - Arithmetic Operators
- Comparison Operators
- Assignment Operators
- Logical Operators
- Membership Operators
- Identity Operators

---

 # Arithmetic Operators

 ใช้สำหรับการคำนวณทางคณิตศาสตร์

 | Operator | Name | Example |
| --- | --- | --- |
| `+` | Addition | `5 + 2` |
| `-` | Subtraction | `5 - 2` |
| `*` | Multiplication | `5 * 2` |
| `/` | Division | `5 / 2` |
| `//` | Floor Division | `5 // 2` |
| `%` | Modulus | `5 % 2` |
| `**` | Exponentiation | `5 ** 2` |

---

 ## Addition

```
a = 10
b = 5

print(a + b)
```

 Output:

```
15
```

---

 ## Subtraction

```
a = 10
b = 5

print(a - b)
```

 Output:

```
5
```

---

 ## Multiplication

```
a = 10
b = 5

print(a * b)
```

 Output:

```
50
```

---

 ## Division

```
a = 10
b = 3

print(a / b)
```

 Output:

```
3.3333333333333335
```

 `/` จะให้ผลลัพธ์เป็น `float`

---

 ## Floor Division

```
a = 10
b = 3

print(a // b)
```

 Output:

```
3
```

 `//` จะหารและปัดลงเป็นจำนวนเต็มตามกฎของ floor division

---

 ## Modulus

 `%` ใช้หาเศษจากการหาร

```
a = 10
b = 3

print(a % b)
```

 Output:

```
1
```

 เพราะ:

```
10 ÷ 3 = 3 เศษ 1
```

---

 ## Exponentiation

 `**` ใช้ยกกำลัง

```
number = 2

print(number ** 3)
```

 Output:

```
8
```

 เพราะ:

```
2 × 2 × 2 = 8
```

---

 # Comparison Operators

 ใช้เปรียบเทียบค่าต่าง ๆ

 ผลลัพธ์จะเป็น `True` หรือ `False`

 | Operator | Meaning |
| --- | --- |
| `==` | เท่ากัน |
| `!=` | ไม่เท่ากัน |
| `>` | มากกว่า |
| `<` | น้อยกว่า |
| `>=` | มากกว่าหรือเท่ากับ |
| `<=` | น้อยกว่าหรือเท่ากับ |

 ตัวอย่าง:

```
a = 10
b = 5

print(a == b)
print(a != b)
print(a > b)
print(a < b)
print(a >= b)
print(a <= b)
```

 Output:

```
False
True
True
False
True
False
```

---

 # Assignment Operators

 ใช้กำหนดหรือเปลี่ยนค่าของตัวแปร

```
x = 10

x += 5
print(x)
```

 Output:

```
15
```

 ตัวอย่าง Operators:

```
x = 10

x += 5    # x = x + 5
x -= 2    # x = x - 2
x *= 3    # x = x * 3
x /= 2    # x = x / 2
```

---

 # Logical Operators

 ใช้เชื่อมเงื่อนไขหลายเงื่อนไขเข้าด้วยกัน

 ## `and`

 ทุกเงื่อนไขต้องเป็น `True`

```
age = 20
has_id = True

print(age >= 18 and has_id)
```

 Output:

```
True
```

---

 ## `or`

 อย่างน้อยหนึ่งเงื่อนไขเป็น `True`

```
age = 20
is_student = False

print(age >= 18 or is_student)
```

 Output:

```
True
```

---

 ## `not`

 ใช้กลับค่าจาก `True` เป็น `False` หรือจาก `False` เป็น `True`

```
is_student = True

print(not is_student)
```

 Output:

```
False
```

---

 # Membership Operators

 ใช้ตรวจสอบว่าข้อมูลอยู่ใน Sequence หรือไม่

 มี 2 ตัว:

 - `in`
- `not in`

```
name = "Jessa"

print("J" in name)
print("x" in name)
```

 Output:

```
True
False
```

 ตัวอย่างกับ List:

```
numbers = [1, 2, 3, 4, 5]

print(3 in numbers)
print(10 in numbers)
```

 Output:

```
True
False
```

---

 # Identity Operators

 ใช้ตรวจสอบว่า Object เป็น Object เดียวกันหรือไม่

 มี 2 ตัว:

 - `is`
- `is not`

```
a = None

print(a is None)
print(a is not None)
```

 Output:

```
True
False
```

 > `is` ใช้ตรวจสอบ **identity** ของ object ส่วน `==` ใช้ตรวจสอบว่า **ค่าของ object เท่ากันหรือไม่**

---

 # Operator Precedence

 Python มีลำดับความสำคัญของ Operators

 ตัวอย่าง:

```
result = 10 + 2 * 3

print(result)
```

 Output:

```
16
```

 เพราะ Python คำนวณ `2 * 3` ก่อน:

```
10 + (2 * 3)
= 10 + 6
= 16
```

 สามารถใช้ `()` เพื่อกำหนดลำดับการคำนวณได้

```
result = (10 + 2) * 3

print(result)
```

 Output:

```
36
```

---

 # Type Conversion

 สามารถแปลง Data Type ได้

 ## Convert to Integer

```
number = int("20")

print(number)
print(type(number))
```

 Output:

```
20
<class 'int'>
```

---

 ## Convert to Float

```
number = float("20.5")

print(number)
print(type(number))
```

 Output:

```
20.5
<class 'float'>
```

---

 ## Convert to String

```
age = 20

text = str(age)

print(text)
print(type(text))
```

 Output:

```
20
<class 'str'>
```

---

 ## Convert to Boolean

```
print(bool(1))
print(bool(0))
```

 Output:

```
True
False
```

---

 # Summary

 | Topic | Example | Description |
| --- | --- | --- |
| Variable | `age = 20` | เก็บข้อมูล |
| `+` | `5 + 2` | บวก |
| `-` | `5 - 2` | ลบ |
| `*` | `5 * 2` | คูณ |
| `/` | `5 / 2` | หาร |
| `//` | `5 // 2` | หารปัดลง |
| `%` | `5 % 2` | หาเศษ |
| `**` | `5 ** 2` | ยกกำลัง |
| `==` | `5 == 5` | เปรียบเทียบ |
| `and` | `True and True` | และ |
| `or` | `True or False` | หรือ |
| `not` | `not True` | กลับค่า |
| `in` | `2 in [1, 2]` | ตรวจสอบสมาชิก |
| `is` | `x is None` | ตรวจสอบ identity |
| `int()` | `int("20")` | แปลงเป็น integer |
| `float()` | `float("20.5")` | แปลงเป็น float |
| `str()` | `str(20)` | แปลงเป็น string |
| `bool()` | `bool(1)` | แปลงเป็น boolean |

---

 # Next Lesson

 หลังจาก Variables และ Operators แล้ว แนะนำให้เรียนตามลำดับนี้:

 1. **Input & Output**
2. **if / elif / else**
3. **for Loop**
4. **while Loop**
5. **Functions**
6. **List Methods**
7. **Dictionary Methods**
8. **Set & Tuple Operations**
9. **String Methods**
10. **Exception Handling**
11. **File Handling**
12. **Modules & Packages**
13. **Object-Oriented Programming (OOP)**
14. **Virtual Environment & pip**
15. **Mini Projects**

 ลำดับถัดไปที่ผมแนะนำเป็นพิเศษคือ **`input()` \+ `if/elif/else`** เพราะสองเรื่องนี้จะทำให้เริ่มเขียนโปรแกรมที่ “โต้ตอบกับผู้ใช้และตัดสินใจเอง” ได้แล้ว เช่น โปรแกรมตรวจสอบอายุ, เครื่องคิดเลข และระบบ Login แบบง่าย ๆ.