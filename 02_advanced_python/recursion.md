# Python Recursion

> **Purpose:** A practical guide to understanding, tracing, writing, and optimizing recursive Python functions, with algorithms, real-world examples, pitfalls, and revision questions.

## Table of Contents

1. [What Is Recursion?](#1-what-is-recursion)
2. [Base Case, Recursive Case, and Progress](#2-base-case-recursive-case-and-progress)
3. [The Call Stack and Return Values](#3-the-call-stack-and-return-values)
4. [Designing and Proving a Recursive Solution](#4-designing-and-proving-a-recursive-solution)
5. [Types of Recursion](#5-types-of-recursion)
6. [Foundational Examples](#6-foundational-examples)
7. [Time and Space Complexity](#7-time-and-space-complexity)
8. [Fibonacci, Memoization, and Dynamic Programming](#8-fibonacci-memoization-and-dynamic-programming)
9. [Divide and Conquer](#9-divide-and-conquer)
10. [Trees and Graphs](#10-trees-and-graphs)
11. [Backtracking](#11-backtracking)
12. [Nested Data and Recursive Generators](#12-nested-data-and-recursive-generators)
13. [Recursion Limits and Iterative Alternatives](#13-recursion-limits-and-iterative-alternatives)
14. [Common Mistakes](#14-common-mistakes)
15. [Debugging and Testing](#15-debugging-and-testing)
16. [Practical Uses and Choosing an Approach](#16-practical-uses-and-choosing-an-approach)
17. [Practice Problems](#17-practice-problems)
18. [Interview Questions and Quick Reference](#18-interview-questions-and-quick-reference)

---

## 1. What Is Recursion?

**Recursion** occurs when a function calls itself, directly or through other functions, to solve a problem in terms of smaller or simpler instances of that problem.

For example, factorial is defined for nonnegative integers as:

```text
0! = 1
n! = n × (n - 1)!    for n > 0

4! = 4 × 3! = 4 × 3 × 2 × 1 = 24
```

A recursive definition describes both how to reduce the problem and when to stop.

Recursion is especially useful when the **data itself has a recursive structure**: a directory contains directories, a tree contains smaller trees, and a JSON object can contain other objects.

## 2. Base Case, Recursive Case, and Progress

Every terminating recursive algorithm needs:

| Component | Role | Factorial example |
|---|---|---|
| Base case | Solves a simplest instance without another recursive call | `n == 0` returns `1` |
| Recursive case | Uses results from simpler instances | `n * factorial(n - 1)` |
| Progress toward termination | Ensures calls eventually reach a base case | `n` decreases by one |

```python
def factorial(n):
    """Return n! for a nonnegative integer; reject bool inputs."""
    if isinstance(n, bool) or not isinstance(n, int):
        raise TypeError("n must be an integer")
    if n < 0:
        raise ValueError("n must be nonnegative")
    if n == 0:
        return 1
    return n * factorial(n - 1)


assert factorial(0) == 1
assert factorial(5) == 120
```

The validation is repeated on each call for simplicity. In larger APIs, validate once in a public function and recurse through a private helper.

**A base case alone does not guarantee termination.** Calling `factorial(n + 1)` would move away from the base case. For graphs, progress can mean visiting a previously unseen node rather than decreasing a number.

## 3. The Call Stack and Return Values

Each active function call has its own execution frame, including local variables and the point at which execution will resume. A caller waits for the called function to finish.

Tracing `factorial(3)`:

```text
Calls descend:                     Returns unwind:
factorial(3) → 3 * factorial(2)     factorial(3) returns 3 * 2 = 6
factorial(2) → 2 * factorial(1)     factorial(2) returns 2 * 1 = 2
factorial(1) → 1 * factorial(0)     factorial(1) returns 1 * 1 = 1
factorial(0) → 1                   factorial(0) returns 1
```

The returns occur from the bottom of the call chain upward. The multiplication happens **after** the deeper call returns.

### Work before and after a recursive call

```python
def show_order(n):
    if n <= 0:
        return
    print("Entering", n)
    show_order(n - 1)
    print("Leaving", n)


show_order(3)
# Entering 3
# Entering 2
# Entering 1
# Leaving 1
# Leaving 2
# Leaving 3
```

Each frame has its own `n`. However, if multiple frames receive the same list or dictionary, they refer to the **same mutable object** unless you explicitly copy it.

`print()` displays something; `return` passes a value to the caller. A function that reaches its end without returning a value returns `None`.

## 4. Designing and Proving a Recursive Solution

Use this sequence:

1. **Define the contract:** What inputs are valid, and what does one call return?
2. **Choose the smallest cases:** Include empty inputs and boundary values where relevant.
3. **Reduce the problem:** Determine which smaller instance or instances to solve.
4. **Combine their results:** Assume the smaller calls correctly follow the same contract.
5. **Check termination:** Identify something that strictly decreases or a finite set of states being consumed.
6. **Analyze cost:** Count total calls, maximum depth, and allocations.

For factorial, a correctness argument resembles mathematical induction:

- **Base:** The function returns `1` for `0`, which equals `0!`.
- **Step:** If the call on `n - 1` returns `(n - 1)!`, multiplying by `n` returns `n!`.
- **Termination:** A nonnegative integer decreases until it reaches zero.

Mathematical termination and practical execution limits are separate: a valid problem may still require more recursion depth than Python permits.

## 5. Types of Recursion

| Type | Meaning | Example |
|---|---|---|
| Direct | A function calls itself | Factorial |
| Indirect or mutual | Functions call one another in a cycle | `is_even` and `is_odd` |
| Linear | At most one recursive call per invocation | Recursive sum |
| Branching or tree recursion | An invocation makes multiple recursive calls | Naive Fibonacci |
| Tail recursion | The recursive call is the final operation; its result is returned directly | Accumulator factorial |
| Non-tail recursion | Work remains after the recursive call returns | Ordinary factorial |
| Nested recursion | A recursive result becomes the argument of another recursive call | Some mathematical recursive definitions |

These categories overlap. A function can be both direct and linear, for example.

### Mutual recursion

These examples assume a nonnegative integer input:

```python
def is_even(n):
    if n == 0:
        return True
    return is_odd(n - 1)


def is_odd(n):
    if n == 0:
        return False
    return is_even(n - 1)


assert is_even(6) is True
assert is_odd(7) is True
```

Use `n % 2 == 0` in ordinary application code; this example demonstrates mutual calls.

### Tail recursion

```python
def factorial_tail(n, accumulator=1):
    # Assumes n is a nonnegative integer.
    if n == 0:
        return accumulator
    return factorial_tail(n - 1, accumulator * n)


assert factorial_tail(5) == 120
```

There is no multiplication pending after the recursive call. Nevertheless, **CPython does not eliminate tail calls**, so this still uses `O(n)` call-stack space. An accumulator alone does not make recursive Python code use constant stack space.

## 6. Foundational Examples

Unless explicitly validated, the following examples assume inputs satisfy their documented conditions.

### Sum a sequence without slicing

```python
def recursive_sum(values):
    def helper(index):
        if index == len(values):
            return 0
        return values[index] + helper(index + 1)

    return helper(0)


assert recursive_sum([]) == 0
assert recursive_sum([2, 4, 6]) == 12
```

Time: `O(n)`. Stack: `O(n)`. Use Python's `sum(values)` for routine summation.

### Palindrome check using two indices

```python
def is_palindrome(text):
    """Compare characters exactly: case and whitespace are significant."""
    def helper(left, right):
        if left >= right:
            return True
        if text[left] != text[right]:
            return False
        return helper(left + 1, right - 1)

    return helper(0, len(text) - 1)


assert is_palindrome("") is True
assert is_palindrome("radar") is True
assert is_palindrome("python") is False
```

Worst-case time and stack: `O(n)`. Indices avoid allocating a shorter string at every call.

### Greatest common divisor: Euclid's algorithm

```python
def gcd(a, b):
    """Return a nonnegative GCD for integer inputs."""
    a, b = abs(a), abs(b)
    if b == 0:
        return a
    return gcd(b, a % b)


assert gcd(48, 18) == 6
assert gcd(-48, 18) == 6
assert gcd(0, 0) == 0
```

For positive inputs, Euclid's algorithm uses `O(log(min(a, b)))` recursive steps. This counts arithmetic operations, not the bit-level cost of large integer division. In application code, use `math.gcd`.

### Fast exponentiation

```python
def power(base, exponent):
    """Compute base ** exponent for a nonnegative integer exponent."""
    if isinstance(exponent, bool) or not isinstance(exponent, int):
        raise TypeError("exponent must be an integer")
    if exponent < 0:
        raise ValueError("exponent must be nonnegative")
    if exponent == 0:
        return 1
    half = power(base, exponent // 2)
    if exponent % 2 == 0:
        return half * half
    return base * half * half


assert power(2, 10) == 1024
assert power(7, 0) == 1
```

Only one recursive call is made per level. For a positive exponent `e`, the algorithm uses `O(log e)` multiplications and stack depth. Big integer multiplication is not constant time. Prefer `**` or `pow()` for normal use.

## 7. Time and Space Complexity

**Total calls determine much of the time cost; maximum simultaneous call depth determines stack space.** They are different quantities.

| Algorithm | Time under unit-cost operations | Auxiliary space |
|---|---|---|
| Factorial | `O(n)` multiplications | `O(n)` stack frames |
| Recursive sum | `O(n)` | `O(n)` stack |
| Binary search using indices | `O(log n)` | `O(log n)` stack |
| Naive Fibonacci | `O(2^n)` upper bound | `O(n)` stack |
| Memoized Fibonacci | `O(n)` arithmetic operations | `O(n)` cache entries and stack |
| Merge sort shown below | `O(n log n)` | `O(n)` peak auxiliary storage |
| Tree traversal | `O(n)` | `O(h)` stack for height `h`, excluding output |
| Graph DFS with a visited set | `O(V + E)` | `O(V)` including visited set and stack |

These estimates assume constant-time element operations and, for graph DFS, expected constant-time set operations. Python integers have arbitrary precision, so large numeric values require more time and memory than the unit-cost model captures.

### Recurrence relations

A recurrence expresses running time in terms of smaller inputs:

```text
Factorial:        T(n) = T(n - 1) + O(1)          → O(n)
Binary search:    T(n) = T(n / 2) + O(1)          → O(log n)
Merge sort:       T(n) = 2T(n / 2) + O(n)         → O(n log n)
Naive Fibonacci:  T(n) = T(n - 1) + T(n - 2) + O(1)
```

Naive Fibonacci has a tighter bound of `Θ(φ^n)`, where `φ ≈ 1.618`; `O(2^n)` is a simpler upper bound. The Master Theorem applies to suitable recurrences of the form `aT(n/b) + f(n)`, not directly to every recursive algorithm.

### Hidden cost of slicing

```python
def sum_with_slices(values):
    if not values:
        return 0
    return values[0] + sum_with_slices(values[1:])
```

For a Python list, each `values[1:]` allocates a new list of references. Across this call chain, copying costs `O(n²)` time. The retained lists can also use `O(n²)` peak storage. Passing an index avoids those copies.

## 8. Fibonacci, Memoization, and Dynamic Programming

The Fibonacci sequence is defined by `F(0) = 0`, `F(1) = 1`, and `F(n) = F(n - 1) + F(n - 2)` for `n >= 2`.

### Naive recursive version

```python
def fibonacci_naive(n):
    # Assumes n is a nonnegative integer.
    if n < 2:
        return n
    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)


assert fibonacci_naive(10) == 55
```

The same subproblems are computed repeatedly:

```text
fib(5)
├── fib(4)
│   ├── fib(3)
│   └── fib(2)
└── fib(3)          ← repeated work
    ├── fib(2)      ← repeated work
    └── fib(1)
```

### Memoization: top-down dynamic programming

Memoization saves the result for an input and reuses it on subsequent calls.

```python
from functools import lru_cache


@lru_cache(maxsize=None)
def fibonacci_memo(n):
    # Assumes n is a nonnegative integer.
    if n < 2:
        return n
    return fibonacci_memo(n - 1) + fibonacci_memo(n - 2)


assert fibonacci_memo(30) == 832040
# Inspect reuse with fibonacci_memo.cache_info().
# Release stored entries with fibonacci_memo.cache_clear().
```

`maxsize=None` creates an unbounded cache. `functools.cache` is another unbounded memoization decorator, available in Python 3.9 and later.

Important conditions:

- Arguments must be hashable; lists and dictionaries cannot be used directly as cache keys.
- Cached results should depend on the arguments, not changing external state.
- A cached mutable return value is shared; modifying it can affect later callers.
- Memoization removes repeated work but **does not remove the initial recursive call chain**.
- An unbounded cache can grow over time; consider its lifetime and clear it when appropriate.

### Tabulation and iterative state compression

Bottom-up dynamic programming computes smaller states first. A full table stores all values; Fibonacci only needs the previous two:

```python
def fibonacci_iterative(n):
    # Assumes n is a nonnegative integer.
    current, following = 0, 1
    for _ in range(n):
        current, following = following, current + following
    return current


assert fibonacci_iterative(30) == 832040
```

This uses `O(n)` additions, constant recursion depth, and a constant number of integer variables. Their bit sizes grow with `n`, so actual byte usage is not constant.

## 9. Divide and Conquer

Divide and conquer splits a problem into smaller subproblems, solves them, and combines their results. Unlike naive Fibonacci, merge sort's subproblems do not overlap.

### Recursive binary search

**Precondition:** The sequence is sorted in ascending order and its elements support comparison with the target. The search interval below is inclusive at both ends.

```python
def binary_search(values, target):
    def search(left, right):
        if left > right:
            return -1
        middle = (left + right) // 2
        if values[middle] == target:
            return middle
        if target < values[middle]:
            return search(left, middle - 1)
        return search(middle + 1, right)

    return search(0, len(values) - 1)


assert binary_search([2, 4, 6, 8, 10], 8) == 3
assert binary_search([2, 4, 6], 5) == -1
assert binary_search([], 5) == -1
```

With duplicates, this returns a matching index, not necessarily the first or last. Passing indices preserves logarithmic work; slicing the sequence at each step adds copying overhead.

### Merge sort

```python
def merge_sort(values):
    """Return a new sorted list; leave the input unchanged."""
    if len(values) <= 1:
        return values[:]

    middle = len(values) // 2
    left = merge_sort(values[:middle])
    right = merge_sort(values[middle:])
    merged = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1

    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged


assert merge_sort([5, 2, 4, 2, 1]) == [1, 2, 2, 4, 5]
assert merge_sort([]) == []
```

Choosing from `left` when values tie makes this implementation stable. It takes `O(n log n)` time and `O(n)` peak auxiliary storage, including temporary lists; the stack itself is `O(log n)`. Total allocation over the full run can exceed peak simultaneously live storage. Use `sorted()` or `list.sort()` for normal Python sorting.

## 10. Trees and Graphs

### Binary tree traversal

```python
class Node:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right


def inorder(root):
    result = []

    def visit(node):
        if node is None:
            return
        visit(node.left)
        result.append(node.value)
        visit(node.right)

    visit(root)
    return result


tree = Node(2, Node(1), Node(3))
assert inorder(tree) == [1, 2, 3]
assert inorder(None) == []
```

| Traversal | Processing order | Typical use |
|---|---|---|
| Preorder | Node, left, right | Serialize structure with suitable null markers |
| Inorder | Left, node, right | Read a binary search tree in sorted order |
| Postorder | Left, right, node | Compute a parent result from child results |
| Level order | One depth level at a time | Breadth-first search using a queue |

Inorder output is sorted only when the tree satisfies the binary search tree ordering rules. Recursive DFS needs `O(h)` stack space: `O(log n)` for a balanced tree and `O(n)` for a skewed tree. The returned result adds `O(n)` space.

### Graph depth-first search with cycle protection

```python
def dfs(graph, start):
    """Visit vertices reachable from start, following neighbor order."""
    visited = set()
    order = []

    def visit(node):
        if node in visited:
            return
        visited.add(node)  # Mark before exploring neighbors.
        order.append(node)
        for neighbor in graph.get(node, ()):
            visit(neighbor)

    visit(start)
    return order


graph = {"A": ["B", "C"], "B": ["A", "D"], "C": ["D"], "D": []}
assert dfs(graph, "A") == ["A", "B", "D", "C"]
```

Vertices must be hashable. This traverses the reachable component; traverse additional unvisited roots to cover a disconnected graph. For reachable vertices and edges, time is `O(V + E)` and auxiliary space is `O(V)`.

A visited set prevents repeated traversal. It does **not** by itself distinguish an actual cycle from a shared descendant. Detecting directed cycles typically also tracks nodes on the active DFS path.

## 11. Backtracking

Backtracking explores choices recursively and reverses a choice before trying another branch:

```text
choose → explore → undo
```

Recursion provides the call structure. Backtracking is the search strategy, and it can also be implemented with an explicit stack.

### Generate all subsets

```python
def subsets(items):
    """Return subsets by position; duplicate inputs can produce equal subsets."""
    result = []
    path = []

    def explore(index):
        if index == len(items):
            result.append(path.copy())
            return

        explore(index + 1)       # Exclude this item.
        path.append(items[index])
        explore(index + 1)       # Include this item.
        path.pop()               # Restore the earlier state.

    explore(0)
    return result


assert subsets([1, 2]) == [[], [2], [1], [1, 2]]
assert subsets([]) == [[]]
```

There are `2^n` subsets by position. Copying the selected elements into the output takes `O(n × 2^n)` total time and output space in the worst case. The active path and call stack use `O(n)` auxiliary space, excluding output.

**Why copy?** Appending `path` itself would store repeated references to one changing list. A shallow copy is sufficient to preserve the chosen sequence of references; it does not clone mutable elements inside it.

### Pruning

Pruning stops exploring branches that cannot produce valid results. In N-Queens, reject a placement as soon as it conflicts with an earlier queen. In a sum search, pruning when the partial sum exceeds the target is valid only under assumptions such as nonnegative remaining values.

Other applications include permutations, maze search, Sudoku, combinations, and constraint satisfaction. Memoization helps only when the stored state fully captures the information needed for future choices.

## 12. Nested Data and Recursive Generators

### Flatten nested lists lazily

```python
def flatten(items):
    """Yield leaves from an acyclic list structure; only lists are expanded."""
    for item in items:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item


assert list(flatten([1, [2, [3, 4]], [], 5])) == [1, 2, 3, 4, 5]
assert list(flatten(["abc", [None]])) == ["abc", None]
```

`yield from` delegates iteration to the inner generator. Creating a generator does not run its entire body; work happens as the consumer requests values. It avoids materializing the flattened output unless the caller uses `list(...)`.

Traversal takes linear time in the total entries visited, including nested containers. Active generator state grows with nesting depth. Recursive generators still have depth limits. A self-referential list needs cycle handling; this example assumes no cycles.

### Walk JSON-like data with explicit paths

```python
def walk_leaves(value, path=()):
    """Yield (path_tuple, leaf) pairs from acyclic dictionaries and lists."""
    if isinstance(value, dict):
        for key, child in value.items():
            yield from walk_leaves(child, path + (key,))
    elif isinstance(value, list):
        for index, child in enumerate(value):
            yield from walk_leaves(child, path + (index,))
    else:
        yield path, value


record = {"user": {"name": "Asha"}, "scores": [8, 9]}
assert list(walk_leaves(record)) == [
    (("user", "name"), "Asha"),
    (("scores", 0), 8),
    (("scores", 1), 9),
]
```

Tuple paths preserve dictionary keys and list indices without ambiguity from dots in keys. Empty containers produce no leaves. Copying path tuples adds work proportional to path length, so this version is not strictly linear for arbitrary nesting. It is useful for inspecting API responses, locating missing values, and preparing nested records for transformation.

## 13. Recursion Limits and Iterative Alternatives

Python limits recursion depth to protect the interpreter from excessive stack use. Exceeding the permitted depth raises `RecursionError`.

```python
import sys

print(sys.getrecursionlimit())
```

Do not rely on a fixed numeric default or expect exactly that many calls in your own function: existing calls and wrappers also contribute to execution depth. `sys.setrecursionlimit(...)` changes the limit, but it does not improve the algorithm or make arbitrarily deep recursion safe. An excessively high limit can crash the process on some platforms.

### Replace linear recursion with a loop

```python
def factorial_iterative(n):
    # Assumes n is a nonnegative integer.
    result = 1
    for value in range(2, n + 1):
        result *= value
    return result


assert factorial_iterative(5) == 120
```

This uses a constant number of variables and no growing recursive stack. The integer result itself still grows in size.

### Replace recursive graph traversal with an explicit stack

```python
def dfs_iterative(graph, start):
    visited = set()
    order = []
    stack = [start]

    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        order.append(node)
        # Assumes adjacency values are lists or other reversible sequences.
        stack.extend(reversed(graph.get(node, ())))

    return order


assert dfs_iterative({"A": ["B", "C"], "B": [], "C": []}, "A") == ["A", "B", "C"]
```

Reversing neighbors preserves the recursive left-to-right visitation order because a stack is last-in, first-out. This simple version can place a vertex on the stack more than once before it is visited, so pending entries can require `O(V + E)` space. More careful implementations can track scheduled vertices or store frames with neighbor iterators.

For algorithms that need work **after** a child finishes, an iterative conversion must also represent that pending work, for example with `(node, expanded)` entries or explicit iterator frames.

Advanced techniques such as trampolines represent the next call as data and execute it in a loop. They can avoid recursive stack growth but add machinery; an ordinary loop or explicit stack is usually clearer.

## 14. Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| Missing or unreachable base case | Calls continue until `RecursionError` | Check termination on the smallest valid inputs |
| Recursive argument does not progress | Same state repeats | Decrease a measure or consume unvisited states |
| Missing `return` | Caller receives `None` | Return the recursive result when the contract requires it |
| Confusing printing with returning | Output appears but cannot be used as a value | Separate computation and display |
| Repeated subproblems | Exponential work | Memoize or use bottom-up dynamic programming |
| Slicing each call | Hidden time and memory costs | Pass bounds or indices |
| Mutable default argument | State leaks across independent calls | Use `None` and initialize inside, or use a wrapper |
| Saving the same mutable path | Results all refer to one changing object | Save `path.copy()` |
| Forgetting to undo a choice | Later branches inherit incorrect state | Restore state after exploration |
| Graph traversal without a visited set | Cycles cause repeated calls | Mark before exploring neighbors |
| Assuming tail-call optimization | Unexpected depth errors | Use a loop when depth may be large |
| Caching a function with external dependencies | Stale or incorrect results | Include relevant state in the key or avoid caching |

### Missing return: a subtle failure

```python
def broken_countdown(n):
    if n == 0:
        return "done"
    broken_countdown(n - 1)  # The result is discarded.


assert broken_countdown(2) is None


def countdown(n):
    # Assumes n is a nonnegative integer.
    if n == 0:
        return "done"
    return countdown(n - 1)


assert countdown(2) == "done"
```

Discarding a return value is fine for functions intentionally used for side effects, such as the inner tree visitor that appends to a shared result list.

## 15. Debugging and Testing

Start with a tiny input and trace the arguments, base-case decision, and returned result at each level.

```python
def traced_factorial(n, depth=0):
    # Assumes n is a nonnegative integer.
    indent = "  " * depth
    print(f"{indent}call factorial({n})")
    if n == 0:
        result = 1
    else:
        result = n * traced_factorial(n - 1, depth + 1)
    print(f"{indent}return {result}")
    return result


assert traced_factorial(3) == 6
```

For reusable recursive functions, check:

- The base case, empty input, and smallest non-base input.
- A typical input and invalid inputs where validation is promised.
- Repeated independent calls to reveal shared-state bugs.
- Duplicate values, missing targets, skewed trees, and cycles where relevant.
- Whether input collections remain unchanged when that is the contract.
- Agreement with a simpler trusted implementation on small examples.

For example, after defining `factorial` from Section 2:

```python
import math


for n in range(15):
    assert factorial(n) == math.factorial(n)

try:
    factorial(-1)
except ValueError:
    pass
else:
    raise AssertionError("negative input should be rejected")
```

Tracebacks with repeated calls help locate a nonterminating branch. A debugger can inspect separate frames. A `RecursionError` can also occur in a correctly terminating algorithm with excessive depth; check the input size before assuming the base case is wrong.

## 16. Practical Uses and Choosing an Approach

Recursion appears in:

- **Data engineering:** Traversing nested JSON, configuration objects, and hierarchical metadata.
- **Compilers and interpreters:** Parsing nested expressions and walking abstract syntax trees.
- **Algorithms:** Tree traversal, divide and conquer, DFS, and backtracking.
- **Machine learning:** Expressing recursive splits in decision-tree construction and traversing trained trees.
- **File systems:** Exploring directory trees, with explicit policies for symbolic links, errors, and depth.

| Situation | Usually a good starting point |
|---|---|
| Naturally nested data with bounded depth | Recursion |
| A simple counter or sequence scan | Loop or built-in function |
| Repeated identical subproblems | Memoization or bottom-up dynamic programming |
| Very deep or externally supplied nesting | Explicit stack and depth/resource bounds |
| Need shortest paths in an unweighted graph | Breadth-first search with a queue |
| Numerical work on large arrays | Suitable array operations rather than Python element-by-element recursion |

Recursion does not automatically make an algorithm faster. Choose it when the structure clarifies the solution and the expected depth is manageable. Standard helpers such as `math.factorial`, `math.gcd`, `sum`, and `sorted` are preferable to educational reimplementations for ordinary use.

## 17. Practice Problems

| Level | Problem | Hint or expected property |
|---|---|---|
| Beginner | Sum integers from `1` to `n` | Base case `n == 0`; compare with `n * (n + 1) // 2` |
| Beginner | Sum digits of a nonnegative integer | Use `% 10` and `// 10`; decide what zero returns |
| Beginner | Count occurrences in a list | Pass an index instead of slicing |
| Intermediate | Compute binary-tree height | Define empty-tree height consistently, e.g. zero |
| Intermediate | Generate permutations of distinct items | Choose an unused item, recurse, undo |
| Intermediate | Solve Tower of Hanoi | Move `n - 1`, move largest, move `n - 1`; `2^n - 1` moves |
| Intermediate | Generate balanced parentheses | Track open/closed counts and prune invalid prefixes |
| Advanced | Detect a directed graph cycle | Track both visited and currently active vertices |
| Advanced | Solve N-Queens | Track occupied columns and diagonals |
| Advanced | Implement an iterative postorder traversal | Represent pending work explicitly |

For every solution, state its input contract, base case, progress measure, time complexity, stack depth, and output size. Include at least one boundary example.

## 18. Interview Questions and Quick Reference

**What is the difference between recursion and iteration?**

Recursion repeats work through function calls; iteration repeats work through loops. Recursive execution stores pending calls in frames. An iterative translation may need an explicit stack to preserve equivalent state.

**Can recursion have multiple base cases?**

Yes. Fibonacci has base cases for zero and one. Search algorithms may stop either when they find a target or when no candidates remain.

**Does every recursive algorithm take exponential time?**

No. Cost depends on the number and size of subproblems and the work in each call. Binary search is logarithmic, recursive sum is linear, and naive Fibonacci is exponential.

**Why is recursive Fibonacci slow?**

Its call tree repeatedly solves the same subproblems. Memoization computes each distinct state once, while iteration can additionally avoid the recursive stack.

**Is backtracking the same as recursion?**

No. Backtracking is a strategy for exploring and undoing choices. Recursion is one way to implement that strategy.

**Is recursion always more memory-intensive than iteration?**

A simple loop can avoid a growing call stack. But an iterative tree or graph algorithm may still need substantial explicit state. Compare the complete algorithms, including output and temporary storage.

**What makes a recursive solution correct?**

Correct base cases, a correct reduction and combination step, and a termination argument over the allowed input domain.

### Revision checklist

- Define what one call means and returns.
- Handle the simplest valid cases first.
- Make every recursive branch progress toward termination.
- Return recursive results when the caller needs them.
- Distinguish total calls from maximum active depth.
- Account for slicing, copying, cached values, and output size.
- Use cycle protection for cyclic data.
- Restore mutable state during backtracking.
- Memoize overlapping subproblems when the state is cacheable.
- Prefer iteration or an explicit stack when depth may be large.

> **Remember:** Solve a simpler instance under the same contract, then use that result to finish the current instance.
