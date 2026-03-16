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
| [⚡ Distributed Cache](use-cases/distributed-cache-system-design.md) | Caching at 1M reads/sec. Cache-aside vs write-through, Redis vs Memcached, consistent hashing, stampede prevention, eviction policies | ⭐⭐⭐ Senior |

> More guides coming soon. Each one added after a real design session.

---

## 🗂️ Repo Structure

```
software-design/
│
├── use-cases/                               # One folder per design problem
│   ├── rate-limiter-system-design.md
│   └── distributed-cache-system-design.md
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
- [ ] Notification System (push, email, SMS at scale)
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


---

## ✍️ Author

**Manas** — ML/AI engineer, system design practitioner.

---

*If this helped you, star the repo ⭐ — it helps others find it.*
