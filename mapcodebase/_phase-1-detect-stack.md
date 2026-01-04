---
description: Phase 1 - Detect project stack and discover source structure dynamically
---

# Phase 1: Detect Stack

## Goal
Dynamically detect project ecosystem, layers, and discover ALL relevant source locations.

## Dynamic Detection Algorithm

### Step 1: Detect Ecosystem
Check for package manager files to determine language ecosystem:

| File | Ecosystem |
|------|-----------|
| `package.json` | Node/JS/TS |
| `pom.xml` or `build.gradle` | JVM (Java/Kotlin) |
| `requirements.txt` or `pyproject.toml` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `*.csproj` or `*.sln` | .NET |

### Step 2: Detect Layers by Pattern Scanning
Scan file/directory names for keywords (NOT specific frameworks):

| Layer | Keywords to Match |
|-------|-------------------|
| HTTP/API | `api`, `controller`, `route`, `handler`, `endpoint` |
| State | `store`, `state`, `slice`, `reducer`, `atom` |
| Data | `repository`, `entity`, `model`, `schema`, `migration` |
| UI | `component`, `view`, `page`, `widget`, `screen` |
| Hooks | `hook`, `use`, `composable` |
| Services | `service`, `provider`, `client` |

### Step 3: Discover ALL Relevant Directories
Use glob patterns to find directories:

```
**/apis/**     → apiLocations[]
**/api/**      → apiLocations[]
**/clients/**  → apiLocations[]
**/hooks/**    → hookLocations[]
**/composables/** → hookLocations[]
**/store/**    → stateLocations[]
**/redux/**    → stateLocations[]
**/state/**    → stateLocations[]
**/services/** → serviceLocations[]
**/types/**    → typeLocations[]
**/models/**   → typeLocations[]
**/entities/** → typeLocations[]
```

**Critical**: List EVERY matching directory, not just the first one found.

### Step 4: Detect Multi-Site/Variant Architecture
Scan for patterns indicating multiple site/theme variants:

| Pattern | Indicator |
|---------|-----------|
| `**/sites/**` or `**/themes/**` | Site variant directories |
| Config with `site`, `tenant`, `brand` keys | Multi-tenant configuration |
| Same component name in different folders | Override pattern |

**Output in `source-structure.json`**:
```json
{
  "multiSite": {
    "detected": true,
    "baseDir": "[path to base/shared components]",
    "variants": ["[variant1]", "[variant2]"],
    "overridePattern": "variant extends base"
  }
}
```

## Output

### `analysis/techstack.md`
```markdown
# [Project] Tech Stack

- **Ecosystem**: [detected]
- **Language**: [with version from config]
- **Layers**: [list of detected layers]
- **Framework**: [if identifiable from dependencies]
- **Test Framework**: [if found]

## Path Aliases
| Alias | Path |
|-------|------|
```

### `analysis/source-structure.json`
```json
{
  "ecosystem": "node | jvm | python | go | rust | dotnet",
  "layers": ["http", "state", "data", "ui"],
  "discoveredLocations": {
    "apis": ["ALL directories matching API patterns"],
    "hooks": ["ALL directories matching hook patterns"],
    "state": ["ALL directories matching state patterns"],
    "services": ["ALL directories matching service patterns"],
    "types": ["ALL directories matching type patterns"],
    "components": ["ALL directories matching component patterns"]
  },
  "fileCount": {
    "apis": 12,
    "hooks": {
      "total": 62,
      "[dir1]": 36,
      "[dir2]": 26
    },
    "state": {
      "slices": 16,
      "total": 20
    },
    "classes": 164,
    "types": 25
  }
}
```

### Step 5: Detect Capabilities (NEW - MANDATORY)
Scan for capability indicators that trigger conditional phases. Output in `source-structure.json.detectedCapabilities`:

| Capability | Detection Patterns | Triggers Phase |
|------------|-------------------|----------------|
| `hasORM` | `@Entity`, `@Table`, `schema.prisma`, `new Schema`, `models.Model` | Phase 2.5 |
| `hasValidation` | `zod`, `yup`, `joi`, `class-validator`, `@IsString`, `@IsNotEmpty` | Phase 2.6 |
| `hasStateManagement` | `createSlice`, `createStore`, `defineStore`, `atom(`, `observable` | Phase 3.7 |
| `hasDesignTokens` | `tailwind.config`, `theme.ts`, `tokens.json`, CSS custom properties | Phase 3.5 |
| `hasComponentLibrary` | `React.FC`, `defineComponent`, `@Component`, `.svelte` files | Phase 3.6 |
| `callsExternalServices` | `HttpClient`, `fetch(`, `axios`, `RestTemplate`, `WebClient` | Phase 4.2 |
| `hasThirdPartySDKs` | `stripe`, `sendgrid`, `aws-sdk`, `twilio`, `@google-cloud` | Phase 4.3 |
| `hasCustomErrors` | `extends Error`, `extends Exception`, `extends RuntimeException` | Phase 4.4 |

**Detection Algorithm**:
```
For EACH capability:
  1. Search codebase for detection patterns
  2. If ANY pattern found → set capability to true
  3. Store evidence (file:line) for verification
```

**Output in `source-structure.json`**:
```json
{
  "detectedCapabilities": {
    "hasORM": true,
    "hasValidation": false,
    "hasStateManagement": false,
    "hasDesignTokens": false,
    "hasComponentLibrary": false,
    "callsExternalServices": true,
    "hasThirdPartySDKs": false,
    "hasCustomErrors": true,
    "_evidence": {
      "hasORM": ["src/main/java/entity/User.java:12 (@Entity)"],
      "hasCustomErrors": ["src/main/java/exception/BookingException.java:5"]
    }
  }
}
```

> [!IMPORTANT]
> **Phase 4.4 (Error Taxonomy) is ALWAYS required** regardless of `hasCustomErrors` flag.
> The flag only indicates if custom error classes exist; the phase documents ALL error handling patterns.

## Critical Rules
1. **Exhaustive discovery**: Count files in each location for later coverage validation
2. **No hardcoded frameworks**: Detect by patterns, not specific library names
3. **Multi-directory support**: Projects may have multiple API/hook/state directories
4. **Mandatory fileCount fields**: `hooks.total`, `state.slices`, `classes`, `apis` are REQUIRED for Phase 4.5 enforcement
5. **Mandatory detectedCapabilities**: MUST output `detectedCapabilities` object for conditional phase triggers
