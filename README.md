
act as a an expert architect and a software pro on system design, judge me on my skill and than guide me.

21:33

Prepared to evaluate architectural and design expertise comprehensively
Prepared to evaluate architectural and design expertise comprehensively
Love the ambition! Before I can judge you fairly and guide you effectively, I need to get a baseline on where you stand.

Let me ask a few quick calibration questions:

Answer these and I'll give you an honest, no-fluff assessment — and a sharp roadmap tailored to exactly where you are. 🎯

Q: How would you rate your current system design knowledge?
A: Advanced – done it in production

Q: Which areas do you have hands-on experience with? (Select all that apply)
A: Distributed systems & scalability, Caching & messaging queues, APIs & microservices, Databases & storage

Q: What's your goal with system design?
A: Pure learning & mastery

21:33

Assessed advanced expertise to calibrate appropriate challenge level
Assessed advanced expertise to calibrate appropriate challenge level
Solid foundation across the board. But self-ratings are just a starting point — let me actually test you before I judge you.

🏛️ The Architect's Bench — Round 1
Here's your problem. Treat this like a senior staff engineer interview at a top-tier company. I want your design, not just your answer.

🔧 Problem: Design a Rate Limiter as a Distributed Service
You're building a standalone rate limiting service used by 50+ internal microservices at a company processing 500 million API requests/day. It must be accurate, fast (sub-millisecond overhead), fault-tolerant, and horizontally scalable.

📋 What I want from you:
Clarify assumptions — what questions would you ask first?
Algorithm choice — which rate limiting algorithm and why? Trade-offs?
Data layer — what stores the counters? Why? What happens if it goes down?
Distributed consistency — how do you handle race conditions across nodes?
Failure modes — what's your strategy when the limiter itself is unavailable?
API contract — what does the interface look like to consuming services?
Observability — how do you know it's working correctly at scale?
🎯 Ground rules:
No hand-waving. Justify every choice.
Call out trade-offs explicitly.
Think out loud — your reasoning matters as much as the answer.
Take your time. Give me your full design. I'll score you across 6 dimensions when you're done. 👇

cna you give me some hints, as its first time

21:35

Devised structured hints for rate limiter design problem
Devised structured hints for rate limiter design problem
Of course! Here are nudges — enough to get you thinking, not enough to do the work for you. 😄

💡 Hint Sheet — Rate Limiter Design
1. Clarifying Assumptions
Think about: Who is being rate limited? A user? An IP? An API key? And is the limit per second, per minute, per day? Also — should it be exact or is approximate okay?

2. Algorithm Choice
There are 4 classic algorithms. Each has a sweet spot:

Algorithm	Hint
Fixed Window	Simple, but has a nasty edge case at window boundaries
Sliding Window Log	Accurate, but think about memory cost
Sliding Window Counter	A clever hybrid — approximates sliding with less memory
Token Bucket	Great for bursty traffic — tokens refill over time
Leaky Bucket	Smooths output — think of it as a queue
🤔 Ask yourself: at 500M requests/day, do I care more about accuracy or speed?

3. Data Layer
Think: where do you store counters? You need something fast and shared across nodes.

🤔 Hint: What in-memory store do you already know that's blazing fast and supports atomic operations?

Also — what if that store goes down? Do you fail open (allow requests) or fail closed (block everything)?

4. Distributed Consistency
If 3 nodes each see a counter of 99 and the limit is 100 — all 3 might allow request 100 simultaneously. That's 3 requests past the limit.

🤔 Hint: Look up Redis INCR + EXPIRE — it's atomic. Or think about Lua scripts in Redis for multi-step atomicity.

5. Failure Modes
Your rate limiter is now in the critical path of every request. If it dies, what happens to your 50 services?

🤔 Ask yourself: Is it safer to let traffic through or block it during an outage? What does your business need?

Also think: circuit breaker pattern. Ring a bell?

6. API Contract
Keep it simple. Your consumers just want to know two things:

"Am I allowed?"
"How long until I can try again?"
🤔 Hint: Look at how real HTTP rate limit headers work — X-RateLimit-Remaining, Retry-After

7. Observability
You can't fix what you can't see. Think about:

