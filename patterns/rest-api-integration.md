# Pattern: REST API Integration

**Pattern Type:** API-Based Request-Response  
**Common Use Cases:**
- Real-time data retrieval and updates
- Microservices communication
- Third-party API consumption
- Mobile/web application backends
- System-to-system integrations

---

## Overview

This pattern describes integration via RESTful HTTP APIs, where one system (client) makes HTTP requests to another system (server) to create, read, update, or delete resources. REST APIs use standard HTTP methods and typically return JSON or XML responses.

## Architecture Pattern

```mermaid
flowchart LR
    A[Client System] -->|HTTP Request| B[API Gateway<br/>Optional]
    B -->|Forward Request| C[REST API<br/>Target System]
    C -->|HTTP Response| B
    B -->|Return Response| A
    
    style A fill:#e1f5ff
    style C fill:#e1f5ff
    style B fill:#fff4e1
```

## HTTP Methods & Operations

| HTTP Method | Operation | Idempotent | Use Case |
|-------------|-----------|------------|----------|
| **GET** | Retrieve resource(s) | ✅ Yes | Fetch data, list resources |
| **POST** | Create new resource | ❌ No | Create new records |
| **PUT** | Replace entire resource | ✅ Yes | Full update of existing record |
| **PATCH** | Partial update | ❌ No* | Update specific fields |
| **DELETE** | Remove resource | ✅ Yes | Delete records |

*PATCH can be designed to be idempotent

## Common API Endpoints

| Endpoint Pattern | Method | Purpose | Example |
|------------------|--------|---------|---------|
| `/api/{resource}` | GET | List all resources | `GET /api/customers` |
| `/api/{resource}/{id}` | GET | Get single resource | `GET /api/customers/12345` |
| `/api/{resource}` | POST | Create new resource | `POST /api/customers` |
| `/api/{resource}/{id}` | PUT | Replace resource | `PUT /api/customers/12345` |
| `/api/{resource}/{id}` | PATCH | Update fields | `PATCH /api/customers/12345` |
| `/api/{resource}/{id}` | DELETE | Delete resource | `DELETE /api/customers/12345` |
| `/api/{resource}/search?q={query}` | GET | Search resources | `GET /api/customers/search?q=john` |

## Request Structure

### Headers

```http
POST /api/customers HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
User-Agent: MyApp/1.0
X-Request-ID: req-12345
```

### Body (POST/PUT/PATCH)

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "phone": "+1-555-0123",
  "address": {
    "street": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "zipCode": "94105"
  }
}
```

## Response Structure

### Success Response (200 OK, 201 Created)

```json
{
  "id": "cust-12345",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "phone": "+1-555-0123",
  "address": {
    "street": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "zipCode": "94105"
  },
  "createdAt": "2026-04-22T10:30:00Z",
  "updatedAt": "2026-04-22T10:30:00Z"
}
```

### Error Response (400 Bad Request, 404 Not Found, etc.)

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email address format",
    "details": [
      {
        "field": "email",
        "issue": "Must be a valid email address"
      }
    ],
    "requestId": "req-12345",
    "timestamp": "2026-04-22T10:30:00Z"
  }
}
```

## HTTP Status Codes

| Status Code | Meaning | Use Case |
|-------------|---------|----------|
| **200 OK** | Success | Successful GET, PUT, PATCH, DELETE |
| **201 Created** | Resource created | Successful POST with new resource |
| **204 No Content** | Success, no body | Successful DELETE or update with no response |
| **400 Bad Request** | Client error | Invalid request format, validation failure |
| **401 Unauthorized** | Auth required | Missing or invalid credentials |
| **403 Forbidden** | Access denied | Valid auth but insufficient permissions |
| **404 Not Found** | Resource missing | Resource ID doesn't exist |
| **409 Conflict** | State conflict | Duplicate, version conflict |
| **429 Too Many Requests** | Rate limited | Exceeded API rate limits |
| **500 Internal Server Error** | Server error | Unexpected server-side error |
| **502 Bad Gateway** | Gateway error | Upstream service unavailable |
| **503 Service Unavailable** | Temp unavailable | Planned maintenance, overload |

## Authentication Methods

### 1. API Key (Simple)

```http
GET /api/customers HTTP/1.1
Authorization: ApiKey YOUR_API_KEY_HERE
```

### 2. Bearer Token (OAuth 2.0, JWT)

```http
GET /api/customers HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 3. Basic Auth

```http
GET /api/customers HTTP/1.1
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

### 4. OAuth 2.0 Client Credentials Flow

```
1. POST /oauth/token with client_id and client_secret
2. Receive access_token
3. Use token in Authorization: Bearer {access_token}
4. Refresh token before expiration
```

## Error Handling Strategy

