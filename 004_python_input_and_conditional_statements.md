ได้เลย — ต่อจาก `003_python_variables_and_operators.md` ผมจัดให้เป็น `004_python_input_and_conditional_statements.md` ในสไตล์เดียวกัน โดยรวม `input()` \+ `if / elif / else` ตามลำดับที่แนะนำไว้

 004\_python\_input\_and\_conditional\_statements.md

# Python Input and Conditional Statements

 ## Input

 `input()` ใช้สำหรับรับข้อมูลจากผู้ใช้ผ่านทาง Keyboard

 ตัวอย่าง:

```
name = input("Enter your name: ")

print(name)
```

 ถ้าผู้ใช้กรอก:

```
Jessa
```

 Output:

```
Jessa
```

---

 ## Input Prompt

 เราสามารถใส่ข้อความเพื่อบอกผู้ใช้ว่าต้องกรอกอะไร

```
name = input("What is your name? ")

print("Hello", name)
```

 ตัวอย่าง Output:

```
What is your name? Jessa
Hello Jessa
```

---

 ## Input Always Returns a String

 ข้อมูลที่ได้จาก `input()` จะเป็น `str` เสมอ

```
age = input("Enter your age: ")

print(age)
print(type(age))
```

 ถ้ากรอก:

```
20
```

 Output:

```
20
<class 'str'>
```

 ถึงแม้ผู้ใช้จะกรอก `20` แต่ Python จะมองว่าเป็น String:

```
"20"
```

---

 ## Converting Input to Integer

 ถ้าต้องการรับตัวเลขเพื่อนำไปคำนวณ สามารถใช้ `int()` แปลงข้อมูลได้

```
age = int(input("Enter your age: "))

print(age)
print(type(age))
```

 ถ้ากรอก:

```
20
```

 Output:

```
20
<class 'int'>
```

---

 ## Converting Input to Float

 ถ้าต้องการรับเลขทศนิยม สามารถใช้ `float()`

```
price = float(input("Enter price: "))

print(price)
print(type(price))
```

 ถ้ากรอก:

```
35.75
```

 Output:

```
35.75
<class 'float'>
```

---

 # Conditional Statements

 Conditional Statements ใช้สำหรับให้โปรแกรม **ตัดสินใจตามเงื่อนไข**

 Python มีคำสั่งหลัก ๆ ดังนี้:

 - `if`
- `elif`
- `else`

 ตัวอย่างง่าย ๆ:

```
age = 20

if age >= 18:
    print("Adult")
```

 Output:

```
Adult
```

---

 # if Statement

 `if` ใช้ตรวจสอบว่าเงื่อนไขเป็น `True` หรือไม่

 ถ้าเงื่อนไขเป็น `True` Python จะทำงานใน Block ของ `if`

```
age = 20

if age >= 18:
    print("You are an adult")
```

 Output:

```
You are an adult
```

 ถ้าเงื่อนไขเป็น `False` โปรแกรมจะไม่ทำคำสั่งใน `if`

```
age = 15

if age >= 18:
    print("You are an adult")
```

 ไม่มี Output เพราะ:

```
15 >= 18
```

 เป็น `False`

---

 # Indentation

 Python ใช้ **Indentation** เพื่อกำหนด Block ของ Code

 ตัวอย่าง:

```
age = 20

if age >= 18:
    print("Adult")
    print("You can vote")
```

 คำสั่งที่อยู่ภายใต้ `if` ต้องเยื้องเข้ามา

```
if condition:
    statement
    statement
```

 โดยทั่วไปนิยมใช้ **4 spaces**

 ถูกต้อง:

```
if age >= 18:
    print("Adult")
```

 ไม่ถูกต้อง:

```
# if age >= 18:
# print("Adult")
```

---

 # else Statement

 `else` ใช้กำหนดสิ่งที่จะเกิดขึ้นเมื่อเงื่อนไขของ `if` เป็น `False`

```
age = 15

if age >= 18:
    print("Adult")
else:
    print("Minor")
```

 Output:

```
Minor
```

 อีกตัวอย่าง:

```
number = 10

if number > 0:
    print("Positive")
else:
    print("Zero or Negative")
```

 Output:

