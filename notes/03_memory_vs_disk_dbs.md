# Memory-Based vs Disk-Based DBMS

## 1. Fundamental Storage Model

Database management systems store data using **both memory and disk**, but differ in **which medium is primary**.

### Disk-Based DBMS

* Store **most data on disk** (HDD or SSD)
* Use memory mainly for:

  * Caching frequently accessed pages
  * Temporary data (sorting, joins, intermediate results)
* Disk is the **source of truth**

### Memory-Based (In-Memory) DBMS

* Store **most or all data in RAM**
* Use disk primarily for:

  * Logging
  * Backups
  * Recovery
* Memory is the **source of truth**

> Both systems use disk and memory — the distinction is **which one dominates**.

<img src="/assets/memory_vs_disk_dbs.png" >

---

## 2. Performance Motivation

### Memory Access vs Disk Access

* Memory access is **several orders of magnitude faster** than disk access
* Random access in memory is cheap
* Disk access (especially random) is expensive

### Why Use Memory as Primary Storage?

* Significantly lower access latency
* Finer access granularity (byte-addressable)
* Better throughput for read-heavy and latency-sensitive workloads

### Economic Trade-Off

* RAM prices are decreasing
* But RAM is still **much more expensive** than persistent storage (SSD/HDD)
* Cost remains a limiting factor for large datasets

---

## 3. Architectural Implications

Using memory as the primary storage medium **changes system design deeply**.

### Data Structures & Layout

* **In-memory DBMS**:

  * Can use pointer-rich data structures
  * Can optimize for CPU caches and memory locality
  * No need for serialization between memory and disk formats

* **Disk-based DBMS**:

  * Use storage structures optimized for disk I/O
  * Often rely on **wide, shallow trees** (e.g., B+ trees)
  * Must minimize disk seeks and page reads

> Disk-oriented structures are shaped by I/O constraints, not algorithmic elegance.

---

## 4. Programming Model Differences

### Memory Programming

* OS abstracts memory management
* Allocate/free arbitrary-sized chunks
* Pointers can be followed directly
* Fragmentation and layout are largely hidden

### Disk Programming

* Requires explicit management of:

  * Serialization formats
  * On-disk references
  * Free space
  * Fragmentation
* Random access is costly and must be minimized

This makes **in-memory systems simpler to program**, but harder to make durable.

---

## 5. Durability Challenges in In-Memory DBMS

### Volatility of RAM

RAM is **not persistent**:

* Software crashes
* Hardware failures
* Power outages
  → can result in **total data loss**

### Mitigation Techniques

* Write-ahead logging (WAL)
* Disk-based backups
* Battery-backed RAM
* Uninterruptible Power Supplies (UPS)

However:

* These add **hardware cost**
* Require **operational expertise**
* Disks remain cheaper and easier to manage

---

## 6. Durability Mechanisms

### Write-Ahead Logging (WAL)

* Every operation is written to a **sequential log** on disk
* Operation is considered complete only after log write

### Backup & Recovery

* A disk-based backup copy is maintained
* Backup is:

  * Sorted
  * Updated **asynchronously**
  * Written in batches to reduce I/O

### Checkpointing

* Log records are periodically applied to the backup
* After a checkpoint:

  * Backup represents a consistent snapshot
  * Older log entries can be discarded
* Reduces recovery time
* Does **not block clients**

---

## 7. In-Memory ≠ Disk DBMS with a Huge Cache

> An in-memory DBMS is **not** just an on-disk DBMS with a massive buffer pool.

Reasons:

* Disk-based pages still use serialized formats
* Page layout incurs overhead
* Cannot exploit the same low-level optimizations
* In-memory systems can tailor layout, indexing, and access patterns fully for RAM

---

## 8. Data Structure Choices

### Disk-Based Systems

* Optimized for minimizing I/O
* Prefer:

  * Wide and shallow trees
  * Page-oriented layouts
* Variable-size data is complex to manage

### In-Memory Systems

* Random access is cheap
* Pointers are efficient
* Larger choice of data structures
* Variable-size data handled via references

---

## 9. Practical Use Cases for In-Memory DBMS

In-memory databases are well-suited when:

* Dataset size is **bounded**
* Records are small (few KB)
* Total data fits comfortably in RAM

Examples:

* Student records
* Customer profiles
* Inventory systems

---

## Mental Model Summary

| Aspect           | Disk-Based DBMS | In-Memory DBMS              |
| ---------------- | --------------- | --------------------------- |
| Primary storage  | Disk            | RAM                         |
| Performance      | I/O-bound       | CPU & memory-bound          |
| Durability       | Native          | Emulated via logs & backups |
| Data structures  | I/O-optimized   | Pointer & cache optimized   |
| Cost scalability | High            | Limited by RAM              |


