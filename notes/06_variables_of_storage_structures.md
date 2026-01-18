# Storage Structures: Buffering, Immutability, and Ordering

A **storage engine** is built on top of one or more data structures, but these
structures alone do not define important system properties such as caching,
recovery, concurrency, and transactionality. Many differences between storage
engines arise not from the data structure itself, but from how it is *used*.

Most storage engines can be analyzed through **three fundamental variables**:

1. **Buffering**
2. **Mutability vs. Immutability**
3. **Ordering**

Nearly all optimizations and design trade-offs in modern storage engines relate
to one or more of these dimensions.

---

## 1. Buffering

### Definition

**Buffering** describes whether a storage structure deliberately accumulates
data in memory before writing it to disk.

This is distinct from unavoidable buffering caused by disk block sizes. Here,
buffering refers to **intentional, design-level buffering** chosen by the
storage engine.

---

### Motivation

* Disk I/O is expensive
* Writing small updates individually causes high I/O overhead
* Buffering amortizes I/O costs by writing data in larger batches

---

### Examples

* **B-Trees**

  * Traditional B-Trees update pages in place
  * Optimized variants (e.g., *Lazy B-Trees*) add in-memory buffers to nodes
    and flush updates later

* **LSM(Log Structured-Merge)-Trees**

  * Writes go first to an in-memory *memtable*
  * Data is flushed to disk in large, sequential writes
  * Buffering is a **core design principle**, not an optimization

---

### Key Insight

> Buffering trades memory usage and write latency for higher throughput and
> lower write amplification.

---

## 2. Mutability vs. Immutability

### Definition

This dimension defines whether data stored on disk is **modified in place** or
**never changed after being written**.

---

### Mutable (In-Place Update) Storage

#### Characteristics

* Pages are read from disk, modified, and written back to the same location
* Requires careful synchronization and recovery mechanisms

#### Examples

* Traditional B-Trees
* Heap-organized tables -HOT with in-place updates

#### Trade-offs

* Simple read paths
* Random writes and complex concurrency control

---

### Immutable (Append-Only) Storage

#### Characteristics

* Once written, data is never modified
* Updates are appended as new versions
* Old versions become obsolete and is clean by the garbage collector - GC

#### Implementation Techniques

* Append-only logs
* Copy-on-write (COW), where updated pages are written elsewhere

#### Examples

* LSM-Trees
* Bitcask
* Bw-Trees (B-Tree–inspired but immutable)

---

### Key Insight

> Immutability simplifies recovery and concurrency but requires background
> processes such as compaction or garbage collection.

---

## 3. Ordering

### Definition

**Ordering** describes whether data records are stored on disk in **key order**.

If ordered, records with nearby keys are placed in nearby disk locations.

---

### Ordered Storage

#### Characteristics

* Data is stored in sorted key order
* Enables efficient range scans

#### Examples

* B-Trees
* Index-organized tables
* Sorted SSTables in LSM-Trees

#### Trade-offs

* Efficient scans and lookups
* Higher write cost due to reorganization (splits, merges)

---

### Unordered (Out-of-Order) Storage

#### Characteristics

* Records are written in insertion order
* No on-disk key ordering

#### Examples

* Bitcask
* WiscKey (values unordered, keys indexed separately)

#### Trade-offs

* Extremely fast writes
* Inefficient range queries without indexes

---

### Key Insight

> Ordering favors read and scan efficiency, while lack of ordering favors write
> throughput.

---

## Combined View

Most storage engines are **combinations** of these three variables rather than
pure extremes.

| Storage Engine | Buffering | Mutability | Ordering           |
| -------------- | --------- | ---------- | ------------------ |
| B-Tree         | Limited   | Mutable    | Ordered            |
| Lazy B-Tree    | Yes       | Mutable    | Ordered            |
| LSM-Tree       | Heavy     | Immutable  | Ordered (on flush) |
| Bitcask        | Minimal   | Immutable  | Unordered          |
| Bw-Tree        | Yes       | Immutable  | Ordered            |

---

## Takeaway

> New storage engines do not emerge randomly. They explore different points in
> the design space defined by buffering, mutability, and ordering.

Understanding these three variables makes it easier to reason about:

* Write amplification
* Read and write paths
* Compaction and garbage collection
* Concurrency and recovery trade-offs