```
Positive
```

---

 # if / else

 สามารถใช้ `if` และ `else` เพื่อสร้างการตัดสินใจแบบ 2 ทาง

```
password = "1234"

if password == "1234":
    print("Login successful")
else:
    print("Invalid password")
```

 Output:

```
Login successful
```

 ถ้าเปลี่ยน Password:

```
password = "9999"

if password == "1234":
    print("Login successful")
else:
    print("Invalid password")
```

 Output:

```
Invalid password
```

---

 # elif Statement

 `elif` ย่อมาจาก **else if**

 ใช้เมื่อต้องการตรวจสอบหลายเงื่อนไข

```
score = 80

if score >= 90:
    print("A")
elif score >= 80:
    print("B")
elif score >= 70:
    print("C")
else:
    print("F")
```

 Output:

```
B
```

 Python จะตรวจสอบจากบนลงล่าง และเมื่อพบเงื่อนไขที่เป็น `True` แล้ว จะไม่ตรวจสอบ `elif` ที่เหลือต่อ

---

 # Multiple Conditions

 สามารถใช้ `if`, `elif` และ `else` หลายตัวได้

```
age = 25

if age < 13:
    print("Child")
elif age < 20:
    print("Teenager")
elif age < 60:
    print("Adult")
else:
    print("Senior")
```

 Output:

```
Adult
```

---

 # Comparison Operators with Conditions

 Conditional Statements มักใช้ร่วมกับ Comparison Operators

```
age = 20

print(age == 20)
print(age != 18)
print(age > 18)
print(age < 30)
print(age >= 20)
print(age <= 20)
```

 Output:

```
True
True
True
True
True
True
```

 สามารถนำ Operators เหล่านี้ไปใช้กับ `if`

```
age = 20

if age >= 18:
    print("You are an adult")
```

---

 # Logical Operators with Conditions

 สามารถใช้ `and`, `or` และ `not` ร่วมกับ `if`

 ## `and`

 ทุกเงื่อนไขต้องเป็น `True`

```
age = 20
has_id = True

if age >= 18 and has_id:
    print("Access granted")
```

 Output:

```
Access granted
```

---

 ## `or`

 อย่างน้อยหนึ่งเงื่อนไขต้องเป็น `True`

```
is_student = True
is_employee = False

if is_student or is_employee:
    print("You have access")
```

 Output:

```
You have access
```

---

 ## `not`

 ใช้กลับค่าของ Boolean

```
is_banned = False

if not is_banned:
    print("Welcome")
```

 Output:

```
Welcome
```

---

 # Nested if

 สามารถเขียน `if` ซ้อนอยู่ภายใน `if` ได้

```
age = 20
has_id = True

if age >= 18:
    if has_id:
        print("Access granted")
```

 Output:

```
Access granted
```

 Nested `if` เหมาะสำหรับกรณีที่มีเงื่อนไขเป็นลำดับชั้น

---

 # Input with if

 สามารถนำ `input()` มาใช้ร่วมกับ Conditional Statements ได้

 ตัวอย่างโปรแกรมตรวจสอบอายุ:

```
age = int(input("Enter your age: "))

if age >= 18:
    print("You are an adult")
else:
    print("You are a minor")
```

 ตัวอย่างการทำงาน:

```
Enter your age: 20
You are an adult
```

---

 # Age Checker

 ตัวอย่างโปรแกรมตรวจสอบช่วงอายุ

```
age = int(input("Enter your age: "))

if age < 13:
    print("Child")
elif age < 20:
    print("Teenager")
elif age < 60:
    print("Adult")
else:
    print("Senior")
```

 ตัวอย่าง:

```
Enter your age: 25
Adult
```

---

 # Simple Calculator

 สามารถใช้ `input()` และ `if/elif/else` สร้างเครื่องคิดเลขแบบง่ายได้

```
a = float(input("Enter first number: "))
operator = input("Enter operator (+, -, *, /): ")
b = float(input("Enter second number: "))

if operator == "+":
    print(a + b)
elif operator == "-":
    print(a - b)
elif operator == "*":
    print(a * b)
elif operator == "/":
    print(a / b)
else:
    print("Invalid operator")
```

 ตัวอย่าง:

