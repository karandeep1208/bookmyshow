# Cache Design

## Strategy: Cache Aside (Lazy Loading)

---

## 1. Event Details

Key:
event:{event_id}

TTL:
3600 seconds (1 hour)

Invalidate:
On event update

---

## 2. Seat Availability Count

Key:
availability:{event_id}:{category}

TTL:
30 seconds

Invalidate:
On any seat status change

---

## 3. Seat Map Layout

Key:
seatmap:{event_id}

TTL:
86400 seconds (24 hours)

Invalidate:
Only on event cancellation

---

## What NOT to Cache

❌ Individual seat status

Reason:
Can cause double-booking due to stale data

---

## Invalidation Flow

1. Update DB
2. Delete Redis key
3. Next request repopulates cache
