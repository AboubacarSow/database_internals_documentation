## Chapter 2 – Inside B-Trees (Ubiquitous B-Trees)

B-Trees can be understood as **hierarchical navigation structures**, similar to a library catalog: each level narrows the search space until the desired record is found. They build upon balanced search trees but are optimized for disk-based storage through **high fanout**, **low height**, and **page-oriented design**.

---

### Structural Characteristics of B-Trees

B-Trees are **balanced, ordered, multiway search trees** designed for external storage.

Key properties:

* **High fanout**: each node contains many keys and child pointers
* **Low height**: reduces disk seeks during traversal
* **Sorted keys**: allow binary search within nodes
* **Page-based layout**: nodes correspond to fixed-size disk pages

Unlike binary search trees, B-Trees store **multiple keys per node**, splitting the key space into many subranges at each level.

---

### B-Tree Node Hierarchy

A B-Tree consists of three logical node types:

* **Root node**

  * Top-level node
  * Has no parent
* **Internal nodes**

  * Connect root to leaves
  * Store only separator keys and child pointers
* **Leaf nodes**

  * Bottom level
  * Store actual key–value pairs (in B+ Trees)

Because B-Trees are page-oriented, the terms *node* and *page* are often used interchangeably.

#### Logical Hierarchy (ASCII)

```
                    [ Root ]
                       |
               -------------------
               |                 |
         [ Internal ]       [ Internal ]
               |                 |
        ----------------   ----------------
        |              |   |              |
     [ Leaf ]       [ Leaf ]       [ Leaf ]
```

---

### Fanout and Occupancy

* **Fanout**: maximum number of keys stored in a node
* **Occupancy**: ratio of used space to total node capacity

Higher fanout:

* Reduces tree height
* Decreases number of disk accesses
* Amortizes rebalancing costs

B-Trees intentionally leave **free space inside nodes** to accommodate future inserts.
Storage utilization may drop to ~50% in worst cases, but this **does not hurt performance**, since disk I/O dominates cost.

---

### B-Trees vs B+ Trees

The term *B-Tree* is often used as an umbrella term.
In practice, most DBMSs use **B+ Trees**.

Differences:

* **B-Trees**: values may appear in any node
* **B+ Trees**: values appear **only in leaf nodes**
* Internal nodes store **only separator keys**

Advantages of B+ Trees:

* All data access happens at leaf level
* Simpler update and delete logic
* Efficient range scans

Most modern databases (e.g., InnoDB, PostgreSQL, SQL Server) use **B+ Trees**, even if they call them B-Trees.

---

### Separator Keys and Tree Invariants

Keys stored in internal nodes are called **separator keys**.

They define subtree boundaries:

```
            [ K1 | K2 | K3 ]
             /    |    |    \
           <K1  K1–K2  K2–K3  ≥K3
```

More formally:

* First pointer → keys `< K₁`
* Middle pointers → `Kᵢ₋₁ ≤ key < Kᵢ`
* Last pointer → keys `≥ Kₙ`

Keys inside each node are stored in **sorted order**, enabling binary search.

---

### Leaf-Level Linking (B+ Trees)

Most implementations maintain **sibling pointers between leaf nodes** to support efficient range scans.

```
[ Leaf ] <-> [ Leaf ] <-> [ Leaf ] <-> [ Leaf ]
```

This allows sequential access **without re-traversing the tree**.

---

### Growth Direction

Unlike binary search trees (which grow top-down), B-Trees grow **bottom-up**:

* Inserts always start at a leaf
* Internal nodes grow only when children split
* Tree height increases **only when the root splits**

This guarantees minimal height.

---

### B-Tree Lookup Complexity

B-Tree lookup cost can be analyzed from two perspectives.

#### Disk Accesses (I/O Complexity)

* One page read per tree level
* Complexity: `O(log_N M)`

  * `N`: keys per node (fanout)
  * `M`: total number of keys

#### Key Comparisons (CPU Complexity)

* Binary search inside nodes
* Complexity: `O(log₂ M)`

In practice, **disk I/O dominates**, which is why high fanout is critical.

---

### B-Tree Lookup Algorithm

Lookup is a **single root-to-leaf traversal**:

```
          [ Root ]
             |
        [ Internal ]
             |
          [ Leaf ]
```

Steps:

1. Start at root
2. Binary search separator keys
3. Follow selected child pointer
4. Repeat until leaf
5. Locate key (point query) or starting position (range query)

Range scans continue via **leaf sibling pointers**.

---

### Key and Pointer Counts

Different descriptions exist, but semantics are equivalent.

In this book:

* A node holds up to **N keys**
* A node holds up to **N + 1 child pointers**

The root node may have fewer keys than internal nodes.

---

### B-Tree Node Splits (Insertion)

Insertion steps:

1. Locate target leaf
2. Insert key–value pair
3. Check for overflow

Overflow:

* Leaf: more than `N` keys
* Internal: more than `N + 1` pointers

Split process (conceptual):

```
Before split:
[ 10 | 20 | 30 | 40 | 50 ]

After split:
[ 10 | 20 ]     [ 40 | 50 ]
        ↑
   promoted key (30)
```

The promoted key is inserted into the parent.
If the parent overflows, the split propagates upward.
If the root splits, a **new root** is created and height increases.

---

### B-Tree Node Merges (Deletion)

Deletion steps:

1. Locate target leaf
2. Remove key–value pair
3. Check for underflow

If underflow occurs:

* Borrow from sibling **or**
* Merge with sibling

Merge example:

```
Before merge:
[ 10 | 20 ]   [ 30 | 40 ]

After merge:
[ 10 | 20 | 30 | 40 ]
```

The separator key is removed from the parent.
Merges may propagate upward and can reduce tree height if the root collapses.

---

### Summary

B-Trees (specifically B+ Trees) achieve efficient disk-based indexing by:

* Maximizing fanout
* Minimizing tree height
* Aligning nodes with disk pages
* Separating navigation from data storage

Their predictable behavior, strong locality, and excellent range-query performance make them the **dominant indexing structure in modern DBMSs**.