```
Enter first number: 10
Enter operator (+, -, *, /): *
Enter second number: 5
50.0
```

---

 # Simple Login

 สามารถใช้ Conditional Statements ทำระบบ Login แบบง่าย ๆ

```
username = input("Username: ")
password = input("Password: ")

if username == "admin" and password == "1234":
    print("Login successful")
else:
    print("Invalid username or password")
```

 ตัวอย่าง:

```
Username: admin
Password: 1234
Login successful
```

---

 # Checking Even and Odd Numbers

 สามารถใช้ `%` ร่วมกับ `if` เพื่อตรวจสอบเลขคู่และเลขคี่

```
number = int(input("Enter a number: "))

if number % 2 == 0:
    print("Even")
else:
    print("Odd")
```

 ถ้ากรอก:

```
10
```

 Output:

```
Even
```

 เพราะ:

```
10 % 2 = 0
```

 ถ้ากรอก:

```
7
```

 Output:

```
Odd
```

 เพราะ:

```
7 % 2 = 1
```

---

 # Checking Positive, Negative, or Zero

 สามารถใช้ `if / elif / else` ตรวจสอบค่าของตัวเลขได้

```
number = int(input("Enter a number: "))

if number > 0:
    print("Positive")
elif number < 0:
    print("Negative")
else:
    print("Zero")
```

 ตัวอย่าง:

```
Enter a number: -5
Negative
```

---

 # Conditional Expression

 Python มีวิธีเขียน `if/else` แบบสั้น ๆ เรียกว่า **Conditional Expression**

 ตัวอย่าง:

```
age = 20

message = "Adult" if age >= 18 else "Minor"

print(message)
```

 Output:

```
Adult
```

 รูปแบบ:

```
value_if_true if condition else value_if_false
```

 เหมาะกับเงื่อนไขสั้น ๆ ที่อ่านง่าย

---

 # Common Mistakes

 ## ลืม `:`

 ผิด:

```
# if age >= 18
#     print("Adult")
```

 ถูก:

```
if age >= 18:
    print("Adult")
```

---

 ## Indentation ไม่ถูกต้อง

 ผิด:

```
# if age >= 18:
# print("Adult")
```

 ถูก:

```
if age >= 18:
    print("Adult")
```

---

 ## เปรียบเทียบด้วย `=` แทน `==`

 `=` ใช้สำหรับ Assign ค่า

```
age = 20
```

 `==` ใช้สำหรับเปรียบเทียบ

```
if age == 20:
    print("Age is 20")
```

---

 ## ลืมแปลง Input เป็นตัวเลข

 ตัวอย่างที่ควรระวัง:

```
age = input("Enter your age: ")

# if age >= 18:
#     print("Adult")
```

 `input()` คืนค่าเป็น `str` ดังนั้นควรแปลงเป็น `int`

```
age = int(input("Enter your age: "))

if age >= 18:
    print("Adult")
```

---

 # Combining Input, Variables, Operators, and Conditions

 ตอนนี้เราสามารถนำหลายเรื่องที่เรียนมาก่อนหน้านี้มารวมกันได้

```
name = input("Enter your name: ")
age = int(input("Enter your age: "))
score = float(input("Enter your score: "))

if score >= 80 and age >= 18:
    result = "Passed"
else:
    result = "Not passed"

print("Name:", name)
print("Age:", age)
print("Score:", score)
print("Result:", result)
```

 ตัวอย่าง:

```
Enter your name: Jessa
Enter your age: 20
Enter your score: 85
Name: Jessa
Age: 20
Score: 85.0
Result: Passed
```

 โปรแกรมนี้ใช้:

 - Variables
- `input()`
- Type Conversion
- Comparison Operators
- Logical Operators
- `if / else`
- `print()`

 ทั้งหมดร่วมกัน

---

 # Summary

 | Topic | Example | Description |
