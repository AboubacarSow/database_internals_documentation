## Chapter 2 – Storage Structures and B-Tree Motivation

Modern storage engines are built on top of fundamental data structures, but these structures alone do not define database semantics such as caching, recovery, or transactionality. Instead, storage engines extend them with additional mechanisms. Despite the diversity of storage engines, most of their design trade-offs can be understood through three core variables: **buffering**, **mutability (immutability)**, and **ordering**. These concepts heavily influence why B-Trees exist, why many variants emerged, and why alternative structures such as LSM Trees continue to evolve.

### Core Variables of Storage Structures

#### 1. Buffering
Buffering defines whether a storage structure accumulates data in memory before persisting it to disk. While all disk-based systems must buffer at least one block (since disks operate on blocks), *avoidable buffering* is a deliberate design choice.

- Buffering amortizes I/O costs by converting many small writes into fewer large ones.
- In B-Trees, buffering can be added at node level (e.g., lazy B-Trees).
- In LSM Trees, buffering is fundamental: writes are first stored in memory and later flushed to disk as immutable files.

Buffering improves write efficiency but introduces complexity in flushing, recovery, and memory management.

#### 2. Mutability vs Immutability
This variable defines whether data is updated **in place** or written **append-only**.

- **Mutable (in-place update)** structures modify data directly at its disk location.
- **Immutable** structures never overwrite existing data; updates are appended or written using copy-on-write.

Although B-Trees are traditionally mutable, immutable variants exist (e.g., Bw-Trees). LSM Trees strongly embrace immutability, which simplifies crash recovery and favors sequential writes but requires background compaction.

#### 3. Ordering
Ordering determines whether records are stored on disk in key order.

- **Ordered storage** allows efficient range scans (a key advantage of B-Trees).
- **Unordered storage** (often insertion order) enables faster writes but requires external indexing for reads.

Append-only systems like Bitcask and WiscKey favor unordered storage to optimize write throughput.

---

### From Binary Search Trees to Disk-Oriented Structures

Binary Search Trees (BSTs) are efficient **in-memory** structures with average-case lookup complexity of `O(log N)`. However, they suffer from several problems when applied to disk storage:

- **Unbalanced trees** degrade to `O(N)` complexity.
- **Low fanout (2 children per node)** leads to high tree height.
- **Poor locality** causes child pointers to span multiple disk pages.
- **Frequent rebalancing** results in costly pointer updates.

Even balanced variants like AVL or 2–3 Trees remain impractical for disk use due to their small node sizes and high pointer overhead.

### Disk-Based Constraints

Disk-resident data structures must respect hardware realities:

#### Hard Disk Drives (HDDs)
- Random I/O is expensive due to seek and rotation latency.
- Sequential I/O is significantly cheaper.
- Data is transferred in fixed-size sectors (typically 512B–4KB).

#### Solid State Drives (SSDs)
- No mechanical seek cost, but still operate on pages and erase blocks.
- Pages must be written sequentially within erased blocks.
- Garbage collection and write amplification can degrade performance.

Both HDDs and SSDs expose a **block-based abstraction**, meaning entire blocks must be read even if only a small portion is needed.

---

### Design Goals for On-Disk Trees

To be efficient on disk, a data structure must:

- Minimize the number of disk accesses
- Maximize locality
- Reduce tree height
- Use high fanout to store many keys per node
- Limit pointer overhead and rebalancing cost

These requirements directly conflict with traditional BST designs.

---

### Why B-Trees?

B-Trees were introduced in 1971 by Bayer and McCreight to address disk-specific constraints. They combine several critical properties:

- **High fanout** → fewer levels → fewer disk seeks
- **Balanced structure** → guaranteed logarithmic height
- **Ordered keys** → efficient point lookups and range scans
- **Page-aligned nodes** → optimal block utilization

By increasing fanout and reducing height, B-Trees dramatically lower the number of disk I/O operations required for searches and updates. This makes them the dominant structure in disk-based databases.

Paged binary trees partially improve locality by grouping nodes into pages, but they still suffer from pointer overhead and costly rebalancing. B-Trees generalize this idea and fully embrace disk-aware design.

---

### Summary

B-Trees emerge naturally when storage structures are designed with disk constraints in mind. The three fundamental variables—**buffering**, **immutability**, and **ordering**—explain most modern storage engine trade-offs. B-Trees represent a balanced point in this design space, which explains both their longevity and the continuous evolution of their variants.

Understanding these foundations is essential before exploring advanced B-Tree optimizations and alternative storage structures.