Metrics — what numbers matter? (hint: rejection rate, latency)
Alerts — when should someone wake up at 2am?
Dashboards — per-client, per-endpoint views
🗺️ Suggested Flow to Answer
Clarify → Pick Algorithm → Pick Storage → 
Handle Race Conditions → Handle Failures → 
Define API → Add Observability
Now take a shot at it — even a rough answer is great. I'll judge you fairly and guide you on every gap. 🎯

use case
07:20

Thinking about defining and exploring practical applications
Thinking about defining and exploring practical applications
Claude's response was interrupted
• A rate limiter prevents DoS attacks, intentional or unintentional, by blocking the excess calls. • Reduces cost where the system is using a 3rd-party API service and is charged on a per-call-basis. • To reduce server load, a rate limiter is used to filter out excess requests caused by bots or users’ misbehaviour.  

image 1
  We use the token bucket algorithm to do rate limiting. This algorithm has a centralized bucket host where you take tokens on each request, and slowly drip more tokens into the bucket. If the bucket is empty, reject the request. In our case, every Stripe user has a bucket, and every time they make a request we remove a token from that bucket. We implement our rate limiters using Redis.   The bucket has tokens with a pre-defined capacity. When a request comes in it takes a token from the bucket and is processed further. If there is no token to be picked, the request is dropped and the user will have to retry. Example use-case:     • We have our rate-limiting rules set to 3 requests per user per minute.     • User1 makes the request at 00 time interval, currently the bucket is full with 3 tokens so request will be processed. Tokens will be updated in the bucket to 2 now.     • User1 makes a 2nd request at 10th sec, bucket has 2 tokens so again the request will be processed further. Tokens will be updated in the bucket to 1 now.     • User1 makes a 3rd request at say 30th sec, bucket has 1 token so again the request will be processed further. Now the bucket is empty till the whole 1 minute completes.     • User1 makes a 4th request at say 55th sec, the bucket has 0 tokens so the request is throttled and the user is returned with a 429 status code — too many requests and asked to retry it later in some time. The HTTP 429 Too Many Requests response status code indicates the user has sent too many requests in a given amount of time.     • At the completion of that 1 minute, the token count is refreshed at a fixed rate and the bucket is again full with 3 tokens and requests can be processed again for that particular user. 

Race condition    
image 2

After things are in place, we can also monitor our rate limiter for metrics like performance, rules, algorithm effectiveness, etc. For example, if rate-limiting rules are too strict, many valid requests are dropped. In this case, we want to relax the rules a little bit. In another example, we notice our rate limiter becomes ineffective when there is a sudden increase in traffic like flash sales. In this scenario, we may replace the algorithm to support burst traffic. The token bucket is a good fit here.

Show more
07:24

Evaluated response across seven dimensions, identified gaps in distributed consistency and failure handling
Evaluated response across seven dimensions, identified gaps in distributed consistency and failure handling
⚖️ Architect's Verdict — Your Design Review
Good first submission. The diagrams are clean and the token bucket example is well-walked. Now here's my honest, dimension-by-dimension judgment.

📊 Scorecard
Dimension	Score	Verdict
Use Case & Clarifications	6/10	Partial
Algorithm Choice	8/10	Solid
Data Layer	5/10	Surface-level
Distributed Consistency	2/10	Almost missing
Failure Modes	1/10	Not addressed
API Contract	4/10	Only 429 mentioned
Observability	5/10	Too vague
Overall: 31/70 — Intermediate, not yet Advanced for this problem

🔍 Detailed Feedback
✅ What You Got Right
Token bucket is a great algorithm pick for burst traffic. You justified it correctly.
The step-by-step example (User1 at 0s, 10s, 30s, 55s) shows you understand the mechanics deeply. Very good.
Redis is the right call for a shared counter store.
You identified 429 as the response — correct.
The diagrams are clean and show you can communicate architecture visually — a real senior skill.
❌ Where You Lost Points
1. Clarifications — 6/10

You assumed per-user rate limiting but never asked:

What is the limit key? User ID? IP? API key? Tenant?
Global vs per-endpoint limits? (e.g., /search might have tighter limits than /ping)
Hard limit or soft limit? (strict cutoff vs gradual throttle)
Synchronous or async enforcement?
At a senior level, clarifying scope before designing is a signal of maturity. Never skip this.

