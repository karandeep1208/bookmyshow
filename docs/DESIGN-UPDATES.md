# Post-Roast Updates

## Update 1: Per-User Seat Hold Limit

**Triggered by:**
Panel Question 3 - Seat holding abuse

**What changed:**
- Added Redis counter: holds:userId
- Limit set to 8 seats per user
- Checked before acquiring lock

**Why this is necessary:**
Prevents a single user from blocking hundreds of seats.

**What it costs:**
+1 Redis read per request (~0.1ms latency)

**What it doesn't solve:**
Does not prevent abuse via multiple accounts


## Update 2: SQS Circuit Breaker

**Triggered by:**
Panel Question 4 - SQS outage

**What changed:**
- If SQS publish fails for 60 seconds → fallback to synchronous payment
- Added booking status polling endpoint

**Why this is necessary:**
Prevents system from appearing stuck during queue failures.

**What it costs:**
Higher latency during fallback mode

**What it doesn't solve:**
Reduced throughput during outages


## Update 3: Reduced Seat Lock TTL

**Triggered by:**
Panel Question 3 - Seat blocking

**What changed:**
- Reduced lock TTL from 10 min → 3 min during peak events

**Why this is necessary:**
Releases unused seats faster during high demand

**What it costs:**
Users have less time to complete booking

**What it doesn't solve:**
Does not prevent intentional abuse completely
