---
description: Phase 2.6 - Extract validation schema contracts for runtime validation rules
---

# Phase 2.6: Extract Validation Schema Contracts

## Goal
Extract validation schemas (Zod, Yup, Joi, class-validator, Spring Validation) with field-level rules—enabling Tech Specs to implement correct runtime validation.

## Trigger Condition
**Execute this phase IF** any of the following are detected:
- Zod (`z.object`, `z.string`)
- Yup (`yup.object`, `yup.string`)
- Joi (`Joi.object`, `Joi.string`)
- class-validator decorators (`@IsNotEmpty`, `@IsEmail`)
- Spring Validation (`@Valid`, `@NotNull`, `@Size`)
- Python Pydantic (`BaseModel` with field validators)

**Skip IF**: No validation library detected.

## Input
Scan `source-structure.json.discoveredLocations.types` and `discoveredLocations.dto` for validation patterns.

## Steps

### 1. Detect Validation Framework
Scan imports and patterns:

| Framework | Detection Patterns |
|-----------|-------------------|
| Zod | `import { z }`, `z.object()`, `z.string()` |
| Yup | `import * as yup`, `yup.object()` |
| Joi | `import Joi`, `Joi.object()` |
| class-validator | `@IsNotEmpty()`, `@IsEmail()`, `@Min()` |
| Spring Validation | `@Valid`, `@NotNull`, `@Size`, `@Pattern` |
| Pydantic | `from pydantic import`, `Field(...)`, `@validator` |

### 2. Extract Schema Definitions
For EACH validation schema:

| Field | Source |
|-------|--------|
| `schemaName` | Variable name or class name |
| `linkedEntity` | Cross-reference with `entity-contracts.json` |
| `fields` | Each validated field with rules |

### 3. Extract Field-Level Rules
Map validation decorators/methods to standardized rules:

| Pattern | Standardized Rule |
|---------|------------------|
| `@IsNotEmpty()`, `.required()`, `@NotNull` | `{ "rule": "required" }` |
| `@IsEmail()`, `.email()` | `{ "rule": "email" }` |
| `@MinLength(N)`, `.min(N)` | `{ "rule": "minLength", "value": N }` |
| `@MaxLength(N)`, `.max(N)` | `{ "rule": "maxLength", "value": N }` |
| `@Min(N)`, `.min(N)` for numbers | `{ "rule": "min", "value": N }` |
| `@Max(N)`, `.max(N)` for numbers | `{ "rule": "max", "value": N }` |
| `@Pattern(regex)`, `.matches(regex)` | `{ "rule": "pattern", "value": "[regex]" }` |
| `@IsOptional()`, `.optional()` | `{ "rule": "optional" }` |
| `.refine()`, `@Validate()` | `{ "rule": "custom", "name": "[validator]" }` |

### 4. Extract Custom Validators
For custom validation functions:

```json
{
  "name": "[validatorName]",
  "file": "[path]",
  "description": "[from JSDoc/docstring]",
  "errorMessage": "[custom error message]"
}
```

### 5. Map Schema-to-Entity
Cross-reference with `entity-contracts.json`:

| Check | Action |
|-------|--------|
| Schema field exists in entity | Document match |
| Schema field missing from entity | Log in `_discrepancies` |
| Entity field has no validation | Log in `_unvalidatedFields` |

## Output

### `analysis/validation-schemas.json`
```json
{
  "detectedFramework": "[Zod|Yup|Joi|class-validator|...]",
  "schemas": {
    "[SchemaName]": {
      "file": "[path]",
      "linkedEntity": "[EntityName from entity-contracts]",
      "fields": {
        "[fieldName]": {
          "type": "string",
          "rules": [
            { "rule": "required", "message": "Field is required" },
            { "rule": "email", "message": "Invalid email format" },
            { "rule": "maxLength", "value": 100 }
          ],
          "customValidator": "[validatorName or null]"
        }
      }
    }
  },
  "customValidators": {
    "[validatorName]": {
      "file": "[path]",
      "usedIn": ["[SchemaName]"],
      "description": "[what it validates]",
      "errorMessage": "[default error message]"
    }
  },
  "_schemaEntityMapping": {
    "[SchemaName]": {
      "entity": "[EntityName]",
      "coverage": "full | partial",
      "unvalidatedFields": ["[field names without validation]"]
    }
  },
  "_coverage": {
    "schemasExtracted": 12,
    "fieldsWithValidation": 87,
    "customValidators": 5
  }
}
```

## Integration with api-contracts.json

After this phase, Phase 3's `requestFields.validation` should reference these schemas:

```json
{
  "requestFields": {
    "[fieldName]": {
      "type": "string",
      "validation": {
        "schemaRef": "[SchemaName from validation-schemas]",
        "rules": "[copied from validation-schemas for quick access]"
      }
    }
  }
}
```

## Critical Rules

1. **Exhaustive Rules**: Extract ALL validation rules, not just presence checks
2. **Custom Validators**: Document custom validators with descriptions
3. **Error Messages**: Capture custom error messages when defined
4. **Entity Mapping**: Always link to `entity-contracts.json` for traceability
5. **Unvalidated Fields**: Flag entity fields without corresponding validation
