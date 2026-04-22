# Pattern: Webhook Integration

**Pattern Type:** Event-Driven Push Notification  
**Common Use Cases:**
- Real-time event notifications
- Asynchronous system updates
- Triggering workflows based on external events
- Reducing polling overhead
- Decoupled system communication

---

## Overview

This pattern describes integration using webhooks, where a source system sends HTTP POST requests to a target system's endpoint when specific events occur. Webhooks enable real-time, event-driven communication without the need for continuous polling.

## Architecture Pattern

```mermaid
sequenceDiagram
    participant Source as Source System
    participant Webhook as Webhook Endpoint
    participant Target as Target System
    participant Queue as Message Queue<br/>(Optional)

    Source->>Source: Event occurs
    Source->>Webhook: POST /webhook (event payload)
    Webhook->>Webhook: Validate signature
    Webhook-->>Source: 200 OK (acknowledge)
    Webhook->>Queue: Enqueue event
    Queue->>Target: Process event async
    Target->>Target: Execute business logic
```

## Webhook Flow

### 1. Event Registration
- Target system provides webhook endpoint URL
- Target system configures which events to receive
- Source system stores webhook configuration
- Source system may provide webhook secret for signature validation

### 2. Event Trigger
1. Event occurs in source system (e.g., record created, updated, deleted)
2. Source system generates event payload (JSON)
3. Source system signs payload with secret (HMAC)
4. Source system sends HTTP POST to webhook endpoint
5. Target system validates signature
6. Target system returns 200 OK immediately (< 5 seconds)
7. Target system processes event asynchronously

### 3. Failure Handling
- If webhook endpoint unavailable or returns error, source retries
- Retry with exponential backoff
- After max retries, event logged/alerted for manual intervention

## Webhook Request Structure

### Headers

```http
POST /webhooks/events HTTP/1.1
Host: target-system.com
Content-Type: application/json
User-Agent: SourceSystem/1.0
X-Webhook-Signature: sha256=5d41402abc4b2a76b9719d911017c592
X-Webhook-ID: evt_1a2b3c4d5e6f
X-Webhook-Timestamp: 1619712000
X-Webhook-Retry: 0
```

### Payload

```json
{
  "id": "evt_1a2b3c4d5e6f",
  "type": "customer.created",
  "timestamp": "2026-04-22T10:30:00Z",
  "source": "source-system",
  "data": {
    "object": "customer",
    "id": "cust-12345",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "createdAt": "2026-04-22T10:30:00Z"
  },
  "metadata": {
    "userId": "user-789",
    "ipAddress": "192.168.1.100"
  }
}
```

## Webhook Response

### Success Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "received": true,
  "eventId": "evt_1a2b3c4d5e6f",
  "timestamp": "2026-04-22T10:30:01Z"
}
```

### Error Response (Will Trigger Retry)

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "Processing error",
  "eventId": "evt_1a2b3c4d5e6f"
}
```

## Security: Signature Validation

### HMAC-SHA256 Signature (Recommended)

**Source System (Signing):**
```python
import hmac
import hashlib

secret = "your_webhook_secret"
payload = json.dumps(event_data)
signature = hmac.new(
    secret.encode(),
    payload.encode(),
    hashlib.sha256
).hexdigest()

headers["X-Webhook-Signature"] = f"sha256={signature}"
```

**Target System (Validation):**
```python
import hmac
import hashlib

received_signature = request.headers.get("X-Webhook-Signature")
expected_signature = "sha256=" + hmac.new(
    secret.encode(),
    request.body,
    hashlib.sha256
).hexdigest()

if not hmac.compare_digest(received_signature, expected_signature):
    return 401  # Unauthorized
```

### Alternative: Shared API Key

```http
POST /webhooks/events HTTP/1.1
Authorization: Bearer YOUR_WEBHOOK_API_KEY
```

## Event Types

### Standard Event Naming Convention

```
{resource}.{action}

Examples:
- customer.created
- customer.updated
- customer.deleted
- order.completed
- payment.succeeded
- payment.failed
- user.registered
- subscription.expired
```

### Event Payload Structure

```json
{
  "id": "unique-event-id",
  "type": "resource.action",
  "timestamp": "ISO-8601 timestamp",
  "source": "system-name",
  "data": {
    // Event-specific data
  },
  "metadata": {
    // Optional context
  }
}
```

## Retry Strategy

| Attempt | Delay | Total Time |
|---------|-------|------------|
| 1 (initial) | 0s | 0s |
| 2 | 1 minute | 1 min |
| 3 | 5 minutes | 6 min |
| 4 | 15 minutes | 21 min |
| 5 | 1 hour | 1h 21min |
| 6 | 4 hours | 5h 21min |
| 7 | 12 hours | 17h 21min |

**Max Retries:** 7 attempts over ~17 hours  
**After Max Retries:** Move to dead-letter queue, alert monitoring

### Retry Response Codes

| Status Code | Retry? | Reason |
|-------------|--------|--------|
| 200-299 | ❌ No | Success |
| 400, 401, 403, 404, 422 | ❌ No | Client error - won't succeed on retry |
| 408, 429 | ✅ Yes | Timeout or rate limit - transient |
| 500, 502, 503, 504 | ✅ Yes | Server error - transient |
| Network error | ✅ Yes | Connection issue - transient |

