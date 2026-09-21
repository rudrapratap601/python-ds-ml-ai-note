# File Handling in Python

> **Purpose:** Read, write, organize, and serialize data reliably, from small text files to streamed datasets. Examples use Python 3.11+ and the standard library unless stated otherwise.

## Contents

- [Paths and file objects](#paths-and-file-objects)
- [Modes and resource management](#modes-and-resource-management)
- [Text and binary data](#text-and-binary-data)
- [Reading and writing patterns](#reading-and-writing-patterns)
- [Structured formats](#structured-formats)
- [Directories and file operations](#directories-and-file-operations)
- [Safer replacement and durability](#safer-replacement-and-durability)
- [Large files and performance](#large-files-and-performance)
- [Errors and testing](#errors-and-testing)
- [Revision and practice](#revision-and-practice)

## Paths and file objects

A **path** identifies a location. A **file object** is an open stream with a mode, position, and lifetime. The path can exist without an open stream; opening a missing path may either fail or create a file, depending on the mode.

Use `pathlib.Path` to combine paths without manually inserting platform-specific separators:

```python
from pathlib import Path

path = Path("data") / "raw" / "sales.csv"
assert path.name == "sales.csv"
assert path.stem == "sales"
assert path.suffix == ".csv"
assert path.parent == Path("data") / "raw"
```

Relative paths resolve from the **current working directory**, which need not be the script's directory. `Path.cwd()` reports it. In a normal script, `Path(__file__).resolve().parent` locates the script directory; notebooks do not generally define `__file__`.

`Path.home()` finds the user's home directory. `expanduser()` expands `~`. `resolve()` produces an absolute resolved path, but does not magically make a file exist or guarantee that an operation is permitted.

On Windows, use `Path(r"C:\data\report.csv")` when writing a literal backslash path. A raw string cannot end with a single backslash.

## Modes and resource management

| Mode | Existing file | Missing file | Intended operation |
|---|---|---|---|
| `r` | Read | Error | Read text |
| `w` | Truncate immediately | Create | Replace content |
| `a` | Append writes | Create | Add content at the end |
| `x` | Error | Create | Create without overwriting |
| `r+` | Read/write without initial truncation | Error | Update an existing file |
| `w+` | Truncate; allow reading/writing | Create | Replace and then read/write |
| `a+` | Read; writes append | Create | Read and append |
| `rb`, `wb` | Binary equivalents | Depends on base mode | Work with bytes |

The `+` means updating, not appending. The `b` means bytes; `t` means text and is the default. Opening in `w` can erase existing content before the first call to `write()`.

Use a context manager to close a stream even when the body raises an exception. The official [file I/O tutorial](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files) describes modes and stream operations.

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    path = Path(directory) / "notes.txt"
    with path.open("w", encoding="utf-8") as stream:
        stream.write("Python\n")
    assert stream.closed
    with path.open("r", encoding="utf-8") as stream:
        assert stream.read() == "Python\n"
```

`with` guarantees the context manager's cleanup protocol, not successful completion of the operation. Closing or flushing a file can itself raise an error.

## Text and binary data

- Text streams accept and return `str`, decoding and encoding bytes using a codec.
- Binary streams accept and return bytes-like values, such as `bytes`.
- Specify `encoding="utf-8"` for portable text files when UTF-8 is your format contract.
- An encoding error and a missing-file error describe different failures; handle them separately.
- `errors="replace"` substitutes unrecognized text. It can be useful for diagnosis but changes the data. `errors="ignore"` silently drops information.
- `utf-8-sig` can consume a UTF-8 byte-order mark when reading files produced by some spreadsheet tools.

```python
message = "café"
payload = message.encode("utf-8")
assert isinstance(payload, bytes)
assert payload.decode("utf-8") == message
```

Text mode may translate newline sequences. Open CSV text files with `newline=""` so the CSV module manages record newlines itself. Binary mode preserves bytes exactly.

## Reading and writing patterns

| Operation | Meaning | Memory concern |
|---|---|---|
| `read()` | Read remaining content | Loads everything remaining |
| `read(size)` | Read up to a size | Text size counts characters; binary size counts bytes |
| `readline()` | Read one line | A single line can still be huge |
| `readlines()` | List of remaining lines | Loads all remaining lines |
| `for line in stream` | Iterate lines | Usually a good streaming default |
| `write(text)` | Write content; return character count in text mode | Does not add a newline |
| `writelines(lines)` | Write an iterable of strings | Does not add separators |

```python
from io import StringIO, BytesIO

stream = StringIO("alpha\nbeta\n")
assert next(stream) == "alpha\n"
assert stream.read() == "beta\n"
assert stream.read() == ""  # Text EOF.
stream.seek(0)
assert len(list(stream)) == 2

binary = BytesIO(b"abcdef")
assert binary.read(2) == b"ab"
assert binary.tell() == 2
binary.seek(0)
assert binary.read() == b"abcdef"
```

At binary EOF, `read()` returns `b""`. A blank text line is `"\n"`, not EOF. `strip()` removes more than a newline; use `rstrip("\r\n")` when preserving spaces matters.

`tell()` reports a stream position. For text files, it can be an opaque position cookie; do not treat it as a simple character index. Arbitrary byte seeking belongs in binary mode. After reading to EOF, call `seek(0)` to read again.

`Path.read_text()`, `write_text()`, `read_bytes()`, and `write_bytes()` are convenient for small whole-file operations. A write is replacement, not append, and parent directories must already exist.

## Structured formats

### CSV: rows and columns

CSV is a text exchange format, not a schema. Values initially arrive as strings. Delimiters, quoting, encodings, null markers, and type conversion need an explicit policy. Never parse arbitrary CSV by splitting every line on commas.

```python
import csv
from io import StringIO

buffer = StringIO(newline="")
writer = csv.DictWriter(buffer, fieldnames=["name", "score"])
writer.writeheader()
writer.writerows([{"name": "Asha, R.", "score": 9}, {"name": "Dev", "score": 8}])
buffer.seek(0)
rows = list(csv.DictReader(buffer))
assert rows[0] == {"name": "Asha, R.", "score": "9"}
```

For real files, use `open(..., encoding="utf-8", newline="")`. If exporting untrusted text for spreadsheet software, decide how to handle values interpreted as formulas.

### JSON: nested interoperable data

```python
import json

record = {"name": "Asha", "scores": [8, 9], "active": True, "manager": None}
encoded = json.dumps(record, ensure_ascii=False, indent=2, allow_nan=False)
assert json.loads(encoded) == record
```

`dump`/`load` use file objects; `dumps`/`loads` use strings or supported in-memory inputs. JSON object keys are strings. Tuples become arrays; sets, datetimes, and arbitrary classes need explicit conversion. Standard JSON has no NaN or infinity, so `allow_nan=False` rejects those values when writing.

For **JSON Lines**, serialize one JSON value per line. It supports incremental processing without reading one huge array. A normal pretty-printed JSON document is not JSON Lines.

```python
lines = [json.dumps({"id": value}) + "\n" for value in range(3)]
decoded = [json.loads(line) for line in lines if line.strip()]
assert decoded == [{"id": 0}, {"id": 1}, {"id": 2}]
```

### Choosing a format

| Format | Suitable for | Important limitation |
|---|---|---|
| Text | Notes, logs, simple protocols | You define the structure |
| CSV | Flat exchange tables | Weak type and nested-data support |
| JSON / JSON Lines | APIs and nested records | Verbose; manual schema validation |
| TOML | Human-edited configuration | `tomllib` reads TOML in Python 3.11+; writing needs another library |
| Pickle | Trusted Python-specific objects | Loading untrusted pickle can execute code |
| Parquet | Typed analytical tables | Requires an engine such as PyArrow or Polars |
| SQLite | Structured local queries and transactions | Database schema and transaction design required |

Pickle is not a safe interchange format for arbitrary uploads, and long-term portability depends on code and library versions. Compression and serialization are separate layers: a JSON Lines file can also be gzip-compressed.

## Directories and file operations

```python
with TemporaryDirectory() as directory:
    root = Path(directory)
    reports = root / "reports"
    reports.mkdir(parents=True, exist_ok=True)
    (reports / "one.txt").write_text("one", encoding="utf-8")
    (reports / "two.txt").write_text("two", encoding="utf-8")
    assert sorted(p.name for p in reports.glob("*.txt")) == ["one.txt", "two.txt"]
```

`iterdir()` lists direct children; `glob()` matches patterns; `rglob()` searches recursively. Filesystem enumeration order is not a sorting guarantee.

| Task | Tool | Check before using |
|---|---|---|
| Copy bytes | `shutil.copyfile` | Destination overwrite policy |
| Copy with some metadata | `shutil.copy2` | Metadata preservation varies by platform |
| Move | `shutil.move` | Cross-filesystem moves may copy then delete |
| Replace destination | `Path.replace` / `os.replace` | Overwrite is intentional |
| Remove a file | `Path.unlink` | Exact target is correct |
| Remove an empty directory | `Path.rmdir` | Directory really is empty |
| Remove a tree | `shutil.rmtree` | Resolve and verify the entire target first |

Do not assume `exists()` followed by `open()` is race-free: another process can change the path between operations. Prefer performing the intended operation and catching the relevant error. Use `x` mode for exclusive creation rather than checking first.

## Safer replacement and durability

Writing directly over an important file can leave it incomplete if the process fails. A common pattern writes a temporary file in the **same directory**, closes it, and replaces the destination.

```python
import os
import tempfile


def replace_text(destination, text):
    destination = Path(destination)
    temporary = None
    try:
        with tempfile.NamedTemporaryFile(
            mode="w", encoding="utf-8", dir=destination.parent,
            prefix=".pending-", delete=False
        ) as stream:
            temporary = Path(stream.name)
            stream.write(text)
            stream.flush()
            os.fsync(stream.fileno())
        os.replace(temporary, destination)
    finally:
        if temporary is not None:
            temporary.unlink(missing_ok=True)


with TemporaryDirectory() as directory:
    target = Path(directory) / "settings.txt"
    replace_text(target, "mode=study\n")
    assert target.read_text(encoding="utf-8") == "mode=study\n"
```

This assumes the destination parent exists. Same-filesystem replacement can provide atomic visibility where supported; it is not a complete database transaction, concurrent-writer lock, metadata-preserving copy, or universal crash-durability guarantee. Durability can require filesystem-specific handling, including directory synchronization. `flush()` alone moves Python buffers onward; it does not guarantee physical persistence.

## Large files and performance

- Stream records instead of using `read()` when the file may exceed memory.
- Read binary data in chunks, for example `iter(lambda: stream.read(1024 * 1024), b"")`.
- Avoid accumulating streamed records in a list unless needed downstream.
- Use `gzip.open(path, "rt", encoding="utf-8")` for compressed text.
- Consider `mmap` for suitable random-access binary workloads; it does not remove file lifetime and platform constraints.
- Use column selection and row filtering with Parquet readers for analytical data.
- Buffering reduces system-call overhead. Disabling buffering is generally for specialized binary I/O, not routine text processing.

## Errors and testing

Common failures include `FileNotFoundError`, `PermissionError`, `IsADirectoryError`, `UnicodeDecodeError`, `json.JSONDecodeError`, and `csv.Error`. Some I/O failures occur during iteration or close, not just when opening.

Test empty files, missing files, non-ASCII text, malformed records, missing parent directories, repeated writes, and interruptions where reliability matters. `TemporaryDirectory`, `StringIO`, and `BytesIO` allow isolated examples without touching personal data.

Keep path validation, format parsing, and business validation separate so errors explain what actually went wrong. See [exception handling](exception-handling.md).

## Revision and practice

1. Count lines without reading a whole file into memory.
2. Convert a CSV with explicit numeric types to JSON Lines.
3. Merge text files in a sorted order while preserving line boundaries.
4. Write a configuration replacement function and test failed serialization before replacement.
5. Explain why `w`, `a`, and `x` are not interchangeable.

**Remember:** Choose the path, mode, encoding, format, resource lifetime, and overwrite policy deliberately.

Further references: [pathlib](https://docs.python.org/3/library/pathlib.html), [csv](https://docs.python.org/3/library/csv.html), [json](https://docs.python.org/3/library/json.html), [tempfile](https://docs.python.org/3/library/tempfile.html).
