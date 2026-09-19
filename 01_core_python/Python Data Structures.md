# Python Data Structures

Python provides several built-in data structures for storing and organizing data.

The most commonly used data structures are:

- **String** — ordered, immutable sequence of characters
- **List** — ordered, mutable collection
- **Tuple** — ordered, immutable collection
- **Set** — unordered collection of unique elements
- **Dictionary** — collection of key-value pairs

---

## 📑 Table of Contents

1. [Introduction](#python-data-structures)
2. [String](#1-string)
   - [Creating Strings](#creating-strings)
   - [Accessing Characters](#accessing-characters)
   - [String Slicing](#string-slicing)
   - [String Immutability](#string-immutability)
   - [String Concatenation & Repetition](#string-concatenation--repetition)
   - [Common String Methods](#common-string-methods)
   - [String Formatting](#string-formatting)
   - [Escape Characters](#escape-characters)
   - [Checking Substrings](#checking-substrings)
   - [Iterating Through a String](#iterating-through-a-string)
   - [String Comprehension](#string-comprehension)
3. [List](#2-list)
   - [Creating & Accessing Lists](#creating-a-list)
   - [Modifying Elements](#modifying-elements)
   - [Common List Methods](#common-list-methods)
   - [List Comprehension](#list-comprehension)
4. [Tuple](#3-tuple)
   - [Creating Tuples](#creating-tuples)
   - [Single-Element Tuple](#single-element-tuple)
   - [Tuple Unpacking](#tuple-unpacking)
   - [Common Tuple Methods](#common-tuple-methods)
5. [Set](#4-set)
   - [Creating an Empty Set](#creating-an-empty-set)
   - [Adding & Removing Elements](#adding-elements)
   - [Set Operations](#set-operations)
6. [Dictionary](#5-dictionary)
   - [Accessing Values](#accessing-values)
   - [Adding & Updating Values](#adding-or-updating-values)
   - [Removing Elements](#removing-elements-1)
   - [Important Dictionary Methods](#important-dictionary-methods)
   - [Iterating Through a Dictionary](#iterating-through-a-dictionary)
   - [Dictionary Comprehension](#dictionary-comprehension)
7. [Comparison](#6-comparison)
8. [Choosing the Right Data Structure](#7-choosing-the-right-data-structure)
9. [Nested Data Structures](#8-nested-data-structures)
10. [Important Quick Reference](#9-important-quick-reference)
11. [Data Structure Cheat Sheet](#10-data-structure-cheat-sheet)
12. [Remember](#11-remember)
13. [Related Topics](#related-topics)

---

## 1. String

A **string** is an ordered and immutable sequence of characters.

Strings are one of the most fundamental data types in Python, used to represent text.

### Syntax

```python
string_name = "text"
# or
string_name = 'text'
```

### Example

```python
message = "Hello, Python!"

print(message)
print(message[0])
```

### Output

```text
Hello, Python!
H
```

### Creating Strings

```python
name = "Rudrapratap"
course = 'BCA'
multiline = """This is a
multiline string"""
empty = ""
```

Single quotes and double quotes work identically:

```python
text1 = "Hello"
text2 = 'Hello'

print(text1 == text2)
```

```text
True
```

### Accessing Characters

Strings support indexing like lists:

```python
text = "Python"

print(text[0])     # P
print(text[-1])    # n
print(text[2])     # t
```

### String Slicing

```python
text = "Python Programming"

print(text[0:6])      # Python
print(text[7:])       # Programming
print(text[:6])       # Python
print(text[-11:])     # Programming
print(text[::2])      # Pto rgamn
```

### String Immutability

Strings cannot be modified after creation:

```python
text = "Python"

text[0] = "J"
```

```text
TypeError: 'str' object does not support item assignment
```

To "modify" a string, create a new one:

```python
text = "Python"

text = "J" + text[1:]

print(text)
```

```text
Jython
```

### String Concatenation & Repetition

#### Concatenation

```python
first = "Hello"
second = "World"

result = first + " " + second

print(result)
```

```text
Hello World
```

#### Repetition

```python
text = "Ha"

print(text * 3)
```

```text
HaHaHa
```

### Common String Methods

#### `upper()`

Converts to uppercase.

```python
text = "python"

print(text.upper())
```

```text
PYTHON
```

#### `lower()`

Converts to lowercase.

```python
text = "PYTHON"

print(text.lower())
```

```text
python
```

#### `capitalize()`

Capitalizes the first character.

```python
text = "python programming"

print(text.capitalize())
```

```text
Python programming
```

#### `title()`

Capitalizes the first letter of each word.

```python
text = "python programming"

print(text.title())
```

```text
Python Programming
```

#### `strip()`

Removes leading and trailing whitespace.

```python
text = "  Hello  "

print(text.strip())
```

```text
Hello
```

Also: `lstrip()` (left), `rstrip()` (right)

#### `replace()`

Replaces occurrences of a substring.

```python
text = "Hello World"

print(text.replace("World", "Python"))
```

```text
Hello Python
```

#### `split()`

Splits a string into a list.

```python
text = "Python,Java,C++"

languages = text.split(",")

print(languages)
```

```text
['Python', 'Java', 'C++']
```

Default split by whitespace:

```python
text = "Hello World Python"

print(text.split())
```

```text
['Hello', 'World', 'Python']
```

#### `join()`

Joins elements of a list into a string.

```python
words = ["Python", "is", "awesome"]

sentence = " ".join(words)

print(sentence)
```

```text
Python is awesome
```

#### `find()`

Returns the index of the first occurrence, or -1 if not found.

```python
text = "Python Programming"

print(text.find("Pro"))
print(text.find("Java"))
```

```text
7
-1
```

#### `count()`

Counts occurrences of a substring.

```python
text = "banana"

print(text.count("a"))
```

```text
3
```

#### `startswith()` and `endswith()`

Check if a string starts or ends with a substring.

```python
text = "Python Programming"

print(text.startswith("Py"))
print(text.endswith("ing"))
```

```text
True
True
```

#### `isdigit()`, `isalpha()`, `isalnum()`

Check string content type.

```python
print("123".isdigit())
print("abc".isalpha())
print("abc123".isalnum())
```

```text
True
True
True
```

### String Formatting

#### Using f-strings (recommended)

```python
name = "Rudrapratap"
age = 21

message = f"My name is {name} and I am {age} years old."

print(message)
```

```text
My name is Rudrapratap and I am 21 years old.
```

#### Using `format()`

```python
message = "My name is {} and I am {} years old.".format("Rudrapratap", 21)

print(message)
```

#### Using `%` (old style)

```python
message = "My name is %s and I am %d years old." % ("Rudrapratap", 21)

print(message)
```

### Escape Characters

| Escape | Meaning |
|--------|---------|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |

```python
text = "Hello\nWorld"

print(text)
```

```text
Hello
World
```

```python
path = "C:\\Users\\Rudra"

print(path)
```

```text
C:\Users\Rudra
```

### Checking Substrings

```python
text = "Python Programming"

print("Python" in text)
print("Java" in text)
print("Java" not in text)
```

```text
True
False
True
```

### Iterating Through a String

```python
text = "Python"

for char in text:
    print(char)
```

```text
P
y
t
h
o
n
```

### String Comprehension

Although strings are immutable, you can create new strings using list comprehension and join:

```python
text = "python"

uppercase = ''.join([char.upper() for char in text])

print(uppercase)
```

```text
PYTHON
```

Filter vowels:

```python
text = "Hello World"

consonants = ''.join([char for char in text if char.lower() not in 'aeiou'])

print(consonants)
```

```text
Hll Wrld
```

---

## 2. List

A **list** is an ordered and mutable collection. It can contain duplicate values and different data types.

### Syntax

```python
list_name = [item1, item2, item3]
```

### Example

```python
fruits = ["apple", "banana", "mango"]

print(fruits)
print(fruits[0])
```

### Output

```text
['apple', 'banana', 'mango']
apple
```

### Creating a List

```python
numbers = [10, 20, 30]
mixed = [10, "Python", 3.14, True]
empty = []
```

### Accessing Elements

```python
numbers = [10, 20, 30, 40, 50]

print(numbers[0])     # 10
print(numbers[-1])    # 50
print(numbers[1:4])   # [20, 30, 40]
```

### Modifying Elements

```python
numbers = [10, 20, 30]

numbers[1] = 200

print(numbers)
```

```text
[10, 200, 30]
```

### Common List Methods

#### `append()`

Adds an element to the end.

```python
numbers = [1, 2, 3]

numbers.append(4)

print(numbers)
```

```text
[1, 2, 3, 4]
```

#### `insert()`

Adds an element at a specific position.

```python
numbers = [1, 2, 4]

numbers.insert(2, 3)

print(numbers)
```

```text
[1, 2, 3, 4]
```

#### `extend()`

Adds multiple elements.

```python
numbers = [1, 2]

numbers.extend([3, 4, 5])

print(numbers)
```

```text
[1, 2, 3, 4, 5]
```

#### `remove()`

Removes the first occurrence of a value.

```python
numbers = [1, 2, 3, 2]

numbers.remove(2)

print(numbers)
```

```text
[1, 3, 2]
```

#### `pop()`

Removes and returns an element.

```python
numbers = [10, 20, 30]

value = numbers.pop()

print(value)
print(numbers)
```

```text
30
[10, 20]
```

Remove using an index:

```python
numbers.pop(0)
```

#### `clear()`

Removes all elements.

```python
numbers = [1, 2, 3]

numbers.clear()

print(numbers)
```

```text
[]
```

#### `index()`

Returns the index of the first occurrence.

```python
fruits = ["apple", "banana", "mango"]

print(fruits.index("banana"))
```

```text
1
```

#### `count()`

Counts occurrences.

```python
numbers = [1, 2, 2, 3, 2]

print(numbers.count(2))
```

```text
3
```

#### `sort()`

Sorts the list in place.

```python
numbers = [5, 2, 8, 1]

numbers.sort()

print(numbers)
```

```text
[1, 2, 5, 8]
```

Descending order:

```python
numbers.sort(reverse=True)
```

#### `reverse()`

Reverses the list in place.

```python
numbers = [1, 2, 3, 4]

numbers.reverse()

print(numbers)
```

```text
[4, 3, 2, 1]
```

### List Comprehension

A concise way to create lists.

### Syntax

```python
[expression for item in iterable]
```

### Example

```python
squares = [x ** 2 for x in range(1, 6)]

print(squares)
```

```text
[1, 4, 9, 16, 25]
```

With a condition:

```python
even_numbers = [x for x in range(10) if x % 2 == 0]

print(even_numbers)
```

```text
[0, 2, 4, 6, 8]
```

---

# 3. Tuple

A **tuple** is an ordered and immutable collection.

Once created, its elements cannot be changed.

### Syntax

```python
tuple_name = (item1, item2, item3)
```

### Example

```python
coordinates = (10, 20)

print(coordinates)
print(coordinates[0])
```

```text
(10, 20)
10
```

### Creating Tuples

```python
numbers = (1, 2, 3)
mixed = (10, "Python", 3.14)
empty = ()
```

### Single-Element Tuple

A comma is required:

```python
number = (10,)

print(type(number))
```

```text
<class 'tuple'>
```

Without the comma:

```python
number = (10)

print(type(number))
```

```text
<class 'int'>
```

### Tuple Unpacking

```python
person = ("Rudrapratap", 21, "BCA")

name, age, course = person

print(name)
print(age)
print(course)
```

### Common Tuple Methods

Because tuples are immutable, they have fewer methods.

#### `count()`

```python
numbers = (1, 2, 2, 3)

print(numbers.count(2))
```

```text
2
```

#### `index()`

```python
numbers = (10, 20, 30)

print(numbers.index(20))
```

```text
1
```

---

# 4. Set

A **set** is an unordered collection of unique elements.

Duplicate values are automatically removed.

### Syntax

```python
set_name = {item1, item2, item3}
```

### Example

```python
numbers = {1, 2, 3, 2, 1}

print(numbers)
```

```text
{1, 2, 3}
```

> Sets do not support indexing because they are unordered collections.

### Creating an Empty Set

Be careful:

```python
empty = set()
```

This:

```python
empty = {}
```

creates an **empty dictionary**, not a set.

### Adding Elements

#### `add()`

```python
numbers = {1, 2, 3}

numbers.add(4)

print(numbers)
```

```text
{1, 2, 3, 4}
```

#### `update()`

Adds multiple elements.

```python
numbers = {1, 2}

numbers.update([3, 4, 5])

print(numbers)
```

### Removing Elements

#### `remove()`

Raises an error if the element does not exist.

```python
numbers = {1, 2, 3}

numbers.remove(2)

print(numbers)
```

#### `discard()`

Does not raise an error if the element doesn't exist.

```python
numbers = {1, 2, 3}

numbers.discard(10)

print(numbers)
```

#### `pop()`

Removes and returns an arbitrary element.

```python
numbers = {1, 2, 3}

value = numbers.pop()

print(value)
```

### Set Operations

#### Union

Combines elements from both sets.

```python
a = {1, 2, 3}
b = {3, 4, 5}

print(a | b)
```

```text
{1, 2, 3, 4, 5}
```

Using a method:

```python
a.union(b)
```

#### Intersection

Returns common elements.

```python
a = {1, 2, 3}
b = {2, 3, 4}

print(a & b)
```

```text
{2, 3}
```

#### Difference

Returns elements present in the first set but not the second.

```python
a = {1, 2, 3}
b = {2, 3, 4}

print(a - b)
```

```text
{1}
```

#### Symmetric Difference

Returns elements that are present in either set but not both.

```python
a = {1, 2, 3}
b = {2, 3, 4}

print(a ^ b)
```

```text
{1, 4}
```

---

# 5. Dictionary

A **dictionary** stores data as **key-value pairs**.

It is mutable and keys must be unique and hashable.

### Syntax

```python
dictionary = {
    key1: value1,
    key2: value2
}
```

### Example

```python
student = {
    "name": "Rudrapratap",
    "age": 21,
    "course": "BCA"
}

print(student)
print(student["name"])
```

### Output

```text
{'name': 'Rudrapratap', 'age': 21, 'course': 'BCA'}
Rudrapratap
```

### Accessing Values

```python
student = {
    "name": "Rudrapratap",
    "age": 21
}

print(student["name"])
```

Using `get()`:

```python
print(student.get("name"))
```

The difference:

```python
student["city"]
```

raises `KeyError` if `"city"` doesn't exist.

But:

```python
student.get("city")
```

returns:

```text
None
```

You can provide a default:

```python
student.get("city", "Unknown")
```

### Adding or Updating Values

```python
student = {
    "name": "Rudrapratap",
    "age": 21
}

student["city"] = "Dharmanagar"

student["age"] = 22

print(student)
```

### Removing Elements

#### `pop()`

```python
student = {
    "name": "Rudrapratap",
    "age": 21
}

age = student.pop("age")

print(age)
print(student)
```

#### `popitem()`

Removes and returns the last inserted key-value pair.

```python
student = {
    "name": "Rudrapratap",
    "age": 21
}

item = student.popitem()

print(item)
```

#### `del`

```python
del student["age"]
```

#### `clear()`

```python
student.clear()
```

### Important Dictionary Methods

#### `keys()`

```python
student.keys()
```

#### `values()`

```python
student.values()
```

#### `items()`

```python
student.items()
```

### Iterating Through a Dictionary

Keys:

```python
student = {
    "name": "Rudrapratap",
    "age": 21
}

for key in student:
    print(key)
```

Keys and values:

```python
for key, value in student.items():
    print(key, value)
```

### Dictionary Comprehension

### Syntax

```python
{key: value for item in iterable}
```

### Example

```python
squares = {x: x ** 2 for x in range(1, 6)}

print(squares)
```

```text
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

With a condition:

```python
even_squares = {
    x: x ** 2
    for x in range(1, 6)
    if x % 2 == 0
}

print(even_squares)
```

```text
{2: 4, 4: 16}
```

---

# 6. Comparison

| Feature | String | List | Tuple | Set | Dictionary |
|---|---|---|---|---|---|
| Ordered | Yes | Yes | Yes | No* | Yes** |
| Mutable | No | Yes | No | Yes | Yes |
| Duplicates | Yes | Yes | Yes | No | Keys: No |
| Indexing | Yes | Yes | Yes | No | By key |
| Syntax | `""` or `''` | `[]` | `()` | `{}` | `{key: value}` |
| Main use | Text data | General collection | Fixed data | Unique values | Key-value data |

\* Set elements do not support positional indexing.  
\** Dictionaries preserve insertion order in modern Python (Python 3.7+ language guarantee).

---

# 7. Choosing the Right Data Structure

### Use a String when:

```python
name = "Rudrapratap"
```

You need:

- Textual data representation
- Character-level access or slicing
- String formatting and parsing
- Immutable sequences of characters

### Use a List when:

```python
students = ["A", "B", "C"]
```

You need:

- Ordered data
- Duplicates
- Frequent modification
- Index-based access

### Use a Tuple when:

```python
coordinates = (28.61, 77.20)
```

You need:

- Ordered data
- Immutable data
- Fixed collections

### Use a Set when:

```python
unique_users = {101, 102, 103}
```

You need:

- Unique values
- Fast membership testing
- Set operations

Example:

```python
if user_id in unique_users:
    print("User exists")
```

### Use a Dictionary when:

```python
user = {
    "id": 101,
    "name": "Rahul",
    "age": 22
}
```

You need:

- Key-value relationships
- Fast lookup by key
- Structured records

---

# 8. Nested Data Structures

Data structures can contain other data structures.

### List of Dictionaries

Very common in Data Science and APIs:

```python
students = [
    {"name": "A", "marks": 85},
    {"name": "B", "marks": 92},
    {"name": "C", "marks": 78}
]

print(students[0]["name"])
```

```text
A
```

### Dictionary Containing Lists

```python
student = {
    "name": "Rudrapratap",
    "skills": ["Python", "SQL", "Pandas"]
}

print(student["skills"][0])
```

```text
Python
```

### List of Lists

Commonly used to represent matrix-like data:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matrix[1][2])
```

```text
6
```

---

# 9. Important Quick Reference

```python
# String
text = "Python"
text.upper()
text.lower()
text.strip()
text.replace("Py", "Ja")
text.split()
" ".join(["a", "b"])
text.find("th")
text.count("o")

# List
numbers = [1, 2, 3]
numbers.append(4)
numbers.remove(2)
numbers.pop()
numbers.sort()

# Tuple
numbers = (1, 2, 3)
numbers.count(2)
numbers.index(2)

# Set
numbers = {1, 2, 3}
numbers.add(4)
numbers.remove(2)
numbers | {4, 5}       # Union
numbers & {2, 3}       # Intersection
numbers - {2, 3}       # Difference

# Dictionary
student = {"name": "Rahul", "age": 21}
student["name"]
student.get("name")
student.keys()
student.values()
student.items()
student.pop("age")
```

---

# 10. Data Structure Cheat Sheet

| Operation | String | List | Tuple | Set | Dictionary |
|---|---:|---:|---:|---:|---:|
| Create | `""` / `''` | `[]` | `()` | `set()` / `{1,2}` | `{}` |
| Access | Index | Index | Index | Membership | Key |
| Add | Concatenation (`+`) | `append()` | ❌ | `add()` | `dict[key] = value` |
| Remove | Slicing / `replace()` | `remove()` | ❌ | `remove()` / `discard()` | `pop()` / `del` |
| Modify | ❌ | ✅ | ❌ | ✅ | ✅ |
| Duplicates | ✅ | ✅ | ✅ | ❌ | Keys ❌ |
| Slicing | ✅ | ✅ | ✅ | ❌ | ❌ |
| Comprehension | ⚠️** | ✅ | ⚠️* | ✅ | ✅ |

\* Tuple comprehensions do not exist directly; `(x for x in ...)` creates a generator expression.  
\** String comprehension is achieved via list comprehension with `''.join()`.

---

# 11. Remember

```text
STRING     → Ordered + Immutable + Characters
LIST       → Ordered + Mutable + Duplicates
TUPLE      → Ordered + Immutable + Duplicates
SET        → Unique + Unordered + Mutable
DICTIONARY → Key → Value + Mutable + Unique Keys
```

### Simple mental model

```text
String     → "I need to work with text data."
List       → "I need a collection I can change."
Tuple      → "I need a collection that should not change."
Set        → "I only care about unique values."
Dictionary → "I want to access data using a key."
```
