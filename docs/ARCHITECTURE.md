# BookMyShow - Full System Architecture

Mobile / Browser
      │
      ▼
CloudFront CDN
  - Serves: static assets (HTML, JS, CSS), cached event pages (TTL 60s)
  - Cache hit → direct response
  - Cache miss → forward to ALB
      │ (API requests only: /api/*)
      ▼
Application Load Balancer (ALB)
  - SSL termination
  - Health checks: every 10s
  - Rate limit: 200 req/IP/min
      │
      ├──────── Node.js API ×1
      ├──────── Node.js API ×2     ← Auto Scaling Group (min: 4, max: 20)
      ├──────── Node.js API ×3        Scale-out: CPU > 70% for 2 min
      └──────── Node.js API ×N
                │
                ├── READ ──► Redis Cluster (3 nodes)
                │             - Cache: availability:eventId (TTL 30s)
                │             - Cache: event:eventId (TTL 3600s)
                │             - Lock: seat_lock:seatId (SETNX, TTL 180s)
                │             - Counter: holds:userId (limit 8 seats) ← UPDATE
                │
                ├── WRITE ──► PostgreSQL Primary
                │             - Seat booking (transaction)
                │             - Optimistic lock (version column)
                │             └── Replication ──► Read Replica 1
                │                                   Read Replica 2
                │
                └── PUBLISH ──► SQS Payment Queue
                                 - Message: {bookingId, userId, seats}
                                 - Visibility timeout: 20s
                                 - Max retries: 3
                                 - DLQ: payment-dlq
                                      │
                               ┌──────▼──────────────────────┐
                               │ Payment Worker (ECS) ×10     │
                               │ 1. Read from SQS             │
                               │ 2. Call Payment Gateway      │
                               │ 3. Update DB (confirm)       │
                               │ 4. Publish SNS               │
                               │ 5. Delete SQS message        │
                               └──────────────────────────────┘
                                      │
                               ┌──────▼──────────┐
                               │      SNS        │
                               ├──► SES Email    │
                               └──► SMS Gateway  │
                                      │
                                      ▼
                                User Notification
