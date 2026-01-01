---
description: Phase 4.4 - Extract error handling taxonomy with error codes and response shapes
---

# Phase 4.4: Extract Error Taxonomy

## Goal
Extract a unified error taxonomy with custom error classes, error codes, and response shapes—enabling Tech Specs to implement consistent error handling.

## Trigger Condition
**Execute this phase**: ALWAYS (universal for all projects).

Error handling patterns exist in every project; this phase documents them for consistency.

## Input
- All source files (scan for error patterns)
- `source-structure.json.discoveredLocations.exceptions` if available

## Steps

### 1. Find Custom Error Classes
Scan for error/exception class definitions:

| Ecosystem | Detection Patterns |
|-----------|-------------------|
| TypeScript/JS | `class * extends Error`, `class *Exception` |
| Java | `class * extends Exception`, `class * extends RuntimeException` |
| Python | `class *(Exception)`, `class *(BaseException)` |
| Go | `type *Error struct`, `errors.New()` |

### 2. Extract Error Class Definitions
For EACH custom error class:

| Field | Source |
|-------|--------|
| `className` | Class/type name |
| `extends` | Parent error class |
| `fields` | Additional fields (code, status, details) |
| `constructor` | Constructor parameters |

### 3. Identify Error Codes
Search for error code constants/enums:

| Pattern | Example |
|---------|---------|
| String constants | `ERROR_CODES.NOT_FOUND` |
| Numeric codes | `errorCode: 1001` |
| Enum values | `enum ErrorCode { NOT_FOUND = 'NOT_FOUND' }` |
| HTTP status mapping | `status: 404` → `NOT_FOUND` |

### 4. Map HTTP Status Conventions
Document project's HTTP status usage:

| Status Range | Typical Usage |
|--------------|---------------|
| 2xx | Success responses |
| 4xx | Client errors (validation, auth, not found) |
| 5xx | Server errors (internal, unavailable) |

Extract the project's specific conventions:
```javascript
// Example: Search for patterns like:
if (error instanceof ValidationError) {
  res.status(400).json({ code: 'VALIDATION_ERROR', ... });
}
```

### 5. Extract Response Shapes
Document standard response formats:

#### Success Response Shape
```json
{
  "success": {
    "data": "T",
    "meta": { "page": "number", "total": "number" }
  }
}
```

#### Error Response Shape
```json
{
  "error": {
    "code": "string",
    "message": "string", 
    "details": "object | undefined"
  }
}
```

### 6. Map Error Propagation
Document how errors flow through layers:

```
Service Layer (throws ServiceError)
  → Controller Layer (catches, transforms)
    → Client (receives ErrorResponse)
```

## Output

### `analysis/error-taxonomy.json`
```json
{
  "errorClasses": {
    "[ErrorClassName]": {
      "file": "[path]",
      "extends": "Error",
      "fields": {
        "code": { "type": "string", "required": true },
        "message": { "type": "string", "required": true },
        "httpStatus": { "type": "number", "required": false },
        "details": { "type": "object", "required": false }
      },
      "constructor": {
        "parameters": ["message: string", "code: string"]
      },
      "usedIn": ["[file:line]"]
    }
  },
  "errorCodes": {
    "[ERROR_CODE]": {
      "source": "[file:line where defined]",
      "httpStatus": 400,
      "defaultMessage": "[default error message]",
      "errorClass": "[ErrorClassName that uses this code]",
      "usedIn": [
        { "file": "[path]", "line": 45, "context": "[brief context]" }
      ]
    }
  },
  "responseShapes": {
    "success": {
      "structure": {
        "data": "[T]",
        "meta": {
          "page": "number?",
          "pageSize": "number?",
          "total": "number?"
        }
      },
      "definedIn": "[file if centrally defined]"
    },
    "error": {
      "structure": {
        "error": {
          "code": "string",
          "message": "string",
          "details": "object?",
          "stack": "string? (dev only)"
        }
      },
      "definedIn": "[file if centrally defined]"
    }
  },
  "httpStatusConventions": {
    "200": "Success - resource returned",
    "201": "Created - resource created",
    "204": "No Content - success with no body",
    "400": "Bad Request - validation errors",
    "401": "Unauthorized - authentication required",
    "403": "Forbidden - insufficient permissions",
    "404": "Not Found - resource doesn't exist",
    "409": "Conflict - resource state conflict",
    "422": "Unprocessable Entity - semantic validation failure",
    "500": "Internal Server Error - unexpected error",
    "503": "Service Unavailable - dependent service down"
  },
  "errorPropagation": {
    "pattern": "catch-transform | bubble-up | global-handler",
    "layers": [
      {
        "layer": "Repository",
        "throws": ["DatabaseError", "NotFoundError"],
        "catches": []
      },
      {
        "layer": "Service",
        "throws": ["BusinessError", "ValidationError"],
        "catches": ["DatabaseError → NotFoundError"]
      },
      {
        "layer": "Controller",
        "throws": [],
        "catches": ["ALL → HttpErrorResponse"]
      }
    ],
    "globalHandler": {
      "file": "[path to global error handler]",
      "pattern": "Express middleware | Spring @ControllerAdvice | etc"
    }
  },
  "_unhandledPatterns": [
    {
      "location": "[file:line]",
      "issue": "Empty catch block - errors swallowed",
      "severity": "warning"
    }
  ],
  "_coverage": {
    "errorClassesExtracted": 8,
    "errorCodesDocumented": 23,
    "handlersWithProperErrorHandling": "87%"
  }
}
```

## Integration Points

### With api-contracts.json
Error response types should be referenced in endpoint documentation:

```json
{
  "endpoints": [{
    "path": "/api/users",
    "errorResponses": {
      "400": { "ref": "error-taxonomy.json#errorCodes.VALIDATION_ERROR" },
      "404": { "ref": "error-taxonomy.json#errorCodes.NOT_FOUND" }
    }
  }]
}
```

### With inter-service-contracts.json
Document how errors from other services are handled:

```json
{
  "calls": [{
    "path": "/api/orders",
    "errorHandling": {
      "onServiceError": "retry 3x then throw ServiceUnavailableError",
      "onTimeout": "throw TimeoutError with circuit breaker"
    }
  }]
}
```

## Critical Rules

1. **Exhaustive Classes**: Extract ALL custom error/exception classes
2. **Code Registry**: Document ALL error codes in use
3. **Shape Consistency**: Verify all layers use consistent response shapes
4. **Propagation Mapping**: Document error flow through architectural layers
5. **Anti-Patterns**: Flag unhandled errors, empty catches, swallowed exceptions
6. **HTTP Conventions**: Document project-specific status code meanings
