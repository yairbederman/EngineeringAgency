---
name: api-design
description: Design REST and GraphQL APIs with proper versioning, observability, and architecture patterns. Use when designing new APIs or reviewing API specifications.
---

# API Design

Guidelines for designing robust, scalable, and maintainable APIs.

## When to Use

- Designing new REST or GraphQL APIs
- Reviewing API specifications
- Adding endpoints to existing APIs
- Planning API versioning strategy

## REST API Design

### URL Structure

```
https://api.example.com/v1/{resource}/{id}/{sub-resource}
```

**Conventions:**
- Use plural nouns: `/users`, `/orders`, `/products`
- Use kebab-case: `/user-profiles`, `/order-items`
- Hierarchy reflects relationships: `/users/{id}/orders`
- Maximum 3 levels deep

### HTTP Methods

| Method | Usage | Idempotent | Request Body |
|--------|-------|------------|--------------|
| GET | Retrieve resource(s) | Yes | No |
| POST | Create resource | No | Yes |
| PUT | Replace resource | Yes | Yes |
| PATCH | Partial update | No | Yes |
| DELETE | Remove resource | Yes | No |

### Response Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource state conflict |
| 422 | Unprocessable | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Server Error | Unexpected error |

### Request/Response Format

```json
// Successful Response
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": { ... }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "abc-123"
  }
}

// Error Response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      { "field": "email", "message": "Must be valid email" }
    ]
  },
  "meta": {
    "requestId": "abc-123"
  }
}
```

### Pagination

```json
// Request
GET /users?page=2&limit=20

// Response
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  }
}
```

### Filtering & Sorting

```
GET /users?filter[status]=active&filter[role]=admin
GET /users?sort=-createdAt,name
GET /users?include=orders,profile
```

## Versioning Strategy

### URI Versioning (Recommended)
```
/v1/users
/v2/users
```

### Header Versioning
```
Accept: application/vnd.api.v1+json
```

### Breaking vs Non-Breaking Changes

**Breaking (requires new version):**
- Removing endpoints or fields
- Changing field types
- Changing required fields

**Non-Breaking (same version):**
- Adding optional fields
- Adding new endpoints
- Adding new optional parameters

## Observability

### Required Headers

```
X-Request-ID: <uuid>          # Trace correlation
X-Response-Time: <ms>         # Processing time
X-RateLimit-Limit: <n>        # Rate limit cap
X-RateLimit-Remaining: <n>    # Remaining requests
```

### Logging Standards

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "requestId": "abc-123",
  "method": "POST",
  "path": "/v1/users",
  "statusCode": 201,
  "responseTime": 45,
  "userId": "user-456"
}
```

## Security Checklist

- [ ] Authentication on all endpoints (except public)
- [ ] Authorization checks for resources
- [ ] Input validation and sanitization
- [ ] Rate limiting configured
- [ ] CORS properly configured
- [ ] Sensitive data not logged
- [ ] HTTPS enforced

## GraphQL Considerations

```graphql
# Use clear naming
type Query {
  user(id: ID!): User
  users(filter: UserFilter, pagination: Pagination): UserConnection!
}

# Pagination with connections
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

# Always handle errors
type MutationResult {
  success: Boolean!
  errors: [Error!]
  user: User
}
```

## API Design Checklist

Before finalizing API design:

- [ ] Consistent URL structure
- [ ] Proper HTTP methods used
- [ ] All response codes documented
- [ ] Pagination implemented
- [ ] Versioning strategy defined
- [ ] Error format standardized
- [ ] Authentication specified
- [ ] Rate limits defined
- [ ] OpenAPI/Swagger documented
