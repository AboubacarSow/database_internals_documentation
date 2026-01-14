# Row-Oriented vs Column-Oriented DBMS

## 1. Basic Data Model

Relational databases organize data into **tables** composed of:

* **Columns**: attributes with a uniform data type
* **Rows**: logical records, usually identified by a key
* **Fields**: intersection of a row and a column (a single value)

Example:

* All `Name` values belong to the same column and share a type
* All values describing one user form a row

---

## 2. Classification by Physical Data Layout

One major way to classify DBMSs is **how table data is physically stored on disk**:

### Row-Oriented Layout (Horizontal Partitioning)

* Values belonging to the **same row** are stored together
* Entire records are colocated

### Column-Oriented Layout (Vertical Partitioning)

* Values belonging to the **same column** are stored together
* Columns are stored independently

> Logical tables may look identical, but **physical layout differs radically**.

---

## 3. Row-Oriented DBMS

### 3.1 Physical Layout

Row-oriented systems store complete records contiguously:

```
[ID | Name | Birth Date | Phone]
[ID | Name | Birth Date | Phone]
[ID | Name | Birth Date | Phone]
```

Example record:

```
| 10 | John | 01 Aug 1981 | +1 111 222 333 |
```

### 3.2 Strengths

Row-oriented layout is ideal when:

* Records are accessed **as a whole**
* Most queries fetch **multiple fields of a single row**
* Workloads are **transactional (OLTP)**

Reasons:

* High **spatial locality**
* Disk blocks contain all fields of a record
* Efficient inserts and updates of full records

### 3.3 Limitations

* Disk I/O is block-based
* Fetching a single column across many rows requires:

  * Reading unnecessary fields
  * Wasting memory bandwidth and cache space

Example inefficiency:

* Querying *only phone numbers* still loads names, birth dates, etc.

---

## 4. Column-Oriented DBMS

### 4.1 Physical Layout

Column-oriented systems store column values contiguously:

```
ID:     1, 2, 3, 4
Symbol: DOW, DOW, S&P, S&P
Date:   08 Aug, 09 Aug, 08 Aug, 09 Aug
Price:  24314.65, 24136.16, 2414.45, 2232.32
```

Each column is stored in a separate segment or file.

---

### 4.2 Strengths

Column-oriented layout excels when:

* Queries scan **many rows**
* Only a **subset of columns** is required
* Workloads are **analytical (OLAP)**

Typical use cases:

* Aggregations (AVG, SUM, COUNT)
* Trend analysis
* Reporting and analytics

Advantages:

* Reduced I/O (read only relevant columns)
* Better cache utilization
* Efficient sequential access

---

## 5. Tuple Reconstruction

Since values of a row are stored separately:

* Column stores must **reconstruct rows** for:

  * Joins
  * Filters
  * Multi-column results

### Mapping Techniques

1. **Explicit identifiers**

   * Each value stores a row key
   * Causes duplication and storage overhead

2. **Implicit identifiers (Virtual IDs)**

   * Use the value’s **position (offset)** within a column
   * More space-efficient and commonly used

---

## 6. Column Store Optimizations

Columnar databases go beyond layout differences.

### 6.1 CPU Efficiency

* Sequential column scans improve cache locality
* Enable **vectorized execution**
* One CPU instruction processes multiple values (SIMD)

### 6.2 Compression

* Same-type data stored together
* Higher compression ratios
* Different compression algorithms per data type

Examples:

* Run-length encoding
* Dictionary encoding
* Delta encoding

---

## 7. Choosing Between Row and Column Stores

The choice depends on **access patterns**, not ideology.

### Prefer Row-Oriented DBMS when:

* Queries operate on complete records
* Workload is insert/update heavy
* Mostly point queries or small range scans
* OLTP systems

### Prefer Column-Oriented DBMS when:

* Queries scan many rows
* Only a subset of columns is accessed
* Heavy use of aggregates
* OLAP and analytical systems

---

## 8. Popular Systems

### Row-Oriented

* MySQL
* PostgreSQL
* Traditional RDBMSs

### Column-Oriented

* MonetDB
* C-Store / Vertica
* ClickHouse
* Apache Kudu
* Columnar formats: Parquet, ORC, RCFile

---

## 9. Wide Column Stores (Important Distinction)

Column-oriented DBMS **≠** Wide column stores.

### Characteristics of Wide Column Stores

* Data model: **multidimensional sorted map**
* Columns grouped into **column families**
* Within a column family, data is stored **row-wise**
* Optimized for key-based access

Examples:

* Google BigTable
* Apache HBase

---

## 10. BigTable Example (Webtable)

Conceptually:

```
Row Key (Reversed URL)
  └── Column Family (contents, anchors)
        └── Column Qualifier + Timestamp
              └── Value
```

Physical layout:

* Column families stored separately
* Within each family, values for the same row key are colocated
* Multiple versions stored by timestamp

This design:

* Enables fast lookup by key
* Supports versioned data
* Is not optimized for analytical scans like true column stores

---

## Mental Model Summary

| Aspect            | Row-Oriented | Column-Oriented |
| ----------------- | ------------ | --------------- |
| Storage unit      | Row          | Column          |
| Best for          | OLTP         | OLAP            |
| I/O pattern       | Record-based | Scan-based      |
| Compression       | Limited      | Excellent       |
| CPU vectorization | Limited      | Strong          |
| Example systems   | PostgreSQL   | ClickHouse      |

### Disk I/O intuition
```
Disk reads blocks, not values.

Row Store:
[ID | Name | DOB | Phone]  <-- block

Column Store:
[Phone | Phone | Phone]    <-- block
```
---

### One-Sentence Intuition

> **Row stores optimize for record access; column stores optimize for data processing.**

