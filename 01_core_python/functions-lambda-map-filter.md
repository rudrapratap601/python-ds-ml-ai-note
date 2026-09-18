# Python Functions, Lambda, Map & Filter

> **Purpose:** Practical reference for Python functions and functional programming tools commonly used in Data Science, Data Engineering, ETL, automation, and ML workflows.

---

## 📑 Table of Contents

1. [Why Functions Matter](#1-why-functions-matter)
2. [Functions](#2-functions)
   - [Basic Function Syntax](#basic-function-syntax)
   - [Calling a Function](#calling-a-function)
   - [Parameters vs Arguments](#parameters-vs-arguments)
   - [Return Values](#return-values)
   - [Default Arguments](#default-arguments)
   - [Keyword & Positional Arguments](#keyword--positional-arguments)
   - [Variable-Length Arguments](#variable-length-arguments)
   - [`*args`](#args)
   - [`**kwargs`](#kwargs)
   - [Positional-Only & Keyword-Only Parameters](#positional-only--keyword-only-parameters)
   - [Scope](#scope)
   - [Docstrings](#docstrings)
   - [Type Hints](#type-hints)
   - [Functions as Objects](#functions-as-objects)
3. [Lambda Functions](#3-lambda-functions)
   - [Lambda Syntax](#lambda-syntax)
   - [Basic Examples](#basic-examples)
   - [Lambda with Sorting](#lambda-with-sorting)
   - [Lambda in Data Science](#lambda-in-data-science)
   - [When to Use Lambda](#when-to-use-lambda)
4. [`map()` Function](#4-map-function)
   - [How `map()` Works](#how-map-works)
   - [Basic Example](#basic-example)
   - [Using Lambda with `map()`](#using-lambda-with-map)
   - [Mapping Multiple Iterables](#mapping-multiple-iterables)
   - [`map()` in Data Science & Engineering](#map-in-data-science--engineering)
5. [`filter()` Function](#5-filter-function)
   - [How `filter()` Works](#how-filter-works)
   - [Basic Example](#basic-example-1)
   - [Using Lambda with `filter()`](#using-lambda-with-filter)
   - [`filter()` in Data Science & Engineering](#filter-in-data-science--engineering)
6. [Combining Lambda, `map()`, and `filter()`](#6-combining-lambda-map-and-filter)
7. [List Comprehension vs `map()` vs `filter()`](#7-list-comprehension-vs-map-vs-filter)
8. [Practical Data Science Examples](#8-practical-data-science-examples)
   - [Cleaning Data](#cleaning-data)
   - [Feature Transformation](#feature-transformation)
   - [Filtering Records](#filtering-records)
   - [ETL Example](#etl-example)
   - [Working with Pandas](#working-with-pandas)
9. [Common Mistakes](#9-common-mistakes)
10. [Performance & Readability](#10-performance--readability)
11. [Quick Reference](#11-quick-reference)
12. [Interview Questions](#12-interview-questions)
13. [Remember](#13-remember)

---

# 1. Why Functions Matter

A **function** is a reusable block of code designed to perform a specific task.

Instead of repeating logic:

```python
price1 = 100
tax1 = price1 * 0.18

price2 = 200
tax2 = price2 * 0.18
```

we can create reusable logic:

```python
def calculate_price_with_tax(price):
    return price * 1.18

print(calculate_price_with_tax(100))
print(calculate_price_with_tax(200))
```

### Why functions are important

Functions provide:

- **Reusability** — write logic once and use it many times.
- **Modularity** — break large programs into smaller components.
- **Readability** — give meaningful names to operations.
- **Testing** — test individual pieces independently.
- **Maintainability** — change logic in one place.
- **Abstraction** — hide implementation details from the caller.

In Data Science and Data Engineering, functions are frequently used for:

```text
Extract → Clean → Transform → Validate → Load
```

---

# 2. Functions

## Basic Function Syntax

```python
def function_name(parameters):
    # function body
    return result
```

Example:

```python
def greet(name):
    return f"Hello, {name}!"
```

---

## Calling a Function

Defining a function does not execute it.

```python
def greet(name):
    return f"Hello, {name}!"

message = greet("Rudrapratap")
print(message)
```

Output:

```text
Hello, Rudrapratap!
```

---

## Parameters vs Arguments

A **parameter** is a variable in the function definition:

```python
def greet(name):
    return f"Hello {name}"
```

Here `name` is a parameter.

An **argument** is the actual value passed during the call:

```python
greet("Rudrapratap")
```

Here `"Rudrapratap"` is an argument.

> **Parameter = placeholder. Argument = actual value.**

---

## Return Values

`return` sends a result back to the caller.

```python
def add(a, b):
    return a + b

result = add(10, 20)
print(result)
```

Output:

```text
30
```

### `return` vs `print`

`print()` displays a value:

```python
def add(a, b):
    print(a + b)
```

`return` gives the value back so other code can use it:

```python
def add(a, b):
    return a + b

total = add(10, 20)
average = total / 3
```

For reusable Data Science and Engineering code, `return` is generally more useful.

---

## Default Arguments

A parameter can have a default value:

```python
def calculate_tax(amount, tax_rate=0.18):
    return amount * tax_rate

print(calculate_tax(1000))
print(calculate_tax(1000, 0.05))
```

Output:

```text
180.0
50.0
```

If `tax_rate` is omitted, Python uses `0.18`.

A common pattern is:

```python
def function(required, optional=10):
    ...
```

---

## Keyword & Positional Arguments

### Positional Arguments

Arguments are matched by position:

```python
def calculate_total(price, quantity):
    return price * quantity

calculate_total(100, 5)
```

Mapping:

```text
price    → 100
quantity → 5
```

### Keyword Arguments

Arguments can be passed using parameter names:

```python
def create_user(name, age, city):
    return f"{name}, {age}, {city}"

user = create_user(
    name="Rudrapratap",
    age=21,
    city="Delhi"
)
```

Keyword arguments improve readability, especially for functions with many parameters.

---

# Variable-Length Arguments

Python provides:

```text
*args
**kwargs
```

when a function needs to accept a variable number of arguments.

---

## `*args`

`*args` collects extra positional arguments into a tuple.

```python
def calculate_sum(*numbers):
    return sum(numbers)

print(calculate_sum(10, 20))
print(calculate_sum(10, 20, 30, 40))
```

Output:

```text
30
100
```

Inside the function:

```python
def show_args(*args):
    print(args)

show_args(10, 20, 30)
```

Output:

```text
(10, 20, 30)
```

> `*args → tuple`

---

## `**kwargs`

`**kwargs` collects extra keyword arguments into a dictionary.

```python
def show_details(**kwargs):
    print(kwargs)

show_details(
    name="Rudrapratap",
    age=21,
    skill="Python"
)
```

Output:

```text
{'name': 'Rudrapratap', 'age': 21, 'skill': 'Python'}
```

> `**kwargs → dictionary`

### Data Engineering Example

```python
def run_pipeline(source, **config):
    print("Source:", source)
    print("Configuration:", config)

run_pipeline(
    "sales.csv",
    delimiter=",",
    encoding="utf-8",
    clean_missing=True
)
```

---

# Positional-Only & Keyword-Only Parameters

Python lets us design more precise function APIs.

## Positional-only

Parameters before `/` must be passed positionally:

```python
def calculate_total(price, quantity, /):
    return price * quantity

calculate_total(100, 5)
```

This is invalid:

```python
calculate_total(price=100, quantity=5)
```

## Keyword-only

Parameters after `*` must be passed by keyword:

```python
def train_model(X, y, *, test_size=0.2):
    ...
```

Valid:

```python
train_model(X, y, test_size=0.2)
```

This is invalid:

```python
train_model(X, y, 0.2)
```

---

# Scope

## Local Variable

Created inside a function:

```python
def calculate():
    x = 10
    return x
```

`x` belongs to the function's local scope.

## Global Variable

Defined outside functions:

```python
tax_rate = 0.18

def calculate_tax(amount):
    return amount * tax_rate
```

Although functions can read global variables, relying heavily on mutable global state can make programs harder to test and maintain. Passing dependencies explicitly is often cleaner.

---

# Docstrings

A **docstring** documents what a function does.

```python
def calculate_profit(buy_price, sell_price, quantity):
    """
    Calculate gross profit from a trade.

    Parameters:
        buy_price: Purchase price per unit.
        sell_price: Selling price per unit.
        quantity: Number of units.

    Returns:
        Gross profit.
    """
    return (sell_price - buy_price) * quantity
```

Docstrings are useful in shared data and engineering projects.

---

# Type Hints

Type hints communicate expected types:

```python
def calculate_profit(
    buy_price: float,
    sell_price: float,
    quantity: int
) -> float:
    return (sell_price - buy_price) * quantity
```

Type hints:

- Improve readability.
- Help IDEs.
- Help static type checkers.
- Make function contracts clearer.

Type hints do not normally enforce types at runtime by themselves.

---

# Functions as Objects

In Python, functions are **first-class objects**.

A function can be:

- Assigned to a variable.
- Passed to another function.
- Returned from another function.
- Stored in a collection.

Example:

```python
def square(x):
    return x ** 2

operation = square

print(operation(5))
```

Output:

```text
25
```

This concept is important because `map()`, `filter()`, sorting, decorators, and many functional programming patterns pass functions around as values.

---

# 3. Lambda Functions

A **lambda function** is a small anonymous function written as a single expression.

It is useful when a short function is needed temporarily.

## Lambda Syntax

```python
lambda arguments: expression
```

Example:

```python
square = lambda x: x ** 2

print(square(5))
```

Output:

```text
25
```

Equivalent normal function:

```python
def square(x):
    return x ** 2
```

A lambda can accept multiple arguments:

```python
multiply = lambda a, b: a * b

print(multiply(5, 4))
```

Output:

```text
20
```

### Important characteristics

A lambda:

- Is an expression.
- Automatically returns its expression's value.
- Can accept multiple arguments.
- Is best for short, simple logic.

---

## Basic Examples

### Celsius to Fahrenheit

```python
to_fahrenheit = lambda c: (c * 9/5) + 32

print(to_fahrenheit(25))
```

Output:

```text
77.0
```

### Check an even number

```python
is_even = lambda x: x % 2 == 0

print(is_even(10))
```

Output:

```text
True
```

### Calculate percentage

```python
percentage = lambda obtained, total: (obtained / total) * 100

print(percentage(85, 100))
```

---

# Lambda with Sorting

A common practical use of lambda is custom sorting.

```python
students = [
    {"name": "A", "marks": 85},
    {"name": "B", "marks": 92},
    {"name": "C", "marks": 78}
]
```

Sort by marks:

```python
students.sort(key=lambda student: student["marks"])
```

Descending:

```python
students.sort(
    key=lambda student: student["marks"],
    reverse=True
)
```

This pattern is common with:

- API responses
- JSON
- Lists of dictionaries
- Records
- Ranking data

---

# Lambda in Data Science

```python
sales = [1000, 1500, 2000, 2500]

discounted = list(
    map(lambda x: x * 0.9, sales)
)

print(discounted)
```

Output:

```text
[900.0, 1350.0, 1800.0, 2250.0]
```

Here lambda defines the transformation:

```text
x → x × 0.9
```

---

# When to Use Lambda

Good use:

```python
sorted(data, key=lambda x: x["score"])
```

If the logic becomes complex, use `def` instead.

> **Rule:** Use lambda for short, obvious, one-expression operations. Use `def` for reusable or complex business logic.

---

# 4. `map()` Function

`map()` applies a function to every element of an iterable.

Conceptually:

```text
Input:
[1, 2, 3, 4]

Function:
x → x²

Output:
[1, 4, 9, 16]
```

## How `map()` Works

Syntax:

```python
map(function, iterable)
```

Example:

```python
def square(x):
    return x ** 2

numbers = [1, 2, 3, 4]

result = map(square, numbers)

print(list(result))
```

Output:

```text
[1, 4, 9, 16]
```

### Important

In Python 3, `map()` returns a **map iterator**, not a list.

Therefore:

```python
result = map(...)
```

often becomes:

```python
list(result)
```

when a list is needed.

---

# Basic Example

```python
numbers = [1, 2, 3, 4, 5]

doubled = list(
    map(lambda x: x * 2, numbers)
)

print(doubled)
```

Output:

```text
[2, 4, 6, 8, 10]
```

---

# Using Lambda with `map()`

```python
numbers = [1, 2, 3, 4, 5]

squares = list(
    map(lambda x: x ** 2, numbers)
)

print(squares)
```

Output:

```text
[1, 4, 9, 16, 25]
```

> **Mental model:** `map()` = **Transform every item.**

---

# Mapping Multiple Iterables

`map()` can process multiple iterables.

```python
prices = [100, 200, 300]
quantities = [2, 3, 4]

totals = list(
    map(
        lambda price, quantity: price * quantity,
        prices,
        quantities
    )
)

print(totals)
```

Output:

```text
[200, 600, 1200]
```

It effectively calculates:

```text
100 × 2
200 × 3
300 × 4
```

`map()` stops when the shortest iterable is exhausted.

---

# `map()` in Data Science & Engineering

## Example 1 — Cleaning Strings

```python
names = [" rudra ", "rahul ", " PRIYA"]

clean_names = list(
    map(lambda name: name.strip().title(), names)
)

print(clean_names)
```

Output:

```text
['Rudra', 'Rahul', 'Priya']
```

## Example 2 — Converting Data Types

```python
raw_values = ["10", "20", "30", "40"]

values = list(map(int, raw_values))

print(values)
```

Output:

```text
[10, 20, 30, 40]
```

This is common when processing values from:

- CSV files
- APIs
- Text files
- User input

## Example 3 — Feature Transformation

```python
income = [25000, 40000, 60000, 80000]

income_in_lakhs = list(
    map(lambda x: x / 100000, income)
)
```

## Example 4 — ETL Transformation

```python
transactions = [
    {"amount": "1000"},
    {"amount": "2500"},
    {"amount": "1750"}
]

cleaned = list(
    map(
        lambda row: {
            **row,
            "amount": float(row["amount"])
        },
        transactions
    )
)

print(cleaned)
```

Output:

```text
[
    {"amount": 1000.0},
    {"amount": 2500.0},
    {"amount": 1750.0}
]
```

---

# 5. `filter()` Function

`filter()` selects elements from an iterable for which a function returns `True`.

Conceptually:

```text
Input:
[1, 2, 3, 4, 5, 6]

Condition:
Is number even?

Output:
[2, 4, 6]
```

## How `filter()` Works

Syntax:

```python
filter(function, iterable)
```

Example:

```python
def is_even(x):
    return x % 2 == 0

numbers = [1, 2, 3, 4, 5, 6]

result = filter(is_even, numbers)

print(list(result))
```

Output:

```text
[2, 4, 6]
```

Like `map()`, `filter()` returns an iterator in Python 3.

---

# Basic Example

```python
numbers = [1, 2, 3, 4, 5, 6]

even_numbers = list(
    filter(lambda x: x % 2 == 0, numbers)
)

print(even_numbers)
```

Output:

```text
[2, 4, 6]
```

> **Mental model:** `filter()` = **Keep only items that satisfy a condition.**

---

# Using Lambda with `filter()`

```python
numbers = [10, 15, 20, 25, 30]

large_numbers = list(
    filter(lambda x: x >= 20, numbers)
)

print(large_numbers)
```

Output:

```text
[20, 25, 30]
```

---

# `filter()` in Data Science & Engineering

## Example 1 — Filter Valid Transactions

```python
transactions = [
    {"amount": 1000, "status": "success"},
    {"amount": 500, "status": "failed"},
    {"amount": 2000, "status": "success"}
]

valid = list(
    filter(
        lambda row: row["status"] == "success",
        transactions
    )
)

print(valid)
```

## Example 2 — Filter High-Value Transactions

```python
transactions = [
    {"amount": 1000},
    {"amount": 5000},
    {"amount": 2500},
    {"amount": 10000}
]

high_value = list(
    filter(lambda row: row["amount"] >= 5000, transactions)
)
```

## Example 3 — Remove Empty Values

```python
values = ["Python", "", "SQL", None, "Pandas", ""]

clean_values = list(filter(None, values))

print(clean_values)
```

Output:

```text
['Python', 'SQL', 'Pandas']
```

`filter(None, iterable)` keeps truthy values, so it removes all falsy values such as `None`, `False`, `0`, `""`, `[]`, and `{}`.

---

# 6. Combining Lambda, `map()`, and `filter()`

Suppose:

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8]
```

Goal:

```text
1. Keep even numbers
2. Square them
```

First filter:

```python
even_numbers = filter(
    lambda x: x % 2 == 0,
    numbers
)
```

Then map:

```python
squares = map(
    lambda x: x ** 2,
    even_numbers
)

print(list(squares))
```

Output:

```text
[4, 16, 36, 64]
```

Combined:

```python
result = list(
    map(
        lambda x: x ** 2,
        filter(lambda x: x % 2 == 0, numbers)
    )
)
```

Read it from the inside out:

```text
Original Data
     ↓
filter()
     ↓
Keep desired records
     ↓
map()
     ↓
Transform records
     ↓
Result
```

---

# 7. List Comprehension vs `map()` vs `filter()`

Many Python tasks can be solved with comprehensions or functional tools.

## Transformation

Using `map()`:

```python
numbers = [1, 2, 3, 4]

result = list(map(lambda x: x ** 2, numbers))
```

Using list comprehension:

```python
result = [x ** 2 for x in numbers]
```

For simple transformations, list comprehensions are often more readable.

## Filtering

Using `filter()`:

```python
result = list(
    filter(lambda x: x % 2 == 0, numbers)
)
```

Using list comprehension:

```python
result = [x for x in numbers if x % 2 == 0]
```

## Combined

```python
result = [
    x ** 2
    for x in numbers
    if x % 2 == 0
]
```

This is often very readable.

### When `map()` / `filter()` are useful

They are useful when:

- Passing an existing named function.
- Building functional pipelines.
- Working with iterators.
- The operation naturally reads as "map this function" or "filter by this predicate."

Example:

```python
names = ["alice", "bob", "charlie"]

result = list(map(str.upper, names))
```

---

# 8. Practical Data Science Examples

# Cleaning Data

Suppose raw values arrive from an API:

```python
raw_names = [
    " rudra ",
    "RAHUL",
    " priya",
    "",
    None
]
```

Filter invalid values:

```python
valid_names = filter(
    lambda x: x is not None and x.strip() != "",
    raw_names
)
```

Then clean them:

```python
clean_names = list(
    map(
        lambda x: x.strip().title(),
        valid_names
    )
)

print(clean_names)
```

Output:

```text
['Rudra', 'Rahul', 'Priya']
```

Pipeline:

```text
Raw Data
   ↓
filter()
   ↓
Remove invalid records
   ↓
map()
   ↓
Transform valid records
   ↓
Clean Data
```

---

# Feature Transformation

```python
ages = [18, 21, 25, 30, 40]

age_squared = list(
    map(lambda age: age ** 2, ages)
)
```

Normalize values:

```python
values = [10, 20, 30, 40, 50]

normalized = list(
    map(lambda x: x / 50, values)
)
```

For large numerical datasets, NumPy/Pandas vectorized operations are often preferable because they are designed for efficient tabular and numerical processing.

---

# Filtering Records

```python
customers = [
    {"name": "A", "age": 17},
    {"name": "B", "age": 25},
    {"name": "C", "age": 32},
    {"name": "D", "age": 15}
]

adults = list(
    filter(lambda customer: customer["age"] >= 18, customers)
)

print(adults)
```

Output:

```text
[
    {"name": "B", "age": 25},
    {"name": "C", "age": 32}
]
```

---

# ETL Example

Suppose raw sales data arrives as:

```python
raw_sales = [
    {"product": "Laptop", "price": "50000", "status": "completed"},
    {"product": "Mouse", "price": "1000", "status": "cancelled"},
    {"product": "Keyboard", "price": "2500", "status": "completed"}
]
```

### Step 1 — Filter completed orders

```python
completed = filter(
    lambda row: row["status"] == "completed",
    raw_sales
)
```

### Step 2 — Transform price

```python
transformed = map(
    lambda row: {
        **row,
        "price": float(row["price"])
    },
    completed
)
```

### Step 3 — Materialize

```python
sales = list(transformed)

print(sales)
```

Output:

```text
[
    {"product": "Laptop", "price": 50000.0, "status": "completed"},
    {"product": "Keyboard", "price": 2500.0, "status": "completed"}
]
```

This is a simplified example of the transformation stages commonly found in ETL pipelines.

---

# Working with Pandas

In Data Science, these ideas are often implemented with Pandas.

```python
import pandas as pd

df = pd.DataFrame({
    "name": ["alice", "bob", "charlie"],
    "age": [20, 17, 25]
})
```

### Pandas `map()`

```python
df["name"] = df["name"].map(str.title)
```

### Pandas filtering

```python
adults = df[df["age"] >= 18]
```

### Pandas `apply()`

```python
df["age_squared"] = df["age"].apply(
    lambda x: x ** 2
)
```

### Important distinction

Python's:

```python
map()
filter()
```

are general Python built-ins.

Pandas provides methods such as:

```python
Series.map()
Series.apply()
DataFrame.apply()
```

They are related concepts but are not the same API.

For large tabular data, prefer vectorized Pandas/NumPy operations when they express the transformation clearly.

---

# 9. Common Mistakes

## Mistake 1 — Forgetting `map()` Returns an Iterator

```python
result = map(lambda x: x * 2, [1, 2, 3])

print(result)
```

You will see a map object representation rather than the values.

Use:

```python
print(list(result))
```

---

## Mistake 2 — Forgetting `return`

Wrong:

```python
def square(x):
    x ** 2
```

Correct:

```python
def square(x):
    return x ** 2
```

---

## Mistake 3 — Using Lambda for Complex Logic

Avoid turning a lambda into a difficult-to-read mini-program.

Use:

```python
def calculate_customer_score(customer):
    # complex logic
    ...
```

Use `def` when the logic deserves a meaningful name.

---

## Mistake 4 — Confusing `map()` and `filter()`

Remember:

```text
map()
→ Transform items

filter()
→ Select items
```

---

## Mistake 5 — Expecting `filter()` to Transform Values

This:

```python
filter(lambda x: x * 2, numbers)
```

does **not** mean "double every number."

`filter()` checks whether the returned value is truthy.

To transform:

```python
map(lambda x: x * 2, numbers)
```

---

# 10. Performance & Readability

## Functions

Functions improve maintainability by separating responsibilities:

```python
def extract_data():
    ...

def clean_data(data):
    ...

def transform_data(data):
    ...

def validate_data(data):
    ...

def load_data(data):
    ...
```

This structure is useful in data pipelines.

---

## `map()` and `filter()` Are Lazy

Python 3's `map()` and `filter()` return iterators.

For example:

```python
result = map(lambda x: x * 2, range(1_000_000))
```

The result can be consumed progressively.

When you write:

```python
list(result)
```

you materialize the values into a list.

This distinction matters when working with large sequences.

---

## Don't Optimize Prematurely

A useful priority is:

```text
Correctness
   ↓
Readability
   ↓
Maintainability
   ↓
Performance optimization
```

For large-scale numerical Data Science workloads, vectorized NumPy/Pandas operations or specialized processing frameworks are often more appropriate than repeated Python-level function calls.

---

# 11. Quick Reference

## Function

```python
def function_name(parameter):
    return result
```

## Lambda

```python
lambda x: expression
```

Example:

```python
square = lambda x: x ** 2
```

## `map()`

> **Transform every item.**

```python
map(function, iterable)
```

Example:

```python
list(map(lambda x: x * 2, numbers))
```

## `filter()`

> **Keep items that satisfy a condition.**

```python
filter(function, iterable)
```

Example:

```python
list(filter(lambda x: x > 10, numbers))
```

## `*args`

```python
def function(*args):
    ...
```

```text
Extra positional arguments → tuple
```

## `**kwargs`

```python
def function(**kwargs):
    ...
```

```text
Extra keyword arguments → dictionary
```

## Core Comparison

| Feature | Function | Lambda | `map()` | `filter()` |
|---|---|---|---|---|
| Purpose | Reusable logic | Small anonymous function | Transform items | Select items |
| Returns | Depends on function | Expression result | Map iterator | Filter iterator |
| Typical use | General logic | Short operations | Transformation | Selection |
| Multiple inputs | Yes | Yes | Yes, multiple iterables | Usually one iterable |
| Common DS use | ETL, features, validation | Sorting/transformation | Data transformation | Data filtering |

---

# 12. Interview Questions

## Q1. What is a function?

A function is a reusable block of code designed to perform a specific task. It can accept inputs through parameters and return a result.

## Q2. What is the difference between a parameter and an argument?

A parameter is a variable in the function definition, while an argument is the actual value supplied when calling the function.

## Q3. What is a lambda function?

A lambda is a small anonymous function consisting of a single expression. It is useful for short operations where defining a full named function would be unnecessary.

## Q4. What is `map()`?

`map()` applies a function to each element of an iterable and returns a map iterator.

## Q5. What is `filter()`?

`filter()` returns an iterator containing elements for which the supplied function evaluates to `True`.

## Q6. What is the difference between `map()` and `filter()`?

```text
map()
→ transforms

filter()
→ selects
```

Example:

```python
list(map(lambda x: x * 2, numbers))
```

transforms every value.

```python
list(filter(lambda x: x > 10, numbers))
```

keeps only values greater than 10.

## Q7. Why does `map()` return an iterator instead of a list?

Python 3 uses lazy iteration for `map()` and `filter()`. Values can be produced as they are consumed instead of necessarily creating the complete output collection immediately.

## Q8. When should you use a normal function instead of lambda?

Use a normal function when the logic is complex, reused, requires documentation, or deserves a meaningful name.

## Q9. What are `*args` and `**kwargs`?

`*args` collects additional positional arguments into a tuple, while `**kwargs` collects additional keyword arguments into a dictionary.

## Q10. Are Python's `map()` and Pandas' `.map()` the same?

No. Python's `map()` is a built-in function that operates on iterables. Pandas' `.map()` is a Series method designed for element-wise mapping on Pandas data.

---

# 13. Remember

```text
FUNCTION
    ↓
Reusable block of logic

LAMBDA
    ↓
Small anonymous function

MAP
    ↓
Transform every item

FILTER
    ↓
Keep items that satisfy a condition
```

## Simple Mental Model

```text
                 FUNCTION
                    │
        ┌───────────┴───────────┐
        │                       │
      named                  lambda
        │                       │
   reusable/complex        short/simple
                                │
                    ┌───────────┴───────────┐
                    │                       │
                  map()                  filter()
                    │                       │
               TRANSFORM                 SELECT
                    │                       │
              every item             matching items
```

### The Most Important Example

```python
numbers = [1, 2, 3, 4, 5, 6]

result = list(
    map(
        lambda x: x ** 2,
        filter(lambda x: x % 2 == 0, numbers)
    )
)

print(result)
```

Output:

```text
[4, 16, 36]
```

Read it from the inside out:

```text
numbers
   ↓
filter()
   ↓
keep even numbers
   ↓
[2, 4, 6]
   ↓
map()
   ↓
square each number
   ↓
[4, 16, 36]
```

> **Remember:** `filter()` decides **what stays**. `map()` decides **what each item becomes**.

---

## Related Python Concepts

Connect these concepts with:

- List Comprehensions
- Dictionary Comprehensions
- Generators
- Iterators
- Decorators
- Higher-Order Functions
- `functools`
- `itertools`
- Exception Handling
- Object-Oriented Programming
- NumPy Vectorization
- Pandas `map()` / `apply()`
- ETL Pipelines
