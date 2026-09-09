# Python Naming Conventions

 ## 1\. camelCase

 คำแรกขึ้นต้นด้วยตัวเล็ก และคำถัดไปขึ้นต้นด้วยตัวใหญ่

```python
studentName = "Jessa"
totalPrice = 100
myAge = 20
```

 > มักพบใน JavaScript, Java และภาษาอื่น ๆ\
>  **Python นิยมใช้ `snake_case` มากกว่า**

---

 ## 2\. snake\_case

 ใช้ตัวพิมพ์เล็กทั้งหมด และคั่นคำด้วย `_`

```python
student_name = "Jessa"
total_price = 100
my_age = 20
```

 **Python นิยมใช้แบบนี้**

```python
def calculate_total_price():
    pass

student_name = "Jessa"
```

---

 ## 3\. PascalCase

 ทุกคำขึ้นต้นด้วยตัวพิมพ์ใหญ่ และไม่มี `_`

```python
StudentName = "Jessa"
TotalPrice = 100
MyAge = 20
```

 ใน Python มักใช้กับ **Class**

```python
class Student:
    pass

class BankAccount:
    pass
```

---

 ## Summary

 | Case | Example | Python Usage |
| --- | --- | --- |
| camelCase | `studentName` | ไม่ค่อยนิยม |
| snake\_case | `student_name` | Variables / Functions |
| PascalCase | `StudentName` | Classes |

 ## จำง่าย ๆ

```python
camelCase    → studentName
snake_case   → student_name
PascalCase   → StudentName
```

 ## Python Convention

```python
# Variable
student_name = "Jessa"

# Function
def get_student_name():
    pass

# Class
class StudentProfile:
    pass
```