# 6. Descriptors and Class Customization

> **Learning goal:** Understand Python's attribute machinery and choose the smallest extension mechanism that solves the problem.

## Contents

- [Attribute lookup](#attribute-lookup)
- [Reusable descriptors](#reusable-descriptors)
- [How methods bind](#how-methods-bind)
- [Subclass hooks and class decorators](#subclass-hooks-and-class-decorators)
- [Metaclasses](#metaclasses)
- [Slots, weak references, and lifecycle](#slots-weak-references-and-lifecycle)
- [Choosing an extension mechanism](#choosing-an-extension-mechanism)

## Attribute lookup

A descriptor is an object stored on a class that defines `__get__`, `__set__`, or `__delete__`. Properties and function method binding use descriptor behavior.

For ordinary instance lookup, a simplified precedence is:

1. Data descriptor found through the class MRO.
2. Instance dictionary, when present.
3. Non-data descriptor or ordinary class attribute found through the MRO.
4. `__getattr__` fallback when normal lookup fails.

A descriptor with `__set__` or `__delete__` is a **data descriptor**. One with only `__get__` is a **non-data descriptor**. Custom `__getattribute__` can alter normal behavior, so use it carefully. See the official [descriptor guide](https://docs.python.org/3/howto/descriptor.html).

## Reusable descriptors

```python
class NonNegativeInteger:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.storage_name = f"_{name}"

    def __get__(self, instance, owner=None):
        if instance is None:
            return self
        return getattr(instance, self.storage_name)

    def __set__(self, instance, value):
        if isinstance(value, bool) or not isinstance(value, int):
            raise TypeError(f"{self.public_name} must be an integer")
        if value < 0:
            raise ValueError(f"{self.public_name} must be nonnegative")
        setattr(instance, self.storage_name, value)


class Batch:
    size = NonNegativeInteger()
    retries = NonNegativeInteger()

    def __init__(self, size, retries=0):
        self.size = size
        self.retries = retries


first = Batch(32)
second = Batch(64, 2)
first.size = 16
assert (first.size, second.size) == (16, 64)
assert isinstance(Batch.size, NonNegativeInteger)
```

`__set_name__` receives the owning class and assigned name during class creation. The descriptor stores each instance's value on that instance. Storing one value directly on the descriptor would accidentally share it among all instances.

This implementation expects writable instance attributes and a unique descriptor instance per field. A slots-based class needs matching storage slots or another storage design. Attaching a descriptor after class creation does not automatically rerun its `__set_name__` hook.

Use a descriptor when the same attribute behavior is reused across multiple fields or classes. A property is usually simpler for one field.

## How methods bind

Ordinary functions on a class are non-data descriptors. Access through an instance produces a bound method that stores both the underlying function and the instance.

```python
class Greeter:
    def greet(self, name):
        return f"Hello, {name}"


greeter = Greeter()
bound = greeter.greet
assert bound.__self__ is greeter
assert bound.__func__ is Greeter.greet
assert bound("Asha") == "Hello, Asha"
```

`classmethod` binds the class instead; `staticmethod` returns the callable without automatic instance binding. Understanding this helps explain why some callable-object decorators fail when used on methods.

## Subclass hooks and class decorators

`__init_subclass__` lets a base class configure or validate future subclasses without writing a metaclass.

```python
class Plugin:
    registry = {}

    def __init_subclass__(cls, *, name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if name is not None:
            if name in Plugin.registry:
                raise ValueError(f"duplicate plugin: {name}")
            Plugin.registry[name] = cls


class CsvPlugin(Plugin, name="csv"):
    pass


assert Plugin.registry["csv"] is CsvPlugin
```

Forward unconsumed keyword arguments cooperatively to support other bases. Registration occurs when the class statement executes, usually at import time. Manage registry lifetime in tests and define duplicate behavior.

A class decorator is another way to register or transform a finished class. Use it when behavior should be opt-in per class rather than inherited from a common base.

## Metaclasses

Classes are themselves objects. A metaclass controls class creation; the usual metaclass is `type`. A class statement executes a body to build a namespace, then creates the class object using its metaclass.

Relevant hooks include `__prepare__` for the class namespace, metaclass `__new__` for construction, and metaclass `__init__` for initialization. Custom metaclass `__call__` can affect how instances are created.

```python
class RequireRun(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        if not callable(namespace.get("run")):
            raise TypeError(f"{name} must define a callable run method")
        return super().__new__(mcls, name, bases, namespace, **kwargs)


class Job(metaclass=RequireRun):
    def run(self):
        return "complete"


assert Job().run() == "complete"
```

This teaching metaclass deliberately requires a new callable `run` in every class body, including subclass bodies; it does not accept merely inheriting the method. A real interface contract is usually better expressed with an ABC or Protocol.

When forwarding a class namespace to `type.__new__`, preserve entries such as `__classcell__`; disrupting them can break zero-argument `super()`. Multiple bases with incompatible metaclasses can cause conflicts. These costs are reasons to prefer simpler hooks when sufficient.

## Slots, weak references, and lifecycle

`__slots__` declares supported instance storage names and can reduce memory for many small objects. It does not enforce deep immutability or access privacy. Include appropriate support when weak references or dynamic attributes are required, and consider inheritance carefully.

`weakref` permits references that do not keep an object alive by themselves. It is useful for caches and registries whose contents should disappear when no real owner remains. Not all objects support weak references.

Avoid relying on `__del__` for important cleanup. Finalization can occur at inconvenient times, and objects can participate in reference cycles. Explicit resource ownership and context managers are easier to reason about.

## Choosing an extension mechanism

| Requirement | First tool to consider |
|---|---|
| Validate one attribute | Property |
| Reuse managed-attribute behavior | Descriptor |
| Add behavior to a function | Function decorator |
| Transform/register selected classes | Class decorator |
| Enforce a rule on future subclasses | `__init_subclass__` |
| Declare required operations | ABC or Protocol |
| Customize class creation itself | Metaclass |

Practice: trace descriptor precedence when a same-named instance dictionary entry exists; write a bounded-number descriptor; implement a plugin registry using both a decorator and a subclass hook; explain why a metaclass is unnecessary for most business objects.

Next: [Design principles and patterns](patterns.md).
