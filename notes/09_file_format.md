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

Great question — this is a **core idea in database storage**, and it’s normal that it feels abstract at first. Let’s make it concrete with a **step-by-step example**, and tie every step back to the problems described in the text.

---

## 1️⃣ The core problem (without slotted pages)

Assume:

* Page size = **256 bytes**
* Records are **variable-size** (strings, blobs, etc.)

### Insert some records

| Record | Size     |
| ------ | -------- |
| R1     | 40 bytes |
| R2     | 70 bytes |
| R3     | 30 bytes |

They are stored sequentially in the page:

```
| R1 (40) | R2 (70) | R3 (30) | free space |
```

Used space = 140 bytes
Free space = 116 bytes

---

### Delete R2 (70 bytes)

Now the page looks like this:

```
| R1 (40) | [FREE 70] | R3 (30) | free space |
```

### Try to insert R4 (50 bytes)

* The free hole is **70 bytes**
* R4 needs **50 bytes**

**Problem**:

* You *could* put R4 in that space
* But then you’d have **20 bytes of unused space**
* After many inserts/deletes, the page becomes **fragmented**

This is **external fragmentation**.

---

## 2️⃣ Why fixed-size segments don’t fully solve it

Suppose instead:

* You split the page into **64-byte segments**

### Insert R1 (40 bytes)

* Uses 1 segment
* Wasted space = 24 bytes

### Insert R2 (70 bytes)

* Needs 2 segments = 128 bytes
* Wasted space = 58 bytes

➡️ You avoid fragmentation, but now you have **internal fragmentation** (wasted space inside blocks).

So we want:

* Variable-size records
* Minimal waste
* Easy deletion
* Stable references to records

This is where **slotted pages** come in.

---

## 3️⃣ Slotted page idea (big picture)

A **slotted page splits the page into two logical regions**:

```
| Page Header | Slot Directory → ← Free Space → ← Records |
```

* **Slot directory** grows from the **front**
* **Records (cells)** grow from the **back**
* Free space stays in the middle

Crucially:

* Slots contain **pointers (offsets)** to records
* External code refers to records by **slot ID**, not physical offset

---

## 4️⃣ Concrete slotted page example

### Page size = 256 bytes

### Step 1: Insert R1 (40 bytes)

* Allocate a **slot entry**:

  ```
  Slot 0 → offset 216, length 40
  ```
* Write R1 at the end of the page

```
| Header | Slot[0] | free space | R1 |
```

---

### Step 2: Insert R2 (70 bytes)

```
Slot 1 → offset 146, length 70
```

```
| Header | Slot[0] | Slot[1] | free | R2 | R1 |
```

---

### Step 3: Insert R3 (30 bytes)

```
Slot 2 → offset 116, length 30
```

```
| Header | Slot[0] | Slot[1] | Slot[2] | free | R3 | R2 | R1 |
```

Notice:

* Records are packed from the **end**
* Slots grow from the **beginning**

---

## 5️⃣ Deleting a record (this is the key part)

### Delete R2

We do **NOT** immediately move bytes.

Instead:

* Either:

  * Mark `Slot 1` as **invalid**, or
  * Remove it and compact the slot directory

```
Slot 1 → null
```

Page now looks like:

```
| Header | Slot[0] | Slot[1=null] | Slot[2] | free | R3 | [R2 garbage] | R1 |
```

R2’s bytes are now **garbage**, but:

* No one references them anymore
* No external pointer breaks

---

## 6️⃣ Space reclamation (defragmentation)

When the page needs space:

* The database **rewrites the page**
* Live records are packed tightly again
* Slot offsets are updated

After compaction:

```
| Header | Slot[0] | Slot[2] | free | R3 | R1 |
```

Slot directory updated:

```
Slot 0 → new offset of R1
Slot 2 → new offset of R3
```

🚨 External references remain valid because they use **slot IDs**, not offsets.

---

## 7️⃣ Why this solves the original problems

Let’s map directly to the text.

---

### ✅ Minimal overhead

Only extra cost:

* One slot entry per record (offset + length)

No padding, no fixed blocks.

---

### ✅ Space reclamation

* Deleted records leave garbage
* Garbage is removed during page rewrite
* No fragmentation leaks outside the page

---

### ✅ Reference records without knowing location

External references look like:

```
(page_id, slot_id)
```

Not:

```
(page_id, byte_offset)
```

So records can move freely **inside the page**.

---

## 8️⃣ Mental model to remember

Think of a slotted page like:

* **Array of pointers** (slot directory)
* **Heap of variable-sized objects** (records)
* Garbage collection happens **per page**

Very similar to:

* Heap + object references in a VM
* Indirection to enable movement

---

## 9️⃣ Why this matters for you (database internals perspective)

This design:

* Enables efficient B-Trees with variable-size keys
* Makes DELETE cheap
* Enables UPDATE that changes record size
* Is foundational to PostgreSQL, SQLite, many others

You’re now at the point where:

> *Pages are no longer just byte arrays — they’re miniature memory managers.*


