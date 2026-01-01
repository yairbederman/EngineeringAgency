---
description: Phase 4.2 - Extract inter-service communication contracts for microservice calls
---

# Phase 4.2: Extract Inter-Service Contracts

## Goal
Extract contracts for internal service-to-service communication—enabling Tech Specs to know exact payloads when calling other microservices.

## Trigger Condition
**Execute this phase IF**:
- `source-structure.json.detectedStack.type` includes "Backend"
- HTTP client usage detected (RestTemplate, WebClient, FeignClient, axios, fetch in Node backend)
- Cross-service environment variables detected (`*_SERVICE_URL`, `*_API_URL`)

**Skip IF**: Monolith with no external service calls.

## Input
- `source-structure.json.discoveredLocations.clients` (HTTP clients)
- `${GLOBAL_WORKFLOWS_ROOT}/shared/configuration.md` (Registered Projects)

## Relationship to Phase 4
Phase 4's `crossProjectDependencies` identifies WHICH services are called. This phase extracts the **exact contracts** for those calls.

## Steps

### 1. Identify Inter-Service Clients
Scan for HTTP client patterns used for internal calls:

| Framework | Detection Patterns |
|-----------|-------------------|
| Spring RestTemplate | `RestTemplate`, `restTemplate.exchange()` |
| Spring WebClient | `WebClient`, `webClient.get()`, `webClient.post()` |
| Spring Feign | `@FeignClient`, interface proxies |
| Node Axios | `axios.create()` with internal service base URL |
| Node Fetch | `fetch()` with `${*_SERVICE_URL}` pattern |
| Python Requests | `requests.get()`, `requests.post()` to internal URLs |

### 2. Match to Registered Projects
For EACH inter-service client:

1. Extract base URL (from config, env var, or hardcoded)
2. Match URL to registered project in `shared/configuration.md`
3. Document the mapping with code evidence

### 3. Extract Call Contracts
For EACH method/call to another service:

| Field | Source |
|-------|--------|
| `method` | HTTP method (GET, POST, PUT, DELETE) |
| `path` | URL path being called |
| `requestDto` | Request body DTO (resolve type fully) |
| `responseDto` | Response body DTO (resolve type fully) |
| `headers` | Custom headers being sent |
| `queryParams` | Query parameters |
| `timeoutMs` | Configured timeout |
| `retryConfig` | Retry settings if present |

### 4. Verify Against Target's api-contracts
If target service has been mapped by `/map-codebase-agent`:

1. Load target's `.ai-instructions/analysis/api-contracts.json`
2. Find matching endpoint
3. Verify request/response DTOs are compatible
4. Flag mismatches in `_contractMismatches`

### 5. Extract Resilience Patterns
Document circuit breaker, retry, and fallback configurations:

| Pattern | Detection |
|---------|-----------|
| Circuit Breaker | `@CircuitBreaker`, Resilience4j, Hystrix |
| Retry | `@Retry`, retry config, exponential backoff |
| Fallback | `fallback=`, fallback methods |
| Timeout | `timeout=`, `Duration.ofSeconds()` |

## Output

### `analysis/inter-service-contracts.json`
```json
{
  "outboundCalls": {
    "[TargetServiceName]": {
      "registeredProject": "[project-name from shared/configuration]",
      "clientFile": "[path to client class/module]",
      "baseUrl": "${SERVICE_URL}",
      "baseUrlEvidence": "[file:line where URL is defined]",
      "calls": [
        {
          "clientMethod": "[method name in this service]",
          "httpMethod": "POST",
          "path": "/api/v1/orders",
          "requestDto": {
            "name": "[DtoName]",
            "fields": {
              "[field]": "[type]"
            }
          },
          "responseDto": {
            "name": "[DtoName]",
            "fields": {
              "[field]": "[type]"
            }
          },
          "queryParams": {
            "[param]": "[type]"
          },
          "headers": {
            "[header]": "[source]"
          },
          "codeEvidence": "[file:line]",
          "targetContractVerified": true,
          "targetContractRef": "[target-project]/.ai-instructions/api-contracts.json#[endpoint]"
        }
      ],
      "resilience": {
        "circuitBreaker": {
          "enabled": true,
          "failureThreshold": 5,
          "waitDuration": "10s"
        },
        "retry": {
          "maxAttempts": 3,
          "backoff": "exponential"
        },
        "timeout": "5s"
      }
    }
  },
  "_contractMismatches": [
    {
      "call": "[TargetService]/[endpoint]",
      "issue": "Request field 'orderId' is string here but number in target",
      "localFile": "[file:line]",
      "targetContract": "[target api-contracts ref]"
    }
  ],
  "_unverifiedCalls": [
    {
      "target": "[ServiceName]",
      "reason": "Target project not mapped by /map-codebase-agent"
    }
  ],
  "_coverage": {
    "targetServicesIdentified": 5,
    "callsExtracted": 23,
    "contractsVerified": 18,
    "contractMismatches": 2
  }
}
```

## Cross-Project Verification

> [!IMPORTANT]
> For each target service, attempt to load its `.ai-instructions/api-contracts.json` if the project is in the same workspace.

**Verification Steps**:
1. Check if target project path exists
2. Load `api-contracts.json`
3. Find endpoint matching the call
4. Compare request/response DTOs
5. Document result in `targetContractVerified`

## Critical Rules

1. **Full DTOs**: Extract complete request/response types, not just names
2. **URL Evidence**: Always document where base URL is configured
3. **Cross-Verification**: Verify against target's contracts when available
4. **Resilience Docs**: Document circuit breaker/retry configs for reliability
5. **Mismatch Flagging**: Flag DTO mismatches between caller and callee