2. Data Layer — 5/10

You said Redis. Good. But stopped there. A strong answer includes:

Redis data structure: HASH or STRING with INCR + EXPIRE?
Redis deployment: Single node? Sentinel? Redis Cluster? At 500M req/day, a single Redis node is a SPOF and a bottleneck.
Replication strategy: Are you using read replicas?
What if Redis goes down? — You never answered this. This is critical.
3. Distributed Consistency — 2/10

You wrote "Race condition" as a heading but gave zero solution. This is the heart of the problem at scale.

Here's what was missing:

Thread 1 reads counter = 99  ──┐
Thread 2 reads counter = 99  ──┤  Both think limit not hit
Thread 1 writes counter = 100 ─┤  Both allow the request
Thread 2 writes counter = 100 ─┘  You've exceeded the limit!
The fix: Redis INCR is atomic. Better yet — a Lua script in Redis to bundle read + increment + check in a single atomic operation. This prevents race conditions entirely.

This is what separates a senior from a principal. You must address this.

4. Failure Modes — 1/10

Completely missing. At 500M req/day across 50 services, your rate limiter is in the critical path. If it goes down:

Do you fail open → allow all traffic (risk: abuse)
Do you fail closed → block all traffic (risk: outage)
Do you use a circuit breaker to isolate the failure?
Do consuming services have a local fallback (e.g., in-memory approximate counter)?
A production system without a failure strategy is a disaster waiting to happen.

5. API Contract — 4/10

You only mentioned the 429 status code. A complete contract looks like:

http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1710000060
Retry-After: 30
Also — what does the internal SDK look like for the 50 consuming services?

python
result = rate_limiter.check(user_id="u123", endpoint="/search")
# Returns: { allowed: bool, remaining: int, retry_after_ms: int }
```

---

**6. Observability — 5/10**

You said "monitor for metrics like performance, rules, algorithm effectiveness" — but this is hand-wavy. Be specific:

| What to measure | Why |
|---|---|
| **Rejection rate per client** | Detect abusers or misconfigured limits |
| **Limiter latency (p99)** | Must stay sub-millisecond |
| **Redis hit/miss rate** | Detect caching issues |
| **Fallback activation rate** | Know when your backup kicks in |

Alerts: page on-call if rejection rate spikes >10x in 60s, or if limiter latency crosses 5ms.

---

## 🗺️ Your Learning Path from Here
```
Week 1 → Deep dive: Redis atomicity, Lua scripts, INCR/EXPIRE patterns
Week 2 → Failure patterns: circuit breaker, fail-open vs fail-closed
Week 3 → Design the full API contract + client SDK design
Week 4 → Observability: metrics, SLOs, alerting strategy
🎯 Bottom Line
You have solid fundamentals and good intuition — the token bucket walk-through was genuinely good. But at a principal/staff level, the gaps are in: what happens when things break, race conditions at scale, and precise API contracts.

Ready for Round 2 with a harder problem? Or want to re-attempt this one with the feedback applied? 🔥

can you write this with the design digram so i can study. give it as a md file so that i can keep it github. be elaborative as much as possible with everything in place

09:15

Architected comprehensive rate limiter guide with visual diagrams
Architected comprehensive rate limiter guide with visual diagrams
Now let me build the full markdown file. This will be comprehensive — GitHub-ready with Mermaid diagrams, tables, code, everything.


Created a file, read a file
Created a file, read a file
Here's your complete study file. Here's what's inside:

15 sections covering everything:

All 5 algorithms with visual Mermaid diagrams and trade-off tables
Full Redis data layer design with capacity estimates
The race condition problem + 3 solutions (INCR, Lua scripts, ZSET)
Circuit breaker implementation with code
Complete HTTP + gRPC API contracts
A copy-paste ready Python SDK with middleware
Prometheus metrics + Alertmanager alert rules
A Grafana dashboard layout
Multi-window rate limiting, adaptive limits, rule storage
An interview cheat sheet with numbers to memorize
GitHub tips for this file:

