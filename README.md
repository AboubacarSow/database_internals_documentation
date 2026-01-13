# 🗄️ Database Internals: Study Notes & Maps

> "Every database is a set of engineering trade-offs."

This repository is a structured exploration of **Alex Petrov’s *Database Internals***. Rather than a mechanical summary, these notes serve as a "system-level compass" for understanding how modern databases are built, from the disk to the network.

---

## 🧭 The Core Philosophy

I am documenting this journey to transform "black box" technologies into a clear **mental model**. These notes focus on:

* **The "Why" over the "How":** Identifying the problems that led to specific designs.
* **First Principles:** Understanding durability, consistency, and storage structures.
* **Trade-off Analysis:** Evaluating what is sacrificed (e.g., Read Latency vs. Write Throughput).

---

## 🗺️ Roadmap & Progress

I’ve organized the notes into two primary pillars, mirroring the book’s structure but flavored with custom maps and reflections.

### Part I: Storage Engines

*Focus: Data structures, disk management, and local persistence.*

* [ ] **01. Introduction** – Architecture and classifications.
* [ ] **02. B-Tree Basics** – Ubiquitous storage structures.
* [ ] **03. LSM Trees** – Optimizing for write-heavy workloads.
* [ ] **04. Disk-Based Storage** – Layouts and compression.

### Part II: Distributed Systems

*Focus: Consistency, consensus, and the challenges of scale.*

* [ ] **08. Fundamentals** – Failure models and networks.
* [ ] **09. Replication** – Leaderless, Master-Slave, and Multi-master.
* [ ] **10. Consensus** – Paxos, Raft, and the cost of agreement.

---

## 🗂️ Repository Layout

```bash
.
├── 📂 notes/   
└──  README.md           

```

---

## 🧠 Methodology

To ensure these notes are durable for "future-me," each chapter follows a four-layer framework:

| Layer | Focus |
| --- | --- |
| **Conceptual Map** | High-level boundaries and components. |
| **Core Idea** | The fundamental problem being solved. |
| **Trade-offs** | What we gain and what we lose (The "Engineering" part). |
| **Connections** | How this component plugs into the rest of the system. |

---

## 🛠️ Usage for "Future-Me"

These notes are specifically designed to be referenced during:

1. **System Design:** Choosing the right storage engine for a specific workload.
2. **Production Debugging:** Understanding how underlying structures (like WAL or Checkpoints) behave under stress.
3. **Architectural Decisions:** Deciding between consistency models in distributed environments.

---
## 🚧 Work in Progress

This repository evolves as I read, test, and reflect. Notes may be rewritten multiple times as understanding improves — this is intentional.

## 📌 References & Disclaimer

* **Primary Source:** [*Database Internals* by Alex Petrov](https://www.databass.dev/) (O'Reilly Media).
* **Disclaimer:** These are personal study notes. They are meant to supplement, not replace, the original text. I highly recommend purchasing the book for the full technical context.

---

**Are you also studying database internals?** I'd love to discuss trade-offs or clarify concepts. Feel free to open an **Issue** or a **Discussion** thread!