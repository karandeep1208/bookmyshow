# Async Order Processing

## Why Async?

Payment APIs take 200–2000ms.

If synchronous:
- DB connections blocked
- System crashes under load

---

## Message Format

{
  "bookingId": "uuid",
  "userId": "uuid",
  "totalAmount": 5000,
  "paymentToken": "token",
  "seatIds": [1,2,3],
  "eventId": 101,
  "idempotencyKey": "unique-key"
}

---

## Worker Logic

### Success Flow

1. Read message from SQS
2. Call payment gateway
3. Update booking → confirmed
4. Update seats → booked
5. Send notification
6. Delete message

---

### Failure Flow

1. Update booking → failed
2. Release seats → available
3. Notify user
4. Delete message

---

## Edge Cases

### API crashes after sending SQS

- Booking exists in DB
- Worker still processes it
- User may retry or check status

---

### Payment timeout

- Retry up to 3 times
- If still failing → move to DLQ

---

## SQS Configuration

Visibility Timeout:
60 seconds (2× max payment time)

Retries:
3 attempts

DLQ:
Enabled for failed messages
