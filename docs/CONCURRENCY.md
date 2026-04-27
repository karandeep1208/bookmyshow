# Concurrency Strategy

## Problem
5 lakh users may try booking same seat simultaneously. We must ensure zero double-bookings.

---

## Option A: PostgreSQL Row Locking

Uses:
SELECT ... FOR UPDATE

Problem:
DB connection pool becomes bottleneck.

Example:
80% requests → 20ms
20% requests → 800ms

Connections = RPS × time

At ~2000 RPS → 500 connections exhausted

Conclusion:
Fails at high concurrency.

---

## Option B: Redis Distributed Lock

Lock Key:
seat_lock:{seat_id}

Acquire:
SET key value NX EX 30

Release:
Lua script ensures safe unlock

Advantages:
- Very fast (sub-ms)
- Handles 100K+ requests/sec

Problems:
- Redis failure risk
- TTL tuning required

---

## Final Choice: HYBRID APPROACH

We use:
- Redis SETNX → for seat locking (high traffic phase)
- PostgreSQL → final confirmation (ACID guarantee)

---

## Why this works

- Redis handles massive concurrency
- DB ensures final correctness
- Fits within $2000 budget

---

## Limitations

- Redis crash → temporary inconsistency
- TTL expiry issues

---

## When to switch?

If traffic < 5K RPS:
→ Use PostgreSQL only

If traffic > 100K RPS:
→ Scale Redis cluster further
