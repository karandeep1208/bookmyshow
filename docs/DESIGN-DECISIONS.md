# Design Decisions

## Decision: Redis SETNX for Seat Locking

**Context:**
High concurrency booking system with lakhs of users attempting to book simultaneously.

**Options considered:**
1. PostgreSQL FOR UPDATE
   - Strong consistency
   - But high latency and DB connection exhaustion risk
2. Redis SETNX (chosen)
   - Fast (sub-millisecond)
   - Scales independently of DB

**Why chosen:**
Redis provides extremely low latency locking and avoids DB bottlenecks during peak traffic.

**Tradeoffs accepted:**
- Redis failure affects locking
- Requires fallback mechanisms

**Revision trigger:**
If Redis becomes a bottleneck → consider Redlock or DB fallback


## Decision: Cache Strategy (TTL-based)

**Context:**
Frequent reads for event availability and details.

**Options considered:**
1. Event-driven invalidation
   - Complex to implement
2. TTL-based cache (chosen)
   - Simple and predictable

**Why chosen:**
TTL-based caching reduces complexity and handles high read traffic efficiently.

**Tradeoffs accepted:**
- Slightly stale data possible

**Revision trigger:**
If stale data impacts UX significantly → move to event-based invalidation


## Decision: UUID for Booking IDs

**Context:**
Distributed system with multiple services generating booking IDs.

**Options considered:**
1. SERIAL (auto increment)
   - Simple but not scalable
2. UUID (chosen)
   - Globally unique

**Why chosen:**
UUID avoids collisions and supports distributed architecture.

**Tradeoffs accepted:**
- Larger storage size
- Less human readable

**Revision trigger:**
If indexing performance degrades → consider hybrid approach


## Decision: SQS Visibility Timeout = 20 seconds

**Context:**
Payment processing latency varies (external API calls).

**Options considered:**
1. Short timeout (5s)
   - Risk of duplicate processing
2. Long timeout (60s)
   - Slower retries
3. 20 seconds (chosen)

**Why chosen:**
Balances retry speed and processing time for payments.

**Tradeoffs accepted:**
- Some delay in retries

**Revision trigger:**
If payment latency increases → adjust timeout
