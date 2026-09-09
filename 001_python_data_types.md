![Python Data Types](./pic/python_data_types.jpg)
Absolutely — here is the whole thing as **one `.md` file**, including the image reference.

 Python Data Types Markdown File✓

# Python Data Types

 ## Numbers

 ### Int

 จำนวนเต็ม (Integer)

```python
age = 20

print(age)
print(type(age))
```

 Output:

```python
20
<class 'int'>
```

 ### Float

 จำนวนทศนิยม (Floating-point)

```python
price = 35.75

print(price)
print(type(price))
```

 Output:

```python
35.75
<class 'float'>
```

 ### Complex

 จำนวนเชิงซ้อน (Complex number)

```python
number = 1 + 3j

print(number)
print(type(number))
```

 Output:

```python
(1+3j)
<class 'complex'>
```

---

 ## Bool

 Boolean มี 2 ค่า:

 - `True` — ถูก / จริง
- `False` — ผิด / เท็จ

```python
is_student = True
is_teacher = False

print(is_student)
print(is_teacher)
print(type(is_student))
```

 Output:

```python
True
False
<class 'bool'>
```

 ### Truthy Values

 ค่าที่ Python มองว่าเป็น `True`

```python
print(bool(1))
print(bool("Hello"))
print(bool([1, 2]))
print(bool({1, 2}))
```

 Output:

```python
True
True
True
True
```

 ### Falsy Values

 ค่าที่ Python มองว่าเป็น `False`

```python
print(bool(0))
print(bool(""))
print(bool([]))
print(bool({}))
print(bool(None))
print(bool(False))
```

 Output:

```python
False
False
False
False
False
False
```

---

 ## Set

 Set คือข้อมูลที่:

 - ไม่ซ้ำกัน
- ไม่มีลำดับที่แน่นอน

```python
numbers = {2, 4, 6}

print(numbers)
print(type(numbers))
```

 Output:

```python
{2, 4, 6}
<class 'set'>
```

 ### Set ไม่เก็บข้อมูลซ้ำ

```python
numbers = {1, 2, 2, 3, 3}

print(numbers)
```

 Output:

```python
{1, 2, 3}
```

---

 ## Dictionary

 Dictionary หรือ `dict` เป็นข้อมูลแบบ **Key-Value**

```python
student = {
    1: "a",
    2: "b"
}

print(student)
print(type(student))
```

 Output:

```python
{1: 'a', 2: 'b'}
<class 'dict'>
```

 ### Access Value

 สามารถใช้ Key เพื่อเข้าถึง Value ได้:

```python
student = {
    1: "a",
    2: "b"
}

print(student[1])
print(student[2])
```

 Output:

```python
a
b
```

---

 # Sequence

 ## String

 String คือข้อความหรือลำดับของตัวอักษร

```python
name = "Jessa"

print(name)
print(type(name))
```

 Output:

```python
Jessa
<class 'str'>
```

---

 ## List

 List คือรายการข้อมูลที่สามารถแก้ไขได้ (**Mutable**)

```python
numbers = [2, "a", 5.7]

print(numbers)
print(type(numbers))
```

 Output:

```python
[2, 'a', 5.7]
<class 'list'>
```

 ### Modify List

 สามารถเปลี่ยนข้อมูลใน List ได้:

```python
numbers = [2, "a", 5.7]

numbers[0] = 10

print(numbers)
```

 Output:

```python
[10, 'a', 5.7]
```

---

 ## Tuple

 Tuple คือรายการข้อมูลที่แก้ไขไม่ได้ (**Immutable**)

```python
numbers = (3, 4.5, "b")

print(numbers)
print(type(numbers))
```

 Output:

```python
(3, 4.5, 'b')
<class 'tuple'>
```

 ### Access Tuple

```python
numbers = (3, 4.5, "b")

print(numbers[0])
print(numbers[1])
print(numbers[2])
```

 Output:

```python
3
4.5
b
```

---

 # Summary

 | Data Type | Example | Description |
| --- | --- | --- |
| `int` | `20` | จำนวนเต็ม |
| `float` | `35.75` | จำนวนทศนิยม |
| `complex` | `1 + 3j` | จำนวนเชิงซ้อน |
| `bool` | `True` | จริง / เท็จ |
| `set` | `{1, 2, 3}` | ข้อมูลไม่ซ้ำกัน |
| `dict` | `{1: "a"}` | Key-Value |
| `str` | `"Jessa"` | ข้อความ |
| `list` | `[1, 2, 3]` | แก้ไขได้ |
| `tuple` | `(1, 2, 3)` | แก้ไขไม่ได้ |

---

 # Checking Data Type

 ใช้ `type()` เพื่อดูว่า Value เป็น Data Type อะไร:

```python
print(type(20))
print(type(35.75))
print(type(1 + 3j))
print(type(True))
print(type({1, 2, 3}))
print(type({1: "a"}))
print(type("Jessa"))
print(type([1, 2, 3]))
print(type((1, 2, 3)))
```

 Output:

```python
<class 'int'>
<class 'float'>
<class 'complex'>
<class 'bool'>
<class 'set'>
<class 'dict'>
<class 'str'>
<class 'list'>
<class 'tuple'>
```