| --- | --- | --- |
| `input()` | `input("Name: ")` | รับข้อมูลจากผู้ใช้ |
| `int()` | `int("20")` | แปลงเป็น integer |
| `float()` | `float("20.5")` | แปลงเป็น float |
| `if` | `if age >= 18:` | ตรวจสอบเงื่อนไข |
| `elif` | `elif age >= 13:` | ตรวจสอบเงื่อนไขเพิ่มเติม |
| `else` | `else:` | ทำงานเมื่อเงื่อนไขเป็น False |
| `==` | `age == 20` | ตรวจสอบว่าเท่ากัน |
| `!=` | `age != 20` | ตรวจสอบว่าไม่เท่ากัน |
| `>` | `age > 18` | มากกว่า |
| `<` | `age < 18` | น้อยกว่า |
| `>=` | `age >= 18` | มากกว่าหรือเท่ากับ |
| `<=` | `age <= 18` | น้อยกว่าหรือเท่ากับ |
| `and` | `age >= 18 and has_id` | ทุกเงื่อนไขต้องเป็น True |
| `or` | `is_student or is_employee` | อย่างน้อยหนึ่งเงื่อนไขเป็น True |
| `not` | `not is_banned` | กลับค่า Boolean |
| `%` | `number % 2` | ใช้ตรวจสอบเลขคู่/เลขคี่ |

---

 # Key Concepts

 สิ่งสำคัญที่ควรจำ:

```
input() → รับข้อมูลจากผู้ใช้
int() / float() → แปลงข้อมูลเป็นตัวเลข
if → ตรวจสอบเงื่อนไข
elif → ตรวจสอบเงื่อนไขเพิ่มเติม
else → ทำงานเมื่อไม่มีเงื่อนไขใดเป็น True
```

 ตัวอย่างโครงสร้างพื้นฐาน:

```
value = input("Enter value: ")

if condition:
    print("Condition is True")
elif another_condition:
    print("Another condition is True")
else:
    print("All conditions are False")
```

---

 # Mini Exercises

 ## Exercise 1: Age Checker

 เขียนโปรแกรมรับอายุจากผู้ใช้ แล้วแสดง:

```
Adult
```

 ถ้าอายุ `>= 18`

 หรือ:

```
Minor
```

 ถ้าอายุ `< 18`

---

 ## Exercise 2: Even or Odd

 รับตัวเลขจากผู้ใช้ แล้วตรวจสอบว่าเป็น:

```
Even
```

 หรือ:

```
Odd
```

---

 ## Exercise 3: Grade Checker

 รับคะแนนจากผู้ใช้ แล้วแสดง Grade:

```
90 - 100 → A
80 - 89  → B
70 - 79  → C
60 - 69  → D
0 - 59   → F
```

---

 ## Exercise 4: Simple Login

 สร้างโปรแกรมรับ:

```
Username
Password
```

 ถ้า Username และ Password ถูกต้อง ให้แสดง:

```
Login successful
```

 ถ้าไม่ถูกต้อง ให้แสดง:

```
Invalid username or password
```

---

 ## Exercise 5: Simple Calculator

 รับ:

```
First number
Operator
Second number
```

 รองรับ:

```
+
-
*
/
```

 แล้วแสดงผลลัพธ์ที่ถูกต้อง

---

 # Next Lesson

 หลังจากเรียน `Input` และ `if / elif / else` แล้ว ขั้นต่อไปคือ:

 1. **for Loop**
2. **while Loop**
3. **Functions**
4. **List Methods**
5. **Dictionary Methods**
6. **Set & Tuple Operations**
7. **String Methods**
8. **Exception Handling**
9. **File Handling**
10. **Modules & Packages**
11. **Object-Oriented Programming (OOP)**
12. **Virtual Environment & pip**
13. **Mini Projects**

 ลำดับถัดไปที่แนะนำคือ **`for Loop`** เพราะจะทำให้สามารถสั่งให้โปรแกรมทำงานซ้ำกับข้อมูลหลาย ๆ ค่าได้ เช่น แสดงตัวเลข 1–10, วนดูสมาชิกใน List และสร้างตารางสูตรคูณ

 ถ้าต้องการตั้งชื่อตามไฟล์ก่อนหน้า แนะนำใช้ **`004_python_input_and_conditional_statements.md`** ครับ