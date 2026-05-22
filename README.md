# 🏛️ Software Design at Scale

> A living knowledge base for system design — written for engineers who want to go beyond surface-level answers.

[![GitHub Pages](https://img.shields.io/badge/View%20as%20Website-→-0f172a?style=for-the-badge&logo=github)](https://simplymanas.github.io/software-design)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)](LICENSE)
![Last Updated](https://img.shields.io/badge/Updated-March%202026-6366f1?style=for-the-badge)

---

## 🧭 What Is This?

This repository is a **personal system design notebook** — every document here is a deep-dive written after actually working through the design problem from scratch, not copied from a tutorial.

Each guide covers:
- **Clarifying questions** a senior engineer asks before designing
- **Algorithm / pattern selection** with explicit trade-offs
- **Data layer choices** with capacity estimates
- **Failure modes & resilience** strategies
- **API contracts** (HTTP + gRPC)
- **Observability** — metrics, alerts, dashboards
- **Interview cheat sheet** at the end of every guide

---

## 📚 Design Guides

### Infrastructure & Reliability

| Guide | Description | Difficulty |
|-------|-------------|------------|
| [🚦 Rate Limiter](use-cases/rate-limiter-system-design.md) | Distributed rate limiting at 500M req/day. Token bucket, Redis Cluster, Lua atomicity, circuit breakers | ⭐⭐⭐ Senior |
| [⚡ Distributed Cache](use-cases/distributed-cache.md) | Caching at 1M reads/sec. Cache-aside vs write-through, Redis vs Memcached, consistent hashing, stampede prevention, eviction policies | ⭐⭐⭐ Senior |
| [🔔 Notification System](use-cases/notification-system.md) | Push, email, SMS, in-app at 100B/month. Kafka pipeline, delivery guarantees, dedup, preferences, rate limiting, DLQ | ⭐⭐⭐ Senior |

### E-Commerce & Retail Systems

| Guide | Description | Difficulty |
|-------|-------------|------------|
| [⚡ Hyper-Local Flash Inventory Sync](use-cases/flash-inventory-sync.md) | Preventing overselling across 5,000 dark stores. Redis Lua atomics, Redlock, reservation TTL, event sourcing, CRDT, flash sale virtual queues | ⭐⭐⭐⭐⭐ Principal |

### Machine Learning & AI Systems

| Guide | Description | Difficulty |
|-------|-------------|------------|
| [🧠 How LLMs Work](use-cases/how-llm-works.md) | Transformer architecture, attention mechanism, pre-training pipeline, RLHF, inference, RAG, serving at scale | ⭐⭐⭐⭐ Staff |
| [🔍 Vector Database Design](use-cases/vector-database-design.md) | HNSW, IVF-PQ, embeddings, filtered search, hybrid sparse+dense, RAG pipelines, Pinecone vs Qdrant vs pgvector, 1B-vector design | ⭐⭐⭐⭐ Staff |

### Fintech & Payments

| Guide | Description | Difficulty |
|-------|-------------|------------|
| [💸 How UPI Works](use-cases/how-upi-works.md) | NPCI architecture, VPA resolution, 2FA, idempotency, Google Pay vs PhonePe vs Paytm internals, fraud detection at scale | ⭐⭐⭐ Senior |

### Payments & Fintech

| Guide | Description | Difficulty |
|-------|-------------|------------|
| [💸 How UPI Works](use-cases/how-upi-works.md) | NPCI architecture, VPA resolution, P2P/QR/Collect flows, Google Pay/PhonePe/Paytm internals, MPIN security, 10B txn/month scalability | ⭐⭐⭐⭐ Staff |

> More guides coming soon. Each one added after a real design session.

---

## 🗂️ Repo Structure

```
software-design/
│
├── use-cases/                               # One folder per design problem
│   ├── rate-limiter-system-design.md
│   ├── distributed-cache.md
│   ├── notification-system.md
│   ├── flash-inventory-sync.md
│   ├── how-llm-works.md
│   ├── vector-database-design.md
│   └── how-upi-works.md
│
├── docs/                                    # GitHub Pages website source
│   └── index.html
│
├── README.md                                # You are here
└── LICENSE
```

---

## 🧠 How to Use This Repo

**For interview prep:**
1. Read the guide end-to-end once
2. Close it and whiteboard the architecture from memory
3. Re-read and check what you missed
4. Repeat the Interview Cheat Sheet section until fluent

**For production reference:**
- Jump directly to the section you need (data layer, failure modes, API contract)
- Each guide has numbered sections for quick navigation

---

## 🗺️ Roadmap — Upcoming Guides

- [x] ~~Distributed Cache (Redis vs Memcached, eviction, consistency)~~ ✅
- [x] ~~Notification System (push, email, SMS at scale)~~ ✅
- [ ] URL Shortener (hashing, redirects, analytics)
- [ ] Search Autocomplete (trie, prefix indexing, ranking)
- [ ] Distributed Message Queue (Kafka internals)
- [ ] Web Crawler (politeness, deduplication, frontier)
- [ ] Distributed ID Generator (Snowflake, UUID, ULID)
- [ ] Key-Value Store (LSM trees, SSTables, compaction)

---

## 🌐 GitHub Pages

This repo has a companion website where all guides are browsable with a clean UI.

**→ [simplymanas.github.io/software-design](https://simplymanas.github.io/software-design)**

To enable GitHub Pages on your fork:
1. Go to **Settings → Pages**
2. Source: **Deploy from branch**
3. Branch: `main` | Folder: `/docs`
4. Save — site is live in ~60 seconds

---

## ✍️ Author

**Manas** — ML/AI engineer, system design practitioner.

---

*If this helped you, star the repo ⭐ — it helps others find it.*
