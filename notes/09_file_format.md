# Chapter 3 — File Formats

## Goal of the Chapter

After covering **B-Tree semantics**, this chapter explains **how B-Trees and other data structures are implemented on disk**.

Key idea:

> **On-disk data structures are fundamentally different from in-memory ones**
> → they require explicit layout, pointer management, and careful handling of mutability.

This chapter introduces **general principles for designing binary file formats**, not limited to B-Trees.

---

## Memory vs Disk Access

### Memory

* Virtual memory hides physical layout
* Offsets are implicit
* Allocation and deallocation are abstracted

### Disk

* Accessed via system calls
* Explicit file offsets required
* Raw bytes must be interpreted manually
* Fragmentation and garbage collection are **manual responsibilities**

➡️ Efficient on-disk structures **must be designed explicitly**.

---

## B-Trees as Page Management Systems

* On-disk B-Trees should be seen as **page composition and navigation mechanisms**
* Algorithms operate on **pages**, not pointers
* Page identifiers replace in-memory pointers
* Most B-Tree complexity comes from **mutability**:

  * splits
  * merges
  * relocations
  * space reclamation

This contrasts with **LSM Trees**, where complexity mainly comes from sorting and maintenance.

---

## Motivation: Designing File Formats

Creating a file format resembles programming in:

* C
* unmanaged memory environments

But with additional constraints:

* No `malloc` / `free`
* Only `read` and `write`
* Fixed-size blocks
* Persistent fragmentation

Key challenges:

* Efficient layout for disk access
* Compact binary representation
* Fast serialization / deserialization
* Manual space management

---

## Binary Encoding Fundamentals

### Primitive Types

* Fixed-size numeric values:

  * `byte` (8 bits)
  * `short` (16 bits)
  * `int` (32 bits)
  * `long` (64 bits)

### Endianness

Defines byte order for multibyte values:

* **Big-endian**: most-significant byte first
* **Little-endian**: least-significant byte first

Encoding and decoding **must agree on endianness**
(e.g., RocksDB performs byte reversal if needed).

---

## Floating-Point Numbers

* Represented using **IEEE 754**
* Stored as:

  * sign
  * exponent
  * fraction
* Floating-point values are approximations
* Encoding/decoding usually handled by standard libraries

---

## Strings and Variable-Size Data

### Pascal (Length-Prefixed) Strings

```
String {
  uint16 size
  byte[size] data
}
```

Advantages over null-terminated strings:

* Constant-time length access
* No scanning required
* Easy slicing during deserialization

Used widely in binary formats.

---

## Bit-Packed Data

### Booleans

* Stored as bits instead of bytes
* 8 booleans per byte

### Enums

* Represented as integers
* Used for low-cardinality repeated values

Example:

```c
enum NodeType {
  ROOT,      // 0x00
  INTERNAL,  // 0x01
  LEAF       // 0x02
};
```

### Flags

* Multiple boolean values packed into one integer
* Each flag corresponds to a bit

Example:

```c
IS_LEAF_MASK          = 0x01;
VARIABLE_SIZE_VALUES = 0x02;
HAS_OVERFLOW_PAGES   = 0x04;
```

Manipulated using bitwise operations (`|`, `&`, `~`, `<<`).

---

## General File Format Principles

### Page-Based Organization

* Files split into **fixed-size pages** (4–16 KB)
* Simplifies reads, writes, and caching
* Used by most in-place update systems

### Typical File Layout

```
[ Header ][ Pages... ][ Optional Trailer ]
```

* Header / trailer contain metadata
* Pages contain actual data

---

## Fixed Schema Optimization

* Fixed schema avoids storing field names repeatedly
* Fields identified by position
* Reduces disk footprint

### Fixed + Variable Field Layout Example

* Fixed-size fields first
* Variable-size fields follow
* Lengths or offsets stored in fixed region

This allows efficient access without scanning.

---

## Page Structure in Databases

* Database files consist of **pages**
* Pages store data or index entries
* In B-Trees:

  * node ≈ page ≈ block
* Leaf pages store key–value pairs
* Internal pages store keys and child pointers

### Simple B-Tree Page Layout (Original Paper)

* Sequential triplets: `(key, value, pointer)`
* Works only for fixed-size records
* Requires relocation on inserts

---

## Slotted Pages (Slot Directory)

### Problem

Variable-size records cause:

* Fragmentation
* Space waste
* Offset instability

### Solution: Slotted Page

A page is split into:

* **Header**
* **Slot array (cell offsets)**
* **Cells (records)**

Characteristics:

* Cells grow from the end of the page
* Slots grow from the beginning
* Slots store offsets to cells

Used by systems like **PostgreSQL**.

### Benefits

* Minimal overhead
* Efficient space reclamation
* Stable external references via slot IDs
* Logical order independent of physical order

---

## Cell Layout

### Cell Types

* **Key cells** (internal nodes)
* **Key-value cells** (leaf nodes)

### Variable-Size Key Cell

```
[int] key_size
[int] page_id
[bytes] key
```

### Variable-Size Key-Value Cell

```
[byte] flags
[int] key_size
[int] value_size
[bytes] key
[bytes] value
```

Design principles:

* Fixed-size header first
* Variable-size fields follow
* Page ID used instead of raw offsets
* Cell offsets are page-local

---

## Combining Cells into Slotted Pages

* Cells appended sequentially
* Slot array maintains **sorted key order**
* Physical layout ≠ logical order

This allows:

* Fast inserts (no cell relocation)
* Binary search via sorted slots

---

## Managing Variable-Size Data

### Deletion

* Cells marked as deleted
* Space tracked in an **availability list**

### Allocation Strategies

* **First fit**
* **Best fit**

### Defragmentation

* Triggered when space is fragmented
* Live cells rewritten
* Space reclaimed

If still insufficient → **overflow pages**

---

## Versioning

Binary formats evolve → multiple versions must be supported.

Approaches:

* Versioned filenames (e.g., Cassandra)
* Separate version file (e.g., PostgreSQL)
* Version stored in file header
* Magic numbers

Reader selects version-specific decoding logic.

---

## Checksumming

Purpose:

* Detect accidental corruption
* Avoid propagating bad data

Types:

* **Checksums** (weak)
* **CRCs** (detect burst errors)
* **Cryptographic hashes** (tamper resistance)

Databases typically:

* Compute checksums **per page**
* Store checksum in page header

This localizes corruption and avoids discarding entire files.

---

## Chapter Summary

This chapter introduced:

* Binary encoding of primitive and composite data
* Handling of variable-size values
* Page-based storage organization
* Slotted page design
* Cell composition and navigation
* Versioning strategies
* Page-level checksumming

> These principles form the **physical foundation** of on-disk data structures, B-Trees, and storage engines in general.

---