GitHub natively renders Mermaid diagrams — all the flowcharts and sequence diagrams will render beautifully
Add it to a system-design/ folder in your repo for a clean study collection
The quadrantChart and gantt diagrams work on GitHub too
Ready for Round 2 — a harder problem like designing a distributed cache or a notification system? 🔥

Rate limiter system design
Document · MD 
give e comit message for github with decription

09:24

Identified request for GitHub commit message with description
Identified request for GitHub commit message with description
docs: add distributed rate limiter system design study guide

Adds a comprehensive system design document covering the full architecture
of a production-grade distributed rate limiting service handling 500M req/day.

Covers:
- Algorithm deep dive: Token Bucket, Sliding Window, Fixed Window, Leaky Bucket
- Redis Cluster data layer with atomic INCR and Lua script patterns
- Race condition analysis and solutions (atomic ops, sorted sets)
- Failure resilience: circuit breaker, fail-open, local in-memory fallback
- Full API contract: HTTP headers (429, Retry-After, X-RateLimit-*) and gRPC proto
- Client SDK design with middleware patterns (FastAPI/Flask)
- Observability: Prometheus metrics, Alertmanager rules, Grafana dashboard layout
- Capacity estimates, sharding strategy, and multi-window rate limiting
- Interview cheat sheet with key numbers and decision framework

All architecture diagrams written in Mermaid (renders natively on GitHub).

Part of ongoing system design interview prep series.
Here you go. To use it:

bash
git add system-design/rate-limiter-system-design.md
git commit -m "docs: add distributed rate limiter system design study guide

Adds a comprehensive system design document covering the full architecture
of a production-grade distributed rate limiting service handling 500M req/day.

Covers:
- Algorithm deep dive: Token Bucket, Sliding Window, Fixed Window, Leaky Bucket
- Redis Cluster data layer with atomic INCR and Lua script patterns
- Race condition analysis and solutions (atomic ops, sorted sets)
- Failure resilience: circuit breaker, fail-open, local in-memory fallback
- Full API contract: HTTP headers (429, Retry-After, X-RateLimit-*) and gRPC proto
- Client SDK design with middleware patterns (FastAPI/Flask)
- Observability: Prometheus metrics, Alertmanager rules, Grafana dashboard layout
- Capacity estimates, sharding strategy, and multi-window rate limiting
- Interview cheat sheet with key numbers and decision framework

All architecture diagrams written in Mermaid (renders natively on GitHub).

Part of ongoing system design interview prep series."
The commit follows the Conventional Commits spec (docs: prefix) which is standard for study/documentation repos. Each bullet maps exactly to a section in your file so it's searchable in git log later. 🎯

look at this https://github.com/simplymanas/software-design and suggest improvement , also add a gitpage so that can be viewed as a website

09:26

Architected README enhancement and GitHub Pages documentation site
Architected README enhancement and GitHub Pages documentation site
Good — I can see the full picture. The repo is very bare right now: just a use-cases/ folder, a generic one-liner description, and a plain README. Let me build everything — an improved README and a full GitHub Pages site.


Architected enhanced README and dark-themed GitHub Pages site
Architected enhanced README and dark-themed GitHub Pages site
Now let me build both files — a polished README and a full GitHub Pages site.


Improved README.md for the software-design repo
Improved README.md for the software-design repo
Now the GitHub Pages site — dark blueprint aesthetic, very sharp:


Created a file, ran a command, read a file
Created a file, ran a command, read a file
Here's everything you need. Here's exactly what to do:

📁 What to push to your repo
software-design/
├── README.md          ← replace the existing one
├── docs/
│   └── index.html     ← new folder + file
└── use-cases/
    └── rate-limiter-system-design.md  ← already done
🚀 Git commands
bash
# Copy the new files in, then:
git add README.md docs/index.html
git commit -m "docs: overhaul README and add GitHub Pages site

- Rewrote README with guide index, repo structure, roadmap, and badges
- Added docs/index.html — dark blueprint-themed GitHub Pages site
- Cards for all current and upcoming design guides
- Scroll-reveal animations, sticky nav, stats bar
- Links directly to markdown guides on GitHub"