| Error Type | Status Code | Retry Strategy |
|------------|-------------|----------------|
| **Validation Error** | 400 | ❌ Do not retry - fix request |
| **Unauthorized** | 401 | ❌ Do not retry - refresh auth token |
| **Not Found** | 404 | ❌ Do not retry - resource doesn't exist |
| **Rate Limited** | 429 | ✅ Retry after `Retry-After` header |
| **Server Error** | 500 | ✅ Retry with exponential backoff |
| **Timeout** | - | ✅ Retry up to 3 times |
| **Network Error** | - | ✅ Retry with exponential backoff |

### Retry Logic

```
Attempt 1: Immediate
Attempt 2: Wait 2 seconds
Attempt 3: Wait 5 seconds
Attempt 4: Wait 10 seconds
Attempt 5: Wait 30 seconds
Max Attempts: 5
After max attempts: Log error, alert, move to dead-letter queue
```

## Pagination

### Offset-Based Pagination

```http
GET /api/customers?offset=20&limit=10
```

Response:
```json
{
  "data": [...],
  "pagination": {
    "offset": 20,
    "limit": 10,
    "total": 150
  }
}
```

### Cursor-Based Pagination

```http
GET /api/customers?cursor=eyJpZCI6MTIzfQ&limit=10
```

Response:
```json
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTMzfQ",
    "hasMore": true
  }
}
```

## Rate Limiting

### Common Strategies

| Strategy | Description | Example |
|----------|-------------|---------|
| **Fixed Window** | X requests per time window | 1000 requests/hour |
| **Sliding Window** | Rolling time window | 100 requests/minute |
| **Token Bucket** | Burst allowed, refills over time | Burst: 50, Refill: 10/sec |

### Response Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1619712000
Retry-After: 3600
```

## Implementation Checklist

- [ ] Define API base URL for each environment
- [ ] Document all endpoints and methods
- [ ] Choose authentication method
- [ ] Implement request/response logging
- [ ] Handle all standard HTTP status codes
- [ ] Implement retry logic with exponential backoff
- [ ] Respect rate limits (429 responses)
- [ ] Use timeouts (connection: 5s, read: 30s)
- [ ] Implement request deduplication (idempotency keys)
- [ ] Set up API monitoring and alerting
- [ ] Document error codes and meanings
- [ ] Implement pagination for list endpoints
- [ ] Add request ID for traceability
- [ ] Test error scenarios (network, timeout, auth)
- [ ] Load test under expected volume

## Performance Optimization

### Client-Side
- **Connection Pooling**: Reuse HTTP connections
- **Compression**: Enable gzip/brotli
- **Parallel Requests**: Use async/await or threads
- **Caching**: Cache GET responses with TTL
- **Batch Operations**: Group multiple operations when API supports it

### Server-Side Considerations
- **API Gateway**: Use for rate limiting, caching, auth
- **CDN**: Cache static/read-heavy responses at edge
- **Database Indexing**: Optimize queries for API endpoints
- **Response Compression**: Enable gzip/brotli
- **Asynchronous Processing**: Return 202 Accepted for long operations

## Monitoring & Observability

### Key Metrics
- API response time (p50, p95, p99)
- Success rate (%)
- Error rate by status code (%)
- Rate limit hit frequency
- Retry rate
- Timeout rate

### Logging
```json
{
  "timestamp": "2026-04-22T10:30:00Z",
  "requestId": "req-12345",
  "method": "POST",
  "endpoint": "/api/customers",
  "statusCode": 201,
  "duration": 145,
  "clientIp": "192.168.1.100"
}
```

## Security Best Practices

✅ **Always use HTTPS** (TLS 1.2+)  
✅ **Validate all input** (prevent injection attacks)  
✅ **Use authentication** on all non-public endpoints  
✅ **Implement rate limiting** (prevent abuse)  
✅ **Sanitize error messages** (don't leak internal details)  
✅ **Use API keys per environment** (dev, staging, prod)  
✅ **Rotate credentials regularly**  
✅ **Log security events** (failed auth, suspicious activity)  
✅ **Implement CORS** properly for web clients  
✅ **Use request signing** for sensitive operations

## Example Use Cases

- **CRM Integration**: Sync customer records to/from CRM system
- **Payment Gateway**: Process payments via payment API
- **Shipping API**: Get shipping rates, create shipping labels
- **Notification Service**: Send emails, SMS, push notifications
- **Identity Provider**: Authenticate users, retrieve user profiles
- **Search Service**: Query search index, retrieve results
- **Analytics API**: Send events, retrieve reports
- **Content Delivery**: Fetch content, images, videos

## Related Patterns

- [GraphQL Integration](graphql-integration.md) - Alternative API approach with flexible queries
- [Webhook Integration](webhook-integration.md) - Event-driven push notifications
- [Batch File Transfer](batch-file-transfer.md) - Alternative for bulk data transfer
- [Message Queue Integration](message-queue-integration.md) - Async alternative for high reliability
