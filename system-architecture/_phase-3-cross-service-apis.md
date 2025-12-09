# Phase 3: Cross-Service APIs

> **Goal**: Document API contracts between services—what one service calls from another.

---

## Input

- `${ANALYSIS_DIR}/service-topology.json` (from Phase 2)
- Each project's `api-contracts.json`

## Output

- `${ANALYSIS_DIR}/cross-service-apis.json`

---

## Instructions

### Step 1: Identify Cross-Service Calls

For each dependency edge from Phase 2:

```
wg-client → wg-data-api
```

Find the specific endpoints called.

### Step 2: Extract API Contracts

**From Client Side (caller)**:
- Search for API client code (fetch, axios, SDK)
- Match URL patterns to backend endpoints
- Extract request parameters and expected response types

**From Server Side (callee)**:
- Read `api-contracts.json`
- Match endpoints to client calls
- Include validation rules

### Step 3: Map DTOs Across Boundaries

For each cross-service call:

1. **Request DTO**: What the client sends
2. **Response DTO**: What the server returns
3. **Source Definition**: Where the type is defined (which project, which file)

Example:
```json
{
  "caller": "wg-client",
  "callee": "wg-data-api",
  "endpoint": {
    "method": "POST",
    "path": "/site/getEngineData",
    "fullPath": "/api/data-api/site/getEngineData"
  },
  "request": {
    "type": "EngineDataRequest",
    "definedIn": "wg-data-api",
    "file": "src/main/java/com/lognet/wg/api/rest/dto/EngineDataRequest.java",
    "fields": {
      "siteId": { "type": "String", "required": true },
      "locale": { "type": "String", "required": false },
      "packageSubType": { "type": "String", "required": false }
    }
  },
  "response": {
    "type": "EngineDataResponse",
    "definedIn": "wg-data-api",
    "file": "src/main/java/com/lognet/wg/api/rest/dto/EngineDataResponse.java"
  }
}
```

### Step 4: Handle Proxied Calls

Some services proxy to others. Document the full chain:

```json
{
  "chain": ["wg-client", "wg-data-api", "wg-cms-api"],
  "originEndpoint": "/promotions/list",
  "intermediates": [
    {
      "service": "wg-data-api",
      "endpoint": "/data/getComponentData",
      "transforms": "Aggregates and compresses response"
    }
  ],
  "finalEndpoint": {
    "service": "wg-cms-api",
    "endpoint": "/promotions/data"
  }
}
```

---

## Output Schema

```json
{
  "generatedAt": "ISO timestamp",
  "crossServiceCalls": [
    {
      "id": "client-to-data-001",
      "caller": "wg-client",
      "callee": "wg-data-api",
      "endpoint": {
        "method": "POST",
        "path": "/site/getEngineData"
      },
      "request": {
        "type": "EngineDataRequest",
        "definedIn": "wg-data-api",
        "fields": {}
      },
      "response": {
        "type": "EngineDataResponse",
        "definedIn": "wg-data-api"
      },
      "usedBy": ["SearchWidget", "EngineDataProvider"]
    }
  ],
  "sharedTypes": [
    {
      "name": "BookingData",
      "canonicalDefinition": {
        "project": "wg-ordermanager-api",
        "file": "src/main/java/.../BookingData.java"
      },
      "usedIn": ["wg-client", "wg-payment-api", "wg-tripdetails-api"]
    }
  ],
  "_coverage": {
    "dependenciesAnalyzed": 12,
    "endpointsDocumented": 45,
    "sharedTypesFound": 8
  }
}
```

---

## Validation

| Check | Requirement |
|-------|-------------|
| All dependencies covered | Each edge from Phase 2 has ≥1 documented call |
| Types resolved | No `unknown` request/response types |
| Sources linked | Every type has `definedIn` and `file` |

---

## Common Patterns

### Frontend API Client Pattern
```typescript
// wg-client/src/sdk/api/dataApi.ts
export const getEngineData = (request: EngineDataRequest) =>
  api.post<EngineDataResponse>('/api/data-api/site/getEngineData', request);
```

### Backend-to-Backend Pattern
```java
// In wg-data-api calling wg-cms-api
@FeignClient(name = "cms-service")
public interface CmsClient {
    @GetMapping("/promotions/{id}")
    PromotionData getPromotion(@PathVariable String id);
}
```
