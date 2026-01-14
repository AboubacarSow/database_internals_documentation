# DBMS Architecture — Conceptual Overview

## 1. No Universal DBMS Blueprint

There is no single, universally accepted architecture for database management systems.

* Each DBMS is designed differently
* Component boundaries are often **conceptual**, not strict.
* In real implementations:

  * Components may be tightly coupled
  * Abstractions may be violated for performance
  * Edge cases and optimizations blur module boundaries

Architecture diagrams should therefore be treated as **mental models**, not literal representations of code structure.

---

## 2. Client–Server Model

Database management systems typically follow a **client/server architecture**:

* **Clients**: application instances issuing queries
* **Servers (nodes)**: database system instances handling requests

In distributed deployments, database nodes also communicate with each other over a database cluster.

---

## 3. High-Level Request Flow

A query typically flows through the system as follows:

1. Transport subsystem
2. Query processor
3. Query optimizer
4. Execution engine
5. Storage engine

Each stage progressively reduces ambiguity and increases execution commitment.

---

## 4. Transport Subsystem

**Responsibilities:**

* Accepts incoming client requests
* Handles inter-node communication in a cluster
* Transfers queries into the database system

The transport layer does not interpret or execute queries; it only delivers them to internal components.

---

## 5. Query Processor

**Responsibilities:**

* Parses query syntax - acts as a compiler
* Interprets query semantics
* Validates query correctness
* Performs access control checks

Some checks (e.g., permissions) can only be completed after the query is fully interpreted.

---

## 6. Query Optimizer

The query optimizer is responsible for determining **how** a query should be executed.

**Key tasks:**

* Eliminates redundant or impossible operations - acts here as a filter
* Rewrites queries for efficiency
* Uses internal statistics such as:

  * Index cardinality
  * Estimated result sizes
  * Data distribution
* Considers data placement in distributed systems

**Output:**
An **execution plan** (query plan), describing a sequence of operations required to produce the query result.

Multiple execution plans may exist for the same query; the optimizer selects the most efficient one.

It acts as **the brain of DBMS**.

---

## 7. Execution Engine

**Responsibilities:**

* Executes the chosen execution plan
* Coordinates local and remote operations
* Handles data access across nodes
* Collects and returns final results
* It's the **Orchestrator**

The **Execution** engine does not optimize queries; it follows the plan selected by the optimizer. So it trusts the **Query optimizer**

---

## 8. Storage Engine

The storage engine is responsible for **persistent data management** and where abstraction collapses to physics.

It operates at a lower abstraction level than query processing and does not understand schemas or query semantics.

### 8.1 Core Responsibilities

* Storing data on disk
* Managing in-memory data
* Ensuring durability and crash recovery
* Supporting concurrent access

---

### 8.2 Internal Components

#### Transaction Manager

* Schedules transactions
* Ensures atomicity and logical consistency
* Prevents inconsistent database states

#### Lock Manager

* Controls concurrent access to database objects
* Prevents physical data corruption
* Works together with the transaction manager

Together, the transaction manager and lock manager implement **concurrency control**.

---

#### Access Methods (Storage Structures)

* Define how data is organized and accessed on disk
* Examples:
  * Heap files
  * B-Trees
  * LSM Trees

---

#### Buffer Manager

* Caches disk pages in memory
* Manages memory usage
* Plays a critical role in performance
* Decides:
    - What stays in RAM
    - What gets evicted
---

#### Recovery Manager

* Maintains operation logs
* Restores database state after crashes
* Ensures durability

This is where: “the database remembers”
---

## 9. Key Conceptual Distinction

* **Upper layers** (query processor, optimizer, execution engine) operate on **logical abstractions**
* **Storage engine** operates on **granular data units**:

  * Records
  * Pages
  * Bytes

This separation explains why storage engines provide simple data manipulation APIs, while higher-level components handle schemas, queries, and transactions.

---

## 10. Key Takeaways

* DBMS architecture is **conceptual**, not **rigid**
* A database system is a coordinated pipeline of subsystems
* Storage engines form the foundation of the database system
* Understanding storage internals is essential for reasoning about correctness, performance, and failure recovery


