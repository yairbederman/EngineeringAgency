---
description: Phase 2 - Extract all types/interfaces with full field definitions and no `any` leakage
---

# Phase 2: Extract Entities

## Goal
Extract ALL exported types/interfaces/classes with **complete field definitions**—no `any` types without resolution attempts.

## Input
Use `source-structure.json` to find ALL type locations.

## Steps

### 1. Scan Type Files (All Ecosystems)
Scan ALL files in `discoveredLocations.types`:

| Ecosystem | Patterns to Match |
|-----------|-------------------|
| TypeScript | `export interface`, `export type`, `export class`, `export enum` |
| Java | `public class`, `public interface`, `public enum`, `record` |
| Python | `@dataclass`, `class` with type hints, Pydantic models |
| Go | `type ... struct`, `type ... interface` |
| Kotlin | `data class`, `sealed class`, `interface` |

### 1.5 Recursive Class Scanning (Required for OOP Codebases)
For EACH directory in `discoveredLocations.types` that contains classes:
1. **Recursively** list ALL `.ts`, `.java`, `.py`, `.cs`, `.kt` files in subdirectories
2. Extract EVERY `export class` / `public class` definition found
3. Track coverage: `classFilesFound` vs `classesExtracted`
4. **Minimum threshold**: Classes ≥70% coverage required

**Directory Traversal Rule**:
- If a types location is a directory (e.g., `src/sdk/classes/`), scan ALL subdirectories
- Count files in each subdirectory individually
- Document any skipped files with valid reason

**Invalid skip reasons for classes**:
- "Pending deep scanning" → MUST scan now
- "Too many files" → Scan all, no pagination excuse
- "Complex structure" → Scan anyway, mark individual `_unresolved`

### 2. For Each Exported Type, Extract:
- Name and file path
- Kind: `interface | class | type | enum`
- **ALL fields with resolved types**
- Generic parameters
- Parent types (extends/implements)
- Documentation (JSDoc/docstring)

### 3. Anti-`any` Resolution
When encountering `any`, `Object`, `dynamic`, or untyped:

1. **Trace usage** in the codebase to find actual shape
2. **Check runtime assignment** for object structure
3. If unresolvable, document in `_unresolved` with file:line

**Never output `"type": "any"` without a resolution attempt.**

### 3.5 Recursive Type Resolution
When a field references another type:
1. Check if referenced type is already extracted
2. If not, extract it (follow same rules)
3. If external/third-party, document in `_externalTypes`
4. **Limit**: Max recursion depth of 3 levels

### 4. Class Method Extraction
For each entity with `kind: "class"`, extract:

| Field | Description |
|-------|-------------|
| `methods` | Array of public method names |
| `methodSignatures` | Object mapping method name → signature |

**Signature Format**:
```json
{
  "methodName": {
    "parameters": [{ "name": "param", "type": "string" }],
    "returnType": "number",
    "description": "From JSDoc/docstring if available"
  }
}
```

**Priority**: Focus on classes that:
- Are imported by multiple files
- Have >5 methods
- Are domain entities (not utilities)

### 4. Track Coverage
Count files in type locations vs types documented.

## Output

### `analysis/entity-contracts.json`
```json
{
  "[TypeName]": {
    "file": "[path]",
    "kind": "interface | class | type | enum",
    "fields": {
      "[fieldName]": {
        "type": "[resolved type]",
        "required": true,
        "description": "[from docs]"
      }
    },
    "extends": ["[parent]"],
    "methods": ["[methodName]"]
  },
  "_enums": {
    "[EnumName]": {
      "file": "[path]",
      "values": ["VALUE1", "VALUE2"]
    }
  },
  "_unresolved": [
    {
      "type": "[field name]",
      "inEntity": "[TypeName]",
      "file": "[path:line]",
      "reason": "[why unresolvable]"
    }
  ],
  "_coverage": {
    "typeFiles": 15,
    "typesExtracted": 42,
    "unresolvedAny": 3
  }
}
```

## Critical Rules
1. **Never** list just type names—always extract complete field definitions
2. **Never** output `"type": "any"` without logging in `_unresolved`
3. **Always** include enum values, not just enum names
4. **Track** coverage for later validation
