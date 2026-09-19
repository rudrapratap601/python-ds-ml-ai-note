
## 📑 Table of Contents

1. [Why Control Flow Matters](#1-why-control-flow-matters)
2. [Conditional Statements](#2-conditional-statements)
   - [Basic `if`](#basic-if)
   - [`if-else`](#if-else)
   - [`if-elif-else`](#if-elif-else)
   - [Nested Conditions](#nested-conditions)
   - [Multiple Conditions](#multiple-conditions)
   - [Truthy and Falsy Values](#truthy-and-falsy-values)
   - [Conditional Expression](#conditional-expression)
   - [Data Science Examples](#data-science-examples)
3. [Match-Case](#3-match-case)
   - [Basic Syntax](#basic-syntax)
   - [Matching Multiple Values](#matching-multiple-values)
   - [Default Case](#default-case)
   - [Matching Patterns](#matching-patterns)
   - [Guards](#guards)
   - [Data Engineering Example](#data-engineering-example)
   - [When to Use Match-Case](#when-to-use-match-case)
4. [Loops](#4-loops)
   - [`for` Loop](#for-loop)
   - [`while` Loop](#while-loop)
   - [Looping Through Data Structures](#looping-through-data-structures)
   - [`range()`](#range)
   - [`enumerate()`](#enumerate)
   - [`zip()`](#zip)
   - [Nested Loops](#nested-loops)
5. [The `break` Statement](#5-the-break-statement)
   - [Basic Example](#basic-break-example)
   - [Data Engineering Example](#break-in-data-engineering)
6. [The `continue` Statement](#6-the-continue-statement)
   - [Basic Example](#basic-continue-example)
   - [Data Engineering Example](#continue-in-data-engineering)
7. [`break` vs `continue`](#7-break-vs-continue)
8. [Loop `else`](#8-loop-else)
9. [Practical Data Science & Engineering Examples](#9-practical-data-science--engineering-examples)
   - [Data Validation](#data-validation)
   - [ETL Processing](#etl-processing)
   - [Missing Data](#missing-data)
   - [Batch Processing](#batch-processing)
   - [Searching Records](#searching-records)
10. [Common Mistakes](#10-common-mistakes)
11. [Performance & Readability](#11-performance--readability)
12. [Quick Reference](#12-quick-reference)
13. [Interview Questions](#13-interview-questions)
14. [Remember](#14-remember)

---

# 1. Why Control Flow Matters

A Python program normally executes statements from top to bottom.

Control-flow statements allow the program to make decisions and repeat operations.

The three major ideas are:

```text
CONDITION
    ↓
Make a decision

LOOP
    ↓
Repeat an operation

BREAK / CONTINUE
    ↓
Control the loop
```

In Data Engineering and Data Science:

- **Conditions** filter data, validate inputs, and route workflows.
- **Loops** process batches, iterate through datasets, and perform transformations.
- **Break/Continue** handle exceptions, skip invalid records, and control processing flows.

---

# 2. Conditional Statements

Conditional statements allow a program to execute code based on whether a condition is `True` or `False`.

## Basic `if`

Execute code only if a condition is true.

### Syntax

```python
if condition:
    # code to execute
```

### Example

```python
age = 21

if age >= 18:
    print("You are an adult")
```

```text
You are an adult
```

## `if-else`

Execute one block if the condition is true, another if false.

### Syntax

```python
if condition:
    # code if true
else:
    # code if false
```

### Example

```python
score = 45

if score >= 50:
    print("Pass")
else:
    print("Fail")
```

```text
Fail
```

## `if-elif-else`

Check multiple conditions in sequence.

### Syntax

```python
if condition1:
    # code if condition1 is true
elif condition2:
    # code if condition2 is true
else:
    # code if none are true
```

### Example

```python
marks = 85

if marks >= 90:
    grade = "A"
elif marks >= 75:
    grade = "B"
elif marks >= 60:
    grade = "C"
else:
    grade = "F"

print(f"Grade: {grade}")
```

```text
Grade: B
```

## Nested Conditions

Conditions inside conditions.

### Example

```python
age = 25
has_license = True

if age >= 18:
    if has_license:
        print("You can drive")
    else:
        print("Get a license first")
else:
    print("You are too young")
```

```text
You can drive
```

## Multiple Conditions

Combine conditions using `and`, `or`, `not`.

### Example

```python
age = 20
student = True

if age >= 18 and student:
    print("Eligible for student discount")
```

```text
Eligible for student discount
```

```python
day = "Sunday"

if day == "Saturday" or day == "Sunday":
    print("Weekend!")
```

```text
Weekend!
```

```python
logged_in = False

if not logged_in:
    print("Please log in")
```

```text
Please log in
```

## Truthy and Falsy Values

Python treats certain values as `False` even if they are not the boolean `False`.

### Falsy values

```python
False, 0, 0.0, "", None, [], {}, ()
```

### Example

```python
data = []

if data:
    print("Data available")
else:
    print("No data")
```

```text
No data
```

```python
name = ""

if not name:
    print("Name is required")
```

```text
Name is required
```

## Conditional Expression

Also known as a **ternary operator**.

### Syntax

```python
value_if_true if condition else value_if_false
```

### Example

```python
age = 17

status = "Adult" if age >= 18 else "Minor"

print(status)
```

```text
Minor
```

Inline conditional for concise logic:

```python
marks = 82

result = "Pass" if marks >= 50 else "Fail"

print(result)
```

```text
Pass
```

## Data Science Examples

### Filter rows based on condition

```python
temperature = 102

if temperature > 100:
    print("Outlier detected")
```

### Categorize numerical data

```python
income = 75000

if income < 30000:
    category = "Low"
elif income < 70000:
    category = "Medium"
else:
    category = "High"

print(category)
```

```text
High
```

---

# 3. Match-Case

Introduced in Python 3.10, `match-case` is a pattern-matching statement similar to `switch` in other languages.

## Basic Syntax

```python
match variable:
    case pattern1:
        # code
    case pattern2:
        # code
    case _:
        # default case
```

## Matching Multiple Values

```python
day = "Monday"

match day:
    case "Monday":
        print("Start of the week")
    case "Friday":
        print("End of the work week")
    case "Saturday" | "Sunday":
        print("Weekend!")
    case _:
        print("Midweek")
```

```text
Start of the week
```

## Default Case

The `_` acts as a catch-all default.

```python
status_code = 404

match status_code:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case 500:
        print("Server Error")
    case _:
        print("Unknown status")
```

```text
Not Found
```

## Matching Patterns

Match structured data like tuples or lists.

```python
point = (0, 0)

match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"On Y-axis at {y}")
    case (x, 0):
        print(f"On X-axis at {x}")
    case (x, y):
        print(f"Point at ({x}, {y})")
```

```text
Origin
```

## Guards

Add conditions to patterns using `if`.

```python
age = 16

match age:
    case a if a < 13:
        print("Child")
    case a if a < 18:
        print("Teenager")
    case a if a < 60:
        print("Adult")
    case _:
        print("Senior")
```

```text
Teenager
```

## Data Engineering Example

Routing data based on file type:

```python
file_type = "csv"

match file_type:
    case "csv":
        print("Processing CSV file")
    case "json":
        print("Processing JSON file")
    case "parquet":
        print("Processing Parquet file")
    case _:
        print("Unsupported file type")
```

```text
Processing CSV file
```

## When to Use Match-Case

Use `match-case` when:

- You have many discrete values to check
- Pattern matching adds clarity
- You're working with structured data (tuples, lists)

Use `if-elif-else` when:

- You need complex boolean conditions
- Checking ranges or inequalities
- Python version < 3.10

---

# 4. Loops

Loops repeat a block of code multiple times.

## `for` Loop

Iterate over a sequence (list, string, range, etc.).

### Syntax

```python
for item in sequence:
    # code to repeat
```

### Example

```python
fruits = ["apple", "banana", "mango"]

for fruit in fruits:
    print(fruit)
```

```text
apple
banana
mango
```

Iterating over a string:

```python
for char in "Python":
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

## `while` Loop

Repeat code as long as a condition is true.

### Syntax

```python
while condition:
    # code to repeat
```

### Example

```python
count = 1

while count <= 5:
    print(count)
    count += 1
```

```text
1
2
3
4
5
```

### Warning

Always ensure the condition eventually becomes `False`, or you'll create an infinite loop:

```python
# Infinite loop (don't run)
while True:
    print("This will never stop")
```

## Looping Through Data Structures

### List

```python
numbers = [10, 20, 30]

for num in numbers:
    print(num)
```

### Dictionary

```python
student = {"name": "Rahul", "age": 21}

for key, value in student.items():
    print(f"{key}: {value}")
```

```text
name: Rahul
age: 21
```

Keys only:

```python
for key in student.keys():
    print(key)
```

Values only:

```python
for value in student.values():
    print(value)
```

### Set

```python
numbers = {1, 2, 3}

for num in numbers:
    print(num)
```

## `range()`

Generate a sequence of numbers.

### Syntax

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

### Example

```python
for i in range(5):
    print(i)
```

```text
0
1
2
3
4
```

```python
for i in range(2, 8):
    print(i)
```

```text
2
3
4
5
6
7
```

```python
for i in range(0, 10, 2):
    print(i)
```

```text
0
2
4
6
8
```

## `enumerate()`

Get both index and value while looping.

### Syntax

```python
enumerate(iterable, start=0)
```

### Example

```python
fruits = ["apple", "banana", "mango"]

for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
```

```text
0: apple
1: banana
2: mango
```

Start from 1:

```python
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}: {fruit}")
```

```text
1: apple
2: banana
3: mango
```

## `zip()`

Combine multiple iterables element-wise.

### Syntax

```python
zip(iterable1, iterable2, ...)
```

### Example

```python
names = ["Alice", "Bob", "Charlie"]
scores = [85, 90, 78]

for name, score in zip(names, scores):
    print(f"{name}: {score}")
```

```text
Alice: 85
Bob: 90
Charlie: 78
```

## Nested Loops

Loops inside loops.

### Example

```python
for i in range(1, 4):
    for j in range(1, 4):
        print(f"({i}, {j})", end=" ")
    print()
```

```text
(1, 1) (1, 2) (1, 3) 
(2, 1) (2, 2) (2, 3) 
(3, 1) (3, 2) (3, 3)
```

Matrix traversal:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

for row in matrix:
    for value in row:
        print(value, end=" ")
    print()
```

```text
1 2 3 
4 5 6 
7 8 9
```

---

# 5. The `break` Statement

The `break` statement exits a loop immediately.

## Basic Break Example

```python
for i in range(10):
    if i == 5:
        break
    print(i)
```

```text
0
1
2
3
4
```

## Break in Data Engineering

Stop processing when a condition is met:

```python
records = [10, 20, None, 40, 50]

for record in records:
    if record is None:
        print("Null value detected. Stopping.")
        break
    print(record)
```

```text
10
20
Null value detected. Stopping.
```

Search for a value:

```python
users = ["Alice", "Bob", "Charlie", "David"]
target = "Charlie"

for user in users:
    if user == target:
        print(f"Found: {user}")
        break
```

```text
Found: Charlie
```

---

# 6. The `continue` Statement

The `continue` statement skips the current iteration and moves to the next one.

## Basic Continue Example

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

```text
0
1
3
4
```

## Continue in Data Engineering

Skip invalid records:

```python
records = [10, -5, 20, -10, 30]

for record in records:
    if record < 0:
        print(f"Skipping invalid record: {record}")
        continue
    print(f"Processing: {record}")
```

```text
Processing: 10
Skipping invalid record: -5
Processing: 20
Skipping invalid record: -10
Processing: 30
```

Filter out empty strings:

```python
names = ["Alice", "", "Bob", "", "Charlie"]

for name in names:
    if not name:
        continue
    print(name)
```

```text
Alice
Bob
Charlie
```

---

# 7. `break` vs `continue`

| Statement | Action | Use Case |
|-----------|--------|----------|
| `break` | Exits the loop entirely | Stop when a condition is met |
| `continue` | Skips to the next iteration | Skip invalid/unwanted data |

### Example

```python
for i in range(10):
    if i == 3:
        continue  # Skip 3
    if i == 7:
        break     # Stop at 7
    print(i)
```

```text
0
1
2
4
5
6
```

---

# 8. Loop `else`

The `else` block executes if the loop completes **without** hitting a `break`.

### Syntax

```python
for item in sequence:
    # code
else:
    # runs if loop completes normally
```

### Example

```python
numbers = [1, 2, 3, 4, 5]

for num in numbers:
    if num > 10:
        print("Found a number greater than 10")
        break
else:
    print("No number greater than 10 found")
```

```text
No number greater than 10 found
```

With `break`:

```python
numbers = [1, 2, 15, 4, 5]

for num in numbers:
    if num > 10:
        print(f"Found: {num}")
        break
else:
    print("No number greater than 10 found")
```

```text
Found: 15
```

---

# 9. Practical Data Science & Engineering Examples

## Data Validation

```python
data = [10, 20, 30, -5, 40]

for value in data:
    if value < 0:
        print(f"Invalid value: {value}")
        break
else:
    print("All values are valid")
```

```text
Invalid value: -5
```

## ETL Processing

```python
records = [
    {"id": 1, "value": 100},
    {"id": 2, "value": None},
    {"id": 3, "value": 300}
]

for record in records:
    if record["value"] is None:
        print(f"Skipping record {record['id']}: missing value")
        continue
    print(f"Processing record {record['id']}: {record['value']}")
```

```text
Processing record 1: 100
Skipping record 2: missing value
Processing record 3: 300
```

## Missing Data

```python
temperatures = [22, 25, None, 30, 28]

cleaned = []

for temp in temperatures:
    if temp is None:
        continue
    cleaned.append(temp)

print(cleaned)
```

```text
[22, 25, 30, 28]
```

## Batch Processing

```python
batch_size = 3
data = [10, 20, 30, 40, 50, 60, 70]

for i in range(0, len(data), batch_size):
    batch = data[i:i+batch_size]
    print(f"Processing batch: {batch}")
```

```text
Processing batch: [10, 20, 30]
Processing batch: [40, 50, 60]
Processing batch: [70]
```

## Searching Records

```python
users = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30},
    {"name": "Charlie", "age": 22}
]

target_name = "Bob"

for user in users:
    if user["name"] == target_name:
        print(f"Found: {user}")
        break
else:
    print(f"{target_name} not found")
```

```text
Found: {'name': 'Bob', 'age': 30}
```

---

# 10. Common Mistakes

### Mistake 1: Infinite Loop

```python
# Wrong
count = 0
while count < 5:
    print(count)
    # Forgot to increment count
```

**Fix:**

```python
count = 0
while count < 5:
    print(count)
    count += 1
```

### Mistake 2: Modifying List While Iterating

```python
# Wrong
numbers = [1, 2, 3, 4, 5]

for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)  # Modifies list during iteration
```

**Fix:**

```python
numbers = [1, 2, 3, 4, 5]

numbers = [num for num in numbers if num % 2 != 0]

print(numbers)
```

### Mistake 3: Using `=` Instead of `==`

```python
# Wrong
if age = 18:  # Assignment, not comparison
    print("Adult")
```

**Fix:**

```python
if age == 18:
    print("Adult")
```

### Mistake 4: Forgetting Indentation

```python
# Wrong
for i in range(5):
print(i)  # IndentationError
```

**Fix:**

```python
for i in range(5):
    print(i)
```

---

# 11. Performance & Readability

### List Comprehension vs Loop

**Loop:**

```python
squares = []
for x in range(10):
    squares.append(x ** 2)
```

**List comprehension (faster & cleaner):**

```python
squares = [x ** 2 for x in range(10)]
```

### Avoid Unnecessary Nested Loops

**Bad:**

```python
for i in range(100):
    for j in range(100):
        for k in range(100):
            # O(n³) complexity
            pass
```

**Better:**

Optimize by reducing nesting or using efficient algorithms.

### Use `any()` and `all()`

**Instead of:**

```python
found = False
for item in data:
    if condition(item):
        found = True
        break
```

**Use:**

```python
found = any(condition(item) for item in data)
```

---

# 12. Quick Reference

```python
# Conditional
if condition:
    pass
elif condition:
    pass
else:
    pass

# Ternary
value = "Yes" if condition else "No"

# Match-Case (Python 3.10+)
match value:
    case "A":
        pass
    case _:
        pass

# For loop
for item in iterable:
    pass

# While loop
while condition:
    pass

# Range
for i in range(start, stop, step):
    pass

# Enumerate
for index, value in enumerate(iterable):
    pass

# Zip
for a, b in zip(list1, list2):
    pass

# Break
for item in iterable:
    if condition:
        break

# Continue
for item in iterable:
    if condition:
        continue

# Loop else
for item in iterable:
    if condition:
        break
else:
    # Runs if no break
    pass
```

---

# 13. Interview Questions

**Q1: What is the difference between `break` and `continue`?**

- `break` exits the loop entirely.
- `continue` skips the current iteration and moves to the next.

**Q2: What does the `else` clause in a loop do?**

It executes only if the loop completes without hitting a `break`.

**Q3: How do you iterate over a dictionary?**

```python
for key, value in my_dict.items():
    print(key, value)
```

**Q4: What is the output?**

```python
for i in range(5):
    if i == 3:
        continue
    print(i)
```

**Answer:** `0 1 2 4`

**Q5: How do you avoid an infinite loop?**

Ensure the loop condition eventually becomes `False` or use a `break` statement.

**Q6: What is the difference between `range(5)` and `range(1, 5)`?**

- `range(5)` generates `0, 1, 2, 3, 4`
- `range(1, 5)` generates `1, 2, 3, 4`

**Q7: Can you use `break` in a `while` loop?**

Yes, `break` works in both `for` and `while` loops.

---

# 14. Remember

```text
CONDITIONAL → Make decisions (if, elif, else)
MATCH-CASE  → Pattern matching (Python 3.10+)
FOR         → Iterate over sequences
WHILE       → Repeat while condition is true
BREAK       → Exit loop immediately
CONTINUE    → Skip current iteration
LOOP ELSE   → Runs if loop completes without break
```

### Mental Model

```text
if        → "Should I do this?"
for       → "Do this for each item"
while     → "Keep doing this until..."
break     → "Stop everything now"
continue  → "Skip this one, move to next"