## Idempotency

Webhooks may be delivered multiple times (retries, duplicates). Target system must handle duplicates gracefully.

### Idempotency Key

```json
{
  "id": "evt_1a2b3c4d5e6f",  // Use as idempotency key
  "type": "customer.created",
  "data": {...}
}
```

### Idempotency Check

```python
def handle_webhook(event):
    event_id = event["id"]
    
    # Check if already processed
    if redis.exists(f"webhook:{event_id}"):
        return {"status": "already_processed"}
    
    # Process event
    process_event(event)
    
    # Mark as processed (TTL: 7 days)
    redis.setex(f"webhook:{event_id}", 604800, "processed")
    
    return {"status": "processed"}
```

## Implementation Checklist

### Source System (Webhook Sender)
- [ ] Define event types to emit
- [ ] Generate webhook signatures (HMAC-SHA256)
- [ ] Implement retry logic with exponential backoff
- [ ] Log all webhook deliveries (success/failure)
- [ ] Provide webhook configuration UI/API
- [ ] Support webhook testing endpoint
- [ ] Monitor webhook delivery success rate
- [ ] Implement circuit breaker for failing endpoints

### Target System (Webhook Receiver)
- [ ] Create webhook endpoint (POST /webhooks/*)
- [ ] Validate webhook signatures
- [ ] Return 200 OK within 5 seconds
- [ ] Process events asynchronously (queue)
- [ ] Implement idempotency checking
- [ ] Log all received webhooks
- [ ] Monitor webhook processing failures
- [ ] Implement dead-letter queue for failures
- [ ] Set up alerts for webhook errors
- [ ] Test with various event types

## Performance Considerations

### Source System
- **Timeout:** Set reasonable timeout (5-10 seconds)
- **Async Sending:** Don't block main thread waiting for webhook response
- **Batch:** Consider batching events if high volume (with target consent)
- **Circuit Breaker:** Temporarily stop sending to failing endpoints

### Target System
- **Fast Response:** Return 200 OK within 5 seconds (before processing)
- **Async Processing:** Use message queue for actual processing
- **Rate Limiting:** Protect against webhook floods
- **Horizontal Scaling:** Scale webhook receivers independently

## Monitoring & Observability

### Source System Metrics
- Webhook delivery success rate (%)
- Webhook delivery latency (ms)
- Retry rate (%)
- Failed webhooks after max retries
- Circuit breaker trip frequency

### Target System Metrics
- Webhook received count
- Webhook processing time
- Invalid signature count
- Duplicate webhook count
- Processing failure rate

### Logging

**Source System:**
```json
{
  "timestamp": "2026-04-22T10:30:00Z",
  "webhookId": "evt_1a2b3c4d5e6f",
  "eventType": "customer.created",
  "targetUrl": "https://target.com/webhooks",
  "attempt": 1,
  "statusCode": 200,
  "duration": 145,
  "success": true
}
```

**Target System:**
```json
{
  "timestamp": "2026-04-22T10:30:01Z",
  "webhookId": "evt_1a2b3c4d5e6f",
  "eventType": "customer.created",
  "signatureValid": true,
  "processed": true,
  "duration": 2350
}
```

## Testing Webhooks

### Development Testing
- Use tools like ngrok or localtunnel to expose local endpoints
- Webhook testing services: webhook.site, requestbin.com
- Mock webhook sender for testing receiver logic

### Webhook Replay
Source system should provide webhook replay functionality:
- Replay specific event
- Replay events within time range
- Replay all failed events

## Security Best Practices

✅ **Always validate signatures** (prevent unauthorized webhooks)  
✅ **Use HTTPS only** (encrypt data in transit)  
✅ **Whitelist source IPs** (if source provides static IPs)  
✅ **Rate limit webhook endpoint** (prevent DoS)  
✅ **Implement timeouts** (don't wait forever for processing)  
✅ **Log all webhooks** (audit trail)  
✅ **Rotate webhook secrets** periodically  
✅ **Validate event types** (only accept expected events)  
✅ **Sanitize input** (prevent injection attacks)

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Webhook endpoint down | Retry with exponential backoff, alert after max retries |
| Duplicate webhooks | Implement idempotency checking with event ID |
| Slow processing | Return 200 immediately, process async via queue |
| Failed signature validation | Check secret, timestamp drift, payload encoding |
| Webhook floods | Implement rate limiting, validate source |
| Lost webhooks | Provide webhook replay functionality |
| Webhook ordering | Don't rely on order, use timestamps if needed |

## Example Use Cases

- **Payment Processing**: Payment completed, payment failed notifications
- **Order Management**: Order placed, order shipped, order delivered
- **User Management**: User registered, user verified, user deleted
- **Content Publishing**: Content published, content updated, content expired
- **Inventory Updates**: Stock level changed, product added, product removed
- **Notification Systems**: Trigger email, SMS, push notifications
- **CI/CD Pipelines**: Build completed, deployment succeeded
- **Monitoring/Alerting**: Threshold exceeded, service down, recovery

## Related Patterns

- [REST API Integration](rest-api-integration.md) - Request-response alternative
- [Message Queue Integration](message-queue-integration.md) - More reliable async messaging
- [Event Streaming](event-streaming.md) - Higher throughput event processing