git push origin main
⚙️ Enable GitHub Pages (one-time)
Go to your repo → Settings → Pages
Under Source → select Deploy from a branch
Branch: main | Folder: /docs
Hit Save — live at simplymanas.github.io/software-design in ~60 seconds
🔍 What I improved vs what was there
Before	After
One-liner description	Full README with guide index, roadmap, badges
No website	Dark blueprint GitHub Pages site
No structure guidance	Clear folder convention documented
No topic tags	Every guide tagged with tech stack
Generic	Branded to @simplymanas with MIT badge
Index
Code · HTML 
Readme
Document · MD 
commit message for index.html

09:33

Identified straightforward commit message requirement
Identified straightforward commit message requirement
feat: add GitHub Pages site (docs/index.html)

Introduces a static GitHub Pages website for the software-design repo.

- Dark blueprint aesthetic with grid overlay and radial glow
- Sticky nav with smooth scroll
- Hero section with live/active badge
- Stats bar: guides published, roadmap count, design dimensions
- Design guide cards with difficulty badges and tech-stack tags
- Coming-soon cards for 5 upcoming guides
- Approach section explaining the first-principles methodology
- Roadmap section listing 12 upcoming design topics
- Scroll-reveal animations via IntersectionObserver
- Fully responsive — mobile nav collapses gracefully
- Zero dependencies — pure HTML/CSS/JS, no build step

Enable via: Settings → Pages → Branch: main → Folder: /docs
Live at: https://simplymanas.github.io/software-design
bash
git add docs/index.html
git commit -m "feat: add GitHub Pages site (docs/index.html)

Introduces a static GitHub Pages website for the software-design repo.

- Dark blueprint aesthetic with grid overlay and radial glow
- Sticky nav with smooth scroll
- Hero section with live/active badge
- Stats bar: guides published, roadmap count, design dimensions
- Design guide cards with difficulty badges and tech-stack tags
- Coming-soon cards for 5 upcoming guides
- Approach section explaining the first-principles methodology
- Roadmap section listing 12 upcoming design topics
- Scroll-reveal animations via IntersectionObserver
- Fully responsive — mobile nav collapses gracefully
- Zero dependencies — pure HTML/CSS/JS, no build step

Enable via: Settings → Pages → Branch: main → Folder: /docs
Live at: https://simplymanas.github.io/software-design"
Note the feat: prefix (not docs:) since this adds a new functional deliverable — the website itself — not just documentation. 🎯

update the page with my social details and linkedin, with a about me section.

09:40

Thought for 1s
Thought for 1s
I need a few details from you to make it personal!

Also — what should the About Me section say? Drop your bio, title, or a few bullet points about yourself (e.g. "ML/AI engineer, 8 years in backend, based in Bengaluru") and I'll craft it into something sharp. Or just say "write something for me" and I'll go based on what I know about you.

Q: What's your LinkedIn URL?
A: I'll type it below

Q: Which other socials should I add? (Select all that apply)
A: GitHub (@simplymanas), Twitter / X, Instagram (@popularwallpaper), Personal website / blog

09:42

Identified missing social profile details and prepared targeted requests
Identified missing social profile details and prepared targeted requests
I need the actual links/handles! Drop them here:

LinkedIn URL — e.g. linkedin.com/in/yourhandle
Twitter/X handle — e.g. @simplymanas
Personal website — e.g. manas.dev
About Me — a few lines about yourself, or just say "write it for me" and I'll craft it from what I know about you
https://www.linkedin.com/in/manasranjandash/, https://dzone.com/authors/simplymanas, twitter: @simplymanas, for about me read from here : https://www.linkedin.com/in/manasranjandash/

09:43

Synthesized professional profile and prepared portfolio updates
Synthesized professional profile and prepared portfolio updates
Got everything I need. Now building the updated page.


Orchestrated profile integration across multiple platforms
Orchestrated profile integration across multiple platforms
Now I'll make all the targeted edits — nav, About Me section, footer with socials.


Edited 5 files, read a file
Edited 5 files, read a file
Done! Here's what was added/updated:

About Me section includes:

Identity card (left) — your name, title "Software Architect · SDE-3 @ Tesco Technology", Bengaluru location
All 5 social links with branded icons — LinkedIn, GitHub, Twitter/X, DZone, Instagram (@popularwallpaper)
Bio (right) — crafted from your LinkedIn: 19 years experience, Tesco journey, ML/AI master's, DZone author, #MindMapMondaysByManas
Skills pills — distributed systems, Python, Java, Redis, Kafka, LLMs, prompt engineering, etc.
Experience timeline — Tesco, DZone author, IGNOU Master's, DeepLearning.AI cert
Footer updated with all 5 social links
Commit message for this update:

