---
description: Phase 3 - Extract all API endpoints with exhaustive coverage
---

# Phase 3: Extract APIs

## Goal
Extract ALL API endpoints with HTTP method, path, request type, and response type. **Exhaustive coverage—no directory left undocumented.**

## Input
Use `source-structure.json.discoveredLocations.apis` for ALL API locations.

## Steps

### 1. Exhaustive Directory Scanning
For EACH directory in `discoveredLocations.apis`:
1. List all files in the directory
2. Scan each file for API patterns
3. Document at least once per directory (even if empty)

### 2. Dynamic API Detection (Language Agnostic)
Scan files for these patterns:

| Pattern Type | Examples |
|--------------|----------|
| **Method keywords** | `get`, `post`, `put`, `delete`, `patch` in function names |
| **Route patterns** | `/api/`, `/:param`, `/v1/`, path parameters |
| **HTTP imports** | `fetch`, `axios`, `http`, `request`, `HttpClient` |
| **Decorator patterns** | `@Get`, `@Post`, `@RequestMapping`, `@app.route`, `router.` |
| **Export patterns** | `*Client`, `*Controller`, `*Handler`, `*Route` |

### 2.5 Empty Client Verification (BLOCKING GATE)

> **⛔ BLOCKING**: API clients with `endpoints: []` using invalid skip reasons will FAIL the Phase 4.5 gate.

If an API client file would have `endpoints: []`:
1. **MUST view the file contents** to verify no methods making HTTP calls
2. Search for: `fetch`, `axios`, `post`, `get`, `request`, `baseRequest`, `http`, `HttpClient`
3. If methods exist but pattern not matched → **EXTRACT NOW**, log individual `_unresolved` items
4. If truly empty (just setup/config with no HTTP calls) → use ONLY valid reason: `"configuration-only"`

**VALID skip reasons** (these are acceptable for empty endpoints):
- `"configuration-only"` - File contains only config/constants, no HTTP methods
- `"base-class"` - Abstract base class extended by other clients
- `"deprecated"` - Marked as deprecated in code comments

**INVALID reasons** (MUST extract instead, these BLOCK Phase 5):
- `"endpoints not fully extracted"` → **Extract now or fail**
- `"CMS endpoints not fully extracted"` → **Extract now or fail**
- `"complex API structure"` → **Extract now**, mark unresolved items
- `"Pending"` or any "later" language → **Invalid, extract now**
- Any blank or vague reason → **Invalid**

### 3. For Each Endpoint, Extract:
- HTTP method (GET/POST/PUT/DELETE/PATCH)
- Path (resolve path params like `:id` or `{id}`)
- Handler function name
- Request body type → cross-reference with `entity-contracts.json`
- Response type → cross-reference with `entity-contracts.json`
- Query parameters

### 3.5 Extract Validation Rules (MANDATORY for /lognet TaskPlanning)

> **⛔ BLOCKING**: Every `requestFields` entry MUST include a `validation` object. Omission blocks Phase 4.5 gate.

For EACH request field, extract validation metadata:

| Pattern to Detect | Validation Type |
|-------------------|-----------------|
| `@IsNotEmpty()`, `required: true`, `!optional` | `required: true` |
| `@MaxLength(N)`, `maxLength: N`, `max: N` | `maxLength: N` |
| `@MinLength(N)`, `minLength: N`, `min: N` | `minLength: N` |
| `@IsEmail()`, `@IsUrl()`, `@Pattern()` | `pattern: "[regex]"` |
| `@IsInt()`, `@IsNumber()`, `@IsPositive()` | `type: "integer"` / `min: 0` |
| `@IsOptional()`, `optional: true`, `?:` | `required: false` |
| TypeScript `?:` optional marker | `required: false` |
| No `?` in TypeScript field | `required: true` (inferred) |

**Output Format for Validation**:
```json
{
  "requestFields": {
    "destination": {
      "type": "string",
      "validation": {
        "required": true,
        "minLength": 2,
        "maxLength": 100
      }
    },
    "passengers": {
      "type": "number",
      "validation": {
        "required": true,
        "min": 1,
        "max": 9
      }
    }
  }
}
```

**Validation Inference Rules**:
1. If explicit decorator/annotation found → use it
2. If TypeScript `?:` found → `required: false`
3. If no `?:` in TypeScript → `required: true` (inferred)
4. If completely unresolvable → Mark `validation: { "inferred": true, "_unresolved": true }` AND log in `_unresolved.validations`

### 4. Track Coverage
Count API directories and files documented.

## Output

### `analysis/api-contracts.json`
```json
{
  "[ClientOrController]": {
    "file": "[path]",
    "basePath": "[base URL or constant reference]",
    "endpoints": [
      {
        "method": "POST",
        "path": "/api/v1/search",
        "handler": "[function name]",
        "requestType": "[type name]",
        "requestFields": {
          "[field]": "[type with description]"
        },
        "responseType": "[type name]",
        "responseFields": {
          "[field]": "[type]"
        },
        "queryParams": {
          "[param]": "[type]"
        }
      }
    ]
  },
  "_sharedTypes": {
    "[TypeName]": {
      "file": "[path]",
      "fields": {}
    }
  },
  "_coverage": {
    "apiDirectories": 12,
    "apiFilesScanned": 12,
    "endpointsDocumented": 45,
    "skipped": [
      { "path": "[dir]", "reason": "utility file, no endpoints" }
    ]
  }
}
```

## Cross-Project API Owner Tracking (For Multi-Repo Projects)

> **Purpose**: Enables /lognet TechSpec to identify which backend project owns each API endpoint.

For **frontend projects** that call external backend APIs:
1. Identify the backend base URL (from `baseRequest`, axios config, or environment variables)
2. Map each endpoint to its owner project:
   - `/api/cms/*` → `PROJECT_CMS_API`
   - `/api/data/*` → `PROJECT_DATA_API`
   - `/api/*` → Infer from path or config

**Output format**:
```json
{
  "SearchApiClient": {
    "endpoints": [...],
    "backendOwner": "PROJECT_DATA_API",
    "baseUrl": "${DATA_API_URL}/api/search"
  }
}
```

**For backend projects**: Mark `"backendOwner": "self"` for all endpoints.

---

## Critical Rules
1. **Exhaustive**: Every directory in `discoveredLocations.apis` MUST appear in output
2. **No skipping**: If a directory has no endpoints, document it in `_coverage.skipped` with reason (valid reasons ONLY)
3. **Resolved types**: Request/response fields must be resolved, not just type names
4. **Cross-reference**: Types should link to `entity-contracts.json`
5. **Validation required**: Every `requestFields` entry MUST include `validation` object
6. **No invalid skip reasons**: "Pending", "not fully extracted" BLOCK Phase 5
