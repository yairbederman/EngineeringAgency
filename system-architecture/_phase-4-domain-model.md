# Phase 4: Unified Domain Model

> **Goal**: Merge entity definitions across projects to identify canonical entities and type conflicts.

---

## Input

- `${ANALYSIS_DIR}/project-inventory.json` (from Phase 1)
- Each project's `entity-contracts.json`

## Output

- `${ANALYSIS_DIR}/unified-domain-model.json`

---

## Instructions

### Step 1: Extract All Entities

For each `ready` project:
- Read `entity-contracts.json`
- Extract all entities with their fields

### Step 2: Group by Entity Name

Group entities that share the same name across projects:

```
BookingData:
  - wg-ordermanager-api/BookingData (12 fields)
  - wg-payment-api/BookingData (8 fields)
  - wg-tripdetails-api/BookingData (10 fields)
```

### Step 3: Identify Canonical Definitions

For each entity group, determine:

1. **Canonical Source**: The project that "owns" this entity
   - Typically the project where it's most complete
   - Or the project that creates/manages this data

2. **Derived Copies**: Other projects that use a subset
   - May have fewer fields
   - May be read-only views

3. **Conflicts**: Fields with different types across projects

### Step 4: Classify Entities

| Classification | Description |
|---------------|-------------|
| **Canonical** | Single authoritative definition |
| **Shared** | Defined once, used in multiple projects |
| **Duplicated** | Same entity defined in multiple projects |
| **Conflicting** | Same name but incompatible fields |

### Step 5: Build Unified Model

For each canonical entity:
- Merge all fields
- Note field source
- Flag conflicts

---

## Output Schema

```json
{
  "generatedAt": "ISO timestamp",
  "entities": [
    {
      "name": "BookingData",
      "classification": "duplicated",
      "canonicalSource": {
        "project": "wg-ordermanager-api",
        "file": "src/main/java/.../BookingData.java",
        "reason": "Contains all fields, manages booking lifecycle"
      },
      "unifiedFields": [
        {
          "name": "bookingId",
          "type": "String",
          "definedIn": ["wg-ordermanager-api", "wg-payment-api", "wg-tripdetails-api"],
          "status": "consistent"
        },
        {
          "name": "totalPrice",
          "type": "BigDecimal",
          "definedIn": ["wg-ordermanager-api", "wg-payment-api"],
          "conflict": {
            "wg-ordermanager-api": "BigDecimal",
            "wg-tripdetails-api": "Double"
          },
          "status": "conflicting"
        }
      ],
      "usedIn": ["wg-client", "wg-payment-api", "wg-tripdetails-api"]
    }
  ],
  "domainAreas": [
    {
      "name": "Booking",
      "entities": ["BookingData", "BookingRequest", "BookingResponse"],
      "ownedBy": "wg-ordermanager-api"
    },
    {
      "name": "Payment",
      "entities": ["PaymentData", "PaymentRequest", "PaymentResponse"],
      "ownedBy": "wg-payment-api"
    },
    {
      "name": "Search",
      "entities": ["SearchRequest", "SearchResponse", "PackageData"],
      "ownedBy": "wg-search-api"
    }
  ],
  "_conflicts": [
    {
      "entity": "BookingData",
      "field": "totalPrice",
      "issue": "Type mismatch: BigDecimal vs Double",
      "recommendation": "Standardize to BigDecimal for currency precision"
    }
  ],
  "_coverage": {
    "projectsAnalyzed": 8,
    "entitiesFound": 156,
    "canonicalEntities": 45,
    "duplicatedEntities": 12,
    "conflictingFields": 3
  }
}
```

---

## Domain Area Detection

Automatically group entities into domain areas based on:

1. **Naming patterns**: `Booking*`, `Payment*`, `Search*`
2. **Package structure**: Entities in same package
3. **Usage patterns**: Entities used together in API calls

---

## Validation

| Check | Requirement |
|-------|-------------|
| All entity files read | Count matches projects × entity files |
| Canonical assigned | Every entity has a `canonicalSource` |
| Conflicts documented | No silent type mismatches |

---

## Conflict Resolution Recommendations

| Conflict Type | Recommendation |
|--------------|----------------|
| Type mismatch | Standardize to canonical source type |
| Missing field | Check if intentionally excluded or oversight |
| Extra field | May be valid extension or legacy |
| Name collision | Different concepts → rename one |
