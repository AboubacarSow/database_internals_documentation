# Data Files and Index Files

## 1. Why File Organization Matters in a DBMS

The primary goal of a database system is to:

* **Store data**
* **Allow fast and efficient access to it**

A natural question arises:
Why do we need a **DBMS** instead of just storing data in flat files managed by the filesystem?

The answer lies in **file organization**.

Database systems do use files, but they **do not rely on filesystem hierarchies** to locate records. Instead, they use **implementation-specific file formats** designed to optimize database workloads.

---

## 2. Motivation for Specialized File Organization

Using specialized file organization instead of flat files provides three key benefits:

### 2.1 Storage Efficiency

* Data is laid out to minimize overhead per record
* Reduces wasted space and metadata duplication

### 2.2 Access Efficiency

* Records can be located in the **smallest possible number of steps**
* Avoids scanning entire files for every query

### 2.3 Update Efficiency

* Updates are applied in a way that minimizes disk modifications
* Reduces I/O amplification and fragmentation

---

## 3. Tables, Records, and Files

Database systems store:

* **Records** (rows), consisting of multiple fields
* Records are grouped into **tables**
* Each table is typically represented as a **separate file**

Each record can be identified using a **search key**.

To efficiently locate records by search key, databases rely on **indexes**.

---

## 4. Separation of Data Files and Index Files

Most database systems **separate concerns**:

* **Data files** store the actual data records
* **Index files** store metadata that helps locate records in data files

Key properties:

* Index files are typically **smaller** than data files
* Indexes avoid full table scans on every access

---

## 5. Pages and Record Management

Files are partitioned into **pages**:

* Pages are fixed-size units
* Typically equal to one or multiple disk blocks

Pages can be organized as:

* Sequences of records
* **Slotted pages**, which allow flexible record sizes

---

## 6. Inserts, Updates, and Deletions

### 6.1 Representation of Changes

Insertions and updates are represented as **key/value pairs**.

Most modern storage engines:

* Do **not** delete records immediately
* Use **deletion markers (tombstones)** instead

Tombstones store:

* Key
* Deletion metadata (e.g., timestamp)

---

### 6.2 Garbage Collection

Space occupied by obsolete records is reclaimed via **garbage collection**:

* Pages are read
* Live (non-shadowed) records are rewritten elsewhere
* Shadowed or deleted records are discarded

This avoids expensive in-place modifications.

---

## 7. Data Files (Primary Files)

**Data files** (also called *primary files*) store actual data records.

They can be organized in several ways.

---

### 7.1 Heap-Organized Tables (Heap Files)

* Records have **no required order**
* Typically stored in **write (append) order**
* Easy to insert new records
* No reorganization required when appending pages

Limitation:

* Heap files are **not searchable by themselves**
* Require **additional index structures** to locate records

---

### 7.2 Hash-Organized Tables (Hashed Files)

* Records stored in **buckets**
* Bucket determined by **hash(key)**

Inside a bucket:

* Records may be stored in append order
* Or sorted by key to improve lookup speed

---

### 7.3 Index-Organized Tables (IOTs)

* Data records are stored **directly inside the index**
* Records are stored in **key order**

Advantages:

* Efficient range scans via sequential traversal
* Reduces disk seeks:

  * After locating the key in the index, no separate data file lookup is needed

---

### 7.4 Data Entries and Record Location

* If records are stored separately from indexes:

  * Index entries store **record metadata**, such as:

    * File offsets (row locators)
    * Page IDs
    * Bucket IDs (for hash files)

* In index-organized tables:

  * Index entries contain the **actual data records**

---

## 8. Index Files

### 8.1 Definition

An **index** is a specialized structure that:

* Organizes data records on disk
* Maps keys to locations of records

Indexes facilitate **efficient retrieval** without full scans.

---

### 8.2 Primary and Secondary Indexes

* **Primary index**

  * Built on the primary (data) file
  * Often based on the primary key

* **Secondary index**

  * Built on non-primary keys
  * Used to locate records via alternative attributes

---

### 8.3 Index Entries

Secondary indexes may:

* Point directly to data records (via offsets)
* Or store the **primary key**, requiring an additional lookup

Properties:

* Multiple secondary indexes can reference the same record
* Primary indexes hold **one entry per search key**
* Secondary indexes may hold **multiple entries per search key**

---

## 9. Clustered vs Nonclustered Indexes

### 9.1 Clustered Index

* Data records are stored **in the same order as the index key**
* Often stored in the same file or clustered files
* Improves locality and range scans

### 9.2 Nonclustered Index

* Data records stored separately
* Index order does **not** match data order
* Requires additional lookups

Important note:

* Index-organized tables are **clustered by definition**
* Primary indexes are often clustered
* Secondary indexes are **nonclustered by definition**

---

## 10. Primary Key and Implicit Keys

Most database systems define a **primary key** to uniquely identify records.

If none is specified:

* The storage engine may create an **implicit primary key**
* Example: MySQL InnoDB adds an auto-increment column

---

## 11. Primary Index as an Indirection Layer

There are two main approaches to locating records:

### 11.1 Direct Referencing

* Secondary index points directly to data record offsets
* Fewer disk seeks
* Higher cost when records are relocated

### 11.2 Indirection via Primary Index

* Secondary index stores primary key
* Lookup path:

  ```
  Secondary Index → Primary Index → Data Record
  ```
* Reduces pointer update costs
* Increases read latency

Example:

* MySQL InnoDB uses this approach
* Adds an extra primary index lookup

---

### 11.3 Hybrid Approach

Some systems:

* Store both offsets and primary keys
* Validate offset first
* Fall back to primary index if offset is invalid
* Update index entry if relocation occurred

---

## 12. Mental Model Summary

| Concept         | Role                             |
| --------------- | -------------------------------- |
| Data file       | Stores actual records            |
| Index file      | Maps keys to records             |
| Page            | Unit of disk I/O                 |
| Heap file       | Unordered records                |
| Hashed file     | Key → bucket                     |
| IOT             | Data stored in index             |
| Primary index   | Main access path                 |
| Secondary index | Alternative access paths         |
| Clustered index | Data ordered by key              |
| Indirection     | Trade-off between reads & writes |

---

## One-Sentence Intuition

> **Data files store records; index files define how fast and efficiently those records can be found.**

