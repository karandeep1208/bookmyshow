# Panel Q&A Notes

## Q1: What happens if Redis crashes?
Answer:
System returns 503 and does not proceed without lock. DB optimistic locking prevents double booking.

Gap:
Could improve with fallback locking.


## Q2: Replica lag issue?
Answer:
Replica is only for approximate reads. Booking always checks primary DB.

Gap:
None


## Q3: Prevent user holding 200 seats?
Answer:
Added Redis counter per user with limit of 8 seats.

Gap:
Does not prevent multi-account abuse


## Q4: SQS failure?
Answer:
Polling endpoint shows pending status. Circuit breaker fallback to sync processing.

Gap:
Need better alerting explanation


## Q5: Cost spike?
Answer:
Peak cost justified by revenue. Optimizations via caching and spot instances.

Gap:
Could add cost monitoring tools
