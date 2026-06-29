# 🌟 System Design Patterns


```text
                 🏗️ SYSTEM DESIGN PATTERNS
                         │
 ┌───────────────┬───────────────┬──────────────┬────────────────┐
 │               │               │              │
🏢 Architecture 💬 Communication 🚀 Scalability ❤️ Availability
```

---

# 🎯 Imagine You Are Building a Giant LEGO City

Every building is a **service**.
Roads help them talk.
Traffic lights keep everything safe.
More buildings can be added when lots of people visit.

That is exactly what **System Design** is.

---

# 🏢 1. Architectural Patterns

```text
Architecture
│
├── 🧩 Layered
├── 🚀 Microservices
└── ⚡ Event Driven
```

## 🏢 Layered Architecture

```text
👦 User
   │
🎨 UI
   │
🧠 Business Logic
   │
💾 Database
```

### 🍕 Example
Ordering pizza:
- You talk to cashier.
- Cashier tells chef.
- Chef cooks.
- Store keeps record.

Each layer has one job.

---

## 🚀 Microservices

```text
📦 Shop
├── 👤 User Service
├── 💳 Payment Service
├── 📦 Order Service
└── 🚚 Delivery Service
```

Imagine superheroes.
Each hero has only ONE power.

---

## ⚡ Event Driven

```text
🎂 Birthday
      │
      ▼
📧 Email
🎁 Gift
🎵 Music
📱 Notification
```

One event starts many actions.

---

# 💬 2. Communication Patterns

```text
Service A
   │
   ├── 📞 Request / Response
   ├── ⚡ gRPC
   └── 📢 Pub/Sub
```

### 📞 Request-Response

Like asking:

👦 "Can I have water?"

👩 "Yes."

---

### 📢 Pub/Sub

```text
📢 School Bell
     │
──────────────
👦 Class A
👧 Class B
👨 Teachers
```

Everyone hears the bell.

---

### ⚡ gRPC

Like two robots speaking a super-fast robot language.

🤖➡️🤖

---

# 🚀 3. Scalability Patterns

```text
🚀 Scalability
│
├── ⚖️ Load Balancer
├── ⚡ Cache
└── 🧩 Sharding
```

## ⚖️ Load Balancer

```text
       😀😀😀😀😀
          │
    ⚖️ Load Balancer
     ┌──┴───┐
   🖥️      🖥️
```

Like one teacher sending students to different classrooms.

---

## ⚡ Cache

```text
📚 Library
   │
Frequently Read Books
       ↓
📖 Small Shelf
```

Keep popular things nearby.

---

## 🧩 Sharding

Instead of one giant cupboard:

```text
Cupboard A
Cupboard B
Cupboard C
```

Split data into smaller cupboards.

---

# ❤️ 4. Availability Patterns

```text
❤️ Availability
│
├── 🛡 Circuit Breaker
├── 🚢 Bulkhead
└── 🔁 Redundancy
```

## 🛡 Circuit Breaker

```text
Service ❌
    │
Breaker Opens
    │
No More Requests
```

Like a home fuse.

---

## 🚢 Bulkhead

A ship has many rooms.

If one room fills with water,
the whole ship does not sink.

---

## 🔁 Redundancy

```text
💾 Server 1
💾 Server 2
💾 Server 3
```

If one sleeps, another works.

---

# 🎮 Mini Quiz

1. Which pattern sends messages to many listeners?
- ☐ Cache
- ☑ Pub/Sub

2. Which pattern stores popular data?
- ☑ Cache

3. Which pattern divides traffic?
- ☑ Load Balancer

4. Which pattern protects from failing services?
- ☑ Circuit Breaker

---

# 🎨 Memory Map

```text
🏗️ System Design

🏢 Build → Architecture
💬 Talk → Communication
🚀 Grow → Scalability
❤️ Never Stop → Availability
```

## ⭐ Golden Rule

> Build it so it is **easy to understand**, **easy to grow**, **fast**, and **always available**.