bash
git add docs/index.html
git commit -m "feat: add About Me section and social links to GitHub Pages

- Added /about section with identity card, bio, skills, and timeline
- Social links: LinkedIn, GitHub, Twitter/X, DZone, Instagram
- Updated nav to include About anchor link
- Updated footer with all social links
- Bio sourced from LinkedIn profile (19yr experience, Tesco, ML/AI)"
Index
Code · HTML 
design and document the next use case, update all required files

09:52

Thought for 2s
Thought for 2s
Let me read all existing files first before touching anything.


Orchestrated distributed cache documentation and resource updates
Orchestrated distributed cache documentation and resource updates
Good. Now I'll build all 3 files in parallel — the deep-dive guide first, then update README and index.html.


Comprehensive distributed cache system design guide
Comprehensive distributed cache system design guide
Now update the README:


Edited 4 files
Edited 4 files
Now update index.html — flip the cache card from coming-soon to live, update stats:


Edited 2 files
Edited 2 files
Now copy the guide to outputs:


Ran a command, read a file
Ran a command, read a file
3 files, all updated. Here's what changed across each:

📄 distributed-cache-system-design.md — New Guide
16 sections covering everything:

All 4 caching strategies (Cache-Aside, Write-Through, Write-Behind, Read-Through) with code + trade-offs
All 6 Redis eviction policies explained (LRU vs LFU deep dive)
Redis vs Memcached decision matrix — when to use which
Consistent hashing + virtual nodes + Redis Cluster slot assignment
3 invalidation strategies (TTL, Kafka event-driven, version tagging)
3 stampede prevention patterns — Probabilistic Early Recomputation (the best one most miss)
Circuit breaker for cache, Redis Sentinel vs Cluster comparison
Multi-level caching (L1 in-process + L2 Redis), hot key solutions
Full gRPC API contract + Python SDK
Prometheus metrics + Alertmanager rules
Interview cheat sheet with numbers
🌐 index.html — Updated
Stats bar: 1 → 2 guides published
Cache card: flipped from coming-soon (greyed out) → live (clickable)
📋 README.md — Updated
Cache row added to the guides table
Repo structure updated with new file
Roadmap: cache item ticked off ✅
Commit message:

bash
git add use-cases/distributed-cache-system-design.md docs/index.html README.md
git commit -m "docs: add distributed cache system design guide (#2)

- 16-section deep dive: caching strategies, eviction, Redis internals
- Cache-aside, write-through, write-behind, read-through with code
- Redis vs Memcached decision matrix
- Consistent hashing, virtual nodes, Redis Cluster slots
- 3 invalidation strategies: TTL, Kafka event-driven, versioning
- Cache stampede prevention: mutex, PER, stale-while-revalidate
- Circuit breaker, multi-level L1+L2 caching, hot key solutions
- gRPC API contract, Python SDK, Prometheus alerts
- Updated index.html: cache card now live, guides count → 2
- Updated README: cache added to guide table, roadmap ticked"
Distributed cache system design
Document · MD 
Index
Code · HTML 
Readme
Document · MD 
the link is  wrong: https://github.com/simplymanas/software-design/blob/main/use-cases/distributed-cache-system-design.md the right one is https://github.com/simplymanas/software-design/blob/main/use-cases/distributed-cache.md

12:31

Thought for 0s
Thought for 0s
Quick fix!


Fix the distributed cache link
Fix the distributed cache link
Also fix it in the README:


Edited a file, read a file
Edited a file, read a file
Fixed in both files. The correct link distributed-cache.md is now in index.html and README.md.

Commit message:

bash
git add docs/index.html README.md
git commit -m "fix: correct distributed cache guide filename in links

  use-cases/distributed-cache-system-design.md
→ use-cases/distributed-cache.md"
Index
Code · HTML 
Readme
Document · MD 





Claude is AI and can make mistakes. Please double-check responses.
Readme · MD
Copy

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
